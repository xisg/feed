---
title: Devirtualization in LLVM and Clang
url: https://blog.llvm.org/2017/03/devirtualization-in-llvm-and-clang.html
published: "2017-03-10T13:23:00Z"
feed: llvm
guid: https://blog.llvm.org/2017/03/devirtualization-in-llvm-and-clang.html
---

# Devirtualization in LLVM and Clang

_This blog post is part of a series of blog posts from students who were funded by the LLVM Foundation to attend the 2016 LLVM Developers' Meeting in San Jose, CA. Please visit the LLVM Foundation's [webpage](http://llvm.org/foundation) for more information on our Travel Grants program._

_This post is from Piotr Padlewski on his work that he presented at the meeting:_

This blogpost will show how C++ devirtualization is performed in current (4.0) clang and LLVM and also ongoing work on -fstrict-vtable-pointers features.

Devirtualization done by the frontend

In order to transform a virtual call into a direct call, the frontend must be sure that there are no overrides of vfunction in the program or know the dynamic type of object. Compilation proceeds one translation unit at a time, so, barring LTO, there are only a few cases when the compiler may conclude that there are no overrides:

- either the class or virtual method is marked as final
- the class is defined in an anonymous namespace and has no deriving classes in its translation unit

The latter is more tricky for clang, which translates the source code in chunks on the fly (see: ASTProducer and ASTConsumer), so is not able to determine if there are any deriving classes later in the source. This could be dealt with in a couple of ways:

- give up immediate generation
- run data flow analysis in LLVM to find all the dynamic types passed to function, which has static linkage
- hope that every use of the virtual function, which is necessarily in the same translation unit, will be inlined by LLVM -- static linkage increases the chances of inlining

Store to load propagation in LLVM

In order to devirtualize a virtual call we need:

- value of vptr - which virtual table is pointed by it
- value of vtable slot - which exact virtual function it is

Because vtables are constant, the latter value is much easier to get when we have the value of vptr. The only thing we need is vtable definition, which can be achieved by using available\_externally linkage.

In order to figure out the vptr value, we have to find the store to the same location that defines it. There are 2 analysis responsible for it:

- MemDep (Memory Dependence Analysis) is a simple linear algorithm that for each quered instruction iterates through all instructions above and stops when first dependency is found. Because queries might be performed for each instruction we end up with a quadratic algorithm. Of course quadratic algorithms are not welcome in compilers, so MemDep can only check certain number of instructions.
- Memory SSA on the other hand has constant complexity because of caching. To find out more, watch “Memory SSA in 5minutes” ( [https://www.youtube.com/watch?v=bdxWmryoHak](https://www.youtube.com/watch?v=bdxWmryoHak)). MemSSA is a pretty new analysis and it doesn’t have all the features MemDep has, therefore MemDep is still widely used.

The LLVM main pass that does store to load propagation is GVN - Global Value Numbering.

Finding vptr store

In order to figure out the vptr value, we need to see store from constructor. To not rely on constructor's availability or inlining, we decided to use the @llvm.assume intrinsic to indicate the value of vptr. Assume is akin to assert - optimizer seeing call to @llvm.assume(i1 %b) can assume that %b is true after it. We can indicate vptr value by comparing it with the vtable and then call the @llvm.assume with the result of this comparison.

## **call void @\_ZN1AC1Ev(%struct.A\* %a) ; call ctor**  **%3 = load {...} %a                  ; Load vptr**  **%4 = icmp eq %3, @\_ZTV1A      ; compare vptr with vtable**    **call void @llvm.assume(i1 %4)**

Calling multiple virtual functions

## A non-inlined virtual call will clobber the vptr. In other words, optimizer will have to assume that vfunction might change the vptr in passed object. This sounds like something that never happens because vptr is “const”. The truth is that it is actually weaker than C++ const member, because it changes multiple times during construction of an object (every base type constructor or destructor must set vptrs). But vptr can't be changed during a virtual call, right? Well, what about that?

## **void A::foo() { // virtual**  **static\_assert(sizeof(A) == sizeof(Derived));**  **new(this) Derived;**  **}**      **This is call of placement new operator - it doesn’t allocate new memory, it just creates a new object in the provided location. So, by constructing a Derived object in the place where an object of type A was living, we change the vptr to point to Derived’s vtable. Is this code even legal? C++ Standard says yes.**      **However it turns out that if someone called foo 2 times (with the same object), the second call would be undefined behavior. Standard pretty much says that call or dereference of a pointer to an object whose lifetime has ended is UB, and because the standard agrees that nuking object from inside ends its lifetime, the second call is UB. Be aware that this is only because a zombie pointer is used for the second call. The pointer returned by placement new is considered alive, so performing calls on that pointer is valid. Note that we also silently used that fact with the use of assume.**    **(un)clobbering vptr**

We need to somehow say that vptr is invariant during its lifetime. We decided to introduce a new metadata for that purpose - !invariant.group. The presence of the invariant.group metadata on the load/store tells the optimizer that every load and store to the same pointer operand within the same invariant group can be assumed to load or store the same value. With -fstrict-vtable-pointers Clang decorates vtable loads with invariant.group metadana coresponding to caller pointer type.

We can enhance the load of virtual function (second load) by decorating it with !invariant.load, which is equivalent of saying “load from this location is always the same”, which is true because vtables never changes. This way we don’t rely on having the definition of vtable.

## **Call like:**      **void g(A \*a) {**  **a->foo();**  **a->foo();**    **}**      **Will be translated to:** define void @function(%struct.A\* %a) {    %1 = load {...} %a, !invariant.group !0    %2 = load {...} %1, !invariant.load !1    call void %2(%struct.A\* %a)      %3 = load {...} %a, !invariant.group !0    %4 = load {...} %4, !invariant.load !1    call void %4(%struct.A\* %a)    ret void  }    !0 = !{!"\_ZTS1A"} ; mangled type name of A  !1 = !{}     And now by magic of GVN and MemDep:     define void @function(%struct.A\* %a) {    %1 = load {...} %a, !invariant.group !0    %2 = load {...} %1, !invariant.load !1    call void %2(%struct.A\* %a)    call void %2(%struct.A\* %a)    ret void  }     **With this, llvm-4.0 is be able to devirtualize function calls inside loops.**    **Barriers**

In order to prevent the middle-end from finding load/store with the same !invariant.group metadata, that would come from construction/destruction of dead dynamic object, @llvm.invariant.group.barrier was introduced. It returns another pointer that aliases its argument but is considered different for the purposes of load/store invariant.group metadata. Optimizer won’t be able to figure out that returned pointer is the same because intrinsics don’t have a definition. Barrier must be inserted in all the places where the dynamic object changes:

- constructors
- destructors
- placement new of dynamic object

Dealing with barriers

Barriers hinder some other optimizations. Some ideas how it could be fixed:

- stripping invariant.group metadata and barriers just after devirtualization. Currently it is done before codegen. The problem is that most of the devirtualization comes from GVN, which also does most of the optimizations we would miss with barriers. GVN is expensive therefore it is run only once. It also might make less sense if we are in LTO mode, because that would limit the devirtualization in the link phase.
- teaching important passes to look through the barrier. This might be very tricky to preserve the semantics of barrier, but e.g. looking for dependency of load without invariant.group by jumping through the barrier to find a store without invariant.group, is likely to do the trick.
- removing invariant.barrier when its argument comes from alloca and is never used etc.

To find out more details about devirtualization check my talk ( [http://llvm.org/devmtg/2016-11/#talk6](http://llvm.org/devmtg/2016-11/#talk6)) from LLVM Dev Meeting 2016.

About author

## Undergraduate student at University of Warsaw, currently working on C++ static analysis in IIIT.
