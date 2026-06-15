---
title: Cling -- Beyond Just Interpreting C++
url: https://blog.llvm.org/posts/2021-03-25-cling-beyond-just-interpreting-cpp/
published: "2021-03-25T10:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2021-03-25-cling-beyond-just-interpreting-cpp/
---

# Cling -- Beyond Just Interpreting C++

In our previous blog post [“Interactive C++ for Data Science”](https://blog.llvm.org/posts/2020-12-21-interactive-cpp-for-data-science/) we described eval-style programming, interactive C++ in Notebooks and CUDA. Thispost will discuss some developed applications of Cling supportinginteroperability and extensibility. We aim to demonstrate template instantiationon demand; embedding Cling as a service; and showcase an extension enablingon-the-fly automatic differentiation.

## Template Instantiation on Demand

Cling implements a facility called `LookupHelper`, which takes C++ code andchecks if a declaration with that qualified name already exists. For instance:

```cpp
[cling] #include "cling/Interpreter/Interpreter.h"[cling] #include "cling/Interpreter/LookupHelper.h"[cling] #include "clang/AST/Decl.h"[cling] struct S{};[cling] cling::LookupHelper& LH = gCling->getLookupHelper()(cling::LookupHelper &) @0x7fcba3c0bfc0[cling] auto D = LH.findScope("std::vector<S>", cling::LookupHelper::DiagSetting::NoDiagnostics)(const clang::Decl *) 0x1216bdcd8[cling] D->getDeclKindName()(const char *) "ClassTemplateSpecialization"
```

In this particular case, `findScope` instantiates the template and returns itsclang AST representation. Template instantiation on demand addresses the commonlibrary problem of template combinatorial explosion. Template instantiation ondemand and conversion of textual qualified C++ names into entity metainformation has proven to be a very powerful mechanism aiding data serializationand language interoperability.

## Language Interop on Demand

An example is [cppyy](https://cppyy.readthedocs.io/), which provides automaticPython bindings, at runtime, to C++ code through Cling. Python is itself adynamic language executed by an interpreter, thus making the interaction withC++ code more natural when intermediated by Cling. Examples include runtimetemplate instantiations, function (pointer) callbacks, cross-languageinheritance, automatic downcasting, and exception mapping. Many advanced C++features such as placement new, multiple virtual inheritance,variadic templates, etc., are naturally resolved by the LookupHelper.

cppyy achieves high performance through an all-lazy approach to runtime bindingsconstruction and specializations of common cases through runtime reflection.As such, it has a much lower call overhead than e.g. pybind11, and looping overa `std::vector` through cppyy is faster than looping over a numpy array of thesame type. Taking it a step further, its implementation for PyPy, a fullycompatible Python interpreter sporting at [tracing JIT](https://pypy.org), canin many cases provide native access to C++ code in PyPy’s JIT, includingoverload resolution and JIT hints that allow for aggressive optimizations.

Thanks to Cling’s runtime reflection, cppyy makes maintaining a large softwarestack simpler: except for cppyy’s own python-interpreter binding, it does nothave any compiled code that is Python-dependent. I.e., cppyy-based extensionmodules require no recompilation when switching Python versions (or even whenswitching between the CPython and PyPy interpreters, say).

The example below shows the tight integration of C++ and python; shows the tightback and forth communication of the template instantiation and thecross-inheritance overrides; and the runtime behaviors (everything happens atruntime, there is no compiled code here).

```python
import cppyycppyy.cppdef(r"""\template<typename T> class Producer {private: T m_value;protected: virtual T produce_imp() = 0;public: Producer(const T& value) : m_value(value) {} virtual ~Producer() {} T produce_total() { return m_value + produce_imp(); }};class Consumer {public: template<typename T> void consume(Producer<T>& p) { std::cout << "received: \"" << p.produce_total() << "\"\n"; }};""")def factory(base_v, *derived_v): class _F(cppyy.gbl.Producer[type(base_v)]): def __init__(self, base_v, *derived_v): super().__init__(base_v) self._values = derived_v def produce_imp(self): return type(base_v)(sum(self._values)) return _F(base_v, *derived_v)consumer = cppyy.gbl.Consumer()for producer in [factory(*x) for x in \ (("hello ", 42), (3., 0.14, 0.0015))]: consumer.consume(producer)
```

Output:

```bash
python3 cppyy_demo.pyreceived: "hello 42"received: "3.1415"
```

In the snippet we create python classes based on python arguments, which derivefrom a templated C++ class instantiated with a type. The python class providesthe implementation for a protected function that is called from a publicfunction, resulting in the expected return value, which is printed. We aim tohighlight:

- Python creates classes at runtime, as can Cling, even when they aredeclared in a module (the relevant classes are here created in a factorymethod);
- Templated C++ classes can be instantiated on the fly from Python, by takingthe type of the argument (i.e. using introspection at runtime in Python) tocreate the C++ base class for the Python class.
- Cross-language derivation is at runtime, with no support needed from the C++class, other than a virtual destructor and virtual methods;
- C++ “protected” methods can be overridden in Python, even though Python hasno such concept and you can not actually call protected methods from boundC++ objects in Python;
- It all works straight out of the box.

cppyy is used in several large code bases in physics, chemistry, mathematics,and biology. It is readily installable through\[pip from PyPI\] ( [https://pypi.org/project/cppyy/](https://pypi.org/project/cppyy/)) and through [conda](https://anaconda.org/conda-forge/cppyy).

Another example is Symmetry Integration Language (SIL), a D-baseddomain-specific language of functional flavor developed and used internally by [Symmetry Investments](https://symmetryinvestments.com). One of the main goalsof SIL is to be easily interoperable with all sorts of languages and systems,and this is achieved through various plugins. To call C++ code, SIL uses aplugin called `sil-cling`, which acts as a middle ground between SIL and Cling.However, sil-cling does not interact directly with Cling, but throughcppyy-backend, that is cppyy’s C/C++ wrapper around Cling that provides a stableC/C++ reflection API.

There are two core types that are exposed from sil-cling to SIL. One is `CPPNamespace`, which exposes a C++ namespace and allows free function calling,access to namespace’s variables, and object instantiation for the classesdefined in that namespace. The other is `ClingObj`, which is a proxy for a C++object, allowing construction, method calling and the manipulation of theobject’s data members. Given that cppyy represents C++ classes, structs andnamespaces as ‘scopes’ and reflection information about any of these C++entities is obtained through its associated ‘scope’ object, both wrapper typesexposed to SIL hold a reference to their associated scope object which isqueried whenever the wrapper types are used to call C++ code.

All the calls that are done from SIL through the two wrapper types have 3arguments: the wrapper object used, the name of the C++ function that needs tobe called, and (if needed) a sequence of arguments for that function. Once theoverload resolution and the argument conversion are done, sil-cling calls theappropriate cppyy function that will wrap the call and dispatch it to Cling forJIT compilation. At the moment, sil-cling can be used to call C++ libraries like `Boost.Asio`, `dlib` or `Xapian`.

The example below creates a Boost Asio-based client-server application writtenin SIL using the sil-cling plugin. Server.sil contains the SIL code for theserver. It starts by including the relevant header files, using `cppCompile`.The next step is to create wrapper objects for the namespaces that are needed,and this is done by calling cppNamespace with the names of the namespaces thatone needs to access. These CPPNamespace wrappers are used to instantiate classesthat are defined inside the C++ namespaces that they wrap. Using these wrappers,an endpoint, an acceptor socket (which listens for incoming connections) and anactive socket (which handles communication with the client) are created. Thenthe server waits for a connection and, once a client connects, it reads itsmessage and sends a reply.

```d
// Server.silimport * from silclingimport format from formatcppCompile ("#include <boost/asio.hpp>")cppCompile ("#include \"helper.hpp\"")// CPPNamespace wrappersasio = cppNamespace("boost::asio")tcp = cppNamespace("boost::asio::ip::tcp")helpers = cppNamespace("helpme")// Using namespace wrappers to instantiate classes - creates ClingObj(s)ioService = asio.obj("io_service")endpoint = tcp.obj("endpoint", tcp.v4(), 9999)// Acceptor socket - incoming connectionsacceptorSocket = tcp.obj("acceptor", ioService, endpoint)// Active socket - communication with clientactiveSocket = tcp.obj("socket", ioService)// Waiting for connection and use the activeSocket to connect with the clienthelpers.accept(acceptorSocket, activeSocket)// Waiting for messagemessage = helpers.getData(activeSocket);print(format("[Server]: Received \"%s\" from client.", message.getString()))// Send replyreply = "Hello \'" ~ message.getString() ~ "\'!"helpers.sendData(activeSocket, reply)print(format("[Server]: Sent \"%s\" to client.", reply))
```

Client.sil contains the SIL code for the client. As the server, it includes therelevant headers, creates wrappers for the required namespaces and uses them tocreate an endpoint and a socket. Then the client connects to the server, sends amessage, and waits for the server’s reply. Once the reply arrives, the clientprints it to the screen.

```d
// Client.silimport * from silclingimport format from formatcppCompile ("#include <boost/asio.hpp>")cppCompile ("#include \"helper.hpp\"")asio = cppNamespace("boost::asio")tcp = cppNamespace("boost::asio::ip::tcp")helpers = cppNamespace("helpme")// Scope resolution operator <-> address::static_method() or address::static_memberaddress = classScope("boost::asio::ip::address")ioService = asio.obj("io_service")endpoint = tcp.obj("endpoint", address.from_string("127.0.0.1"), 9999)// Creating socketclient_socket = tcp.obj("socket", ioService)// Connectclient_socket.connect(endpoint)message = "demo"helpers.sendData(client_socket, message)print(format("[Client]: Sent \"%s\" to server.", message))message = helpers.getData(client_socket);print(format("[Client]: Received \"%s\" from server.", message.getString()))
```

Output:

```bash
[Client]: Sent "demo" to server.[Server]: Received "demo" from client.[Server]: Sent "Hello demo" to client.[Client]: Received "Hello demo" from server.
```

## Interpreter/Compiler as a Service

The design of Cling, just like Clang, allows it to be used as a library. In thenext example we show how to incorporate libCling in a C++ program. Cling can beused on-demand, as a service, to compile, modify or describe C++ code. Theexample program shows several ways in which compiled and interpreted C++ caninteract:

- `callCompiledFn` – The cling-demo.cpp defines an in global variable, `aGlobal`; a static float variable, `anotherGlobal`; and its accessors. The `interp` argument is an earlier created instance of the Cling interpreter.Just like in standard C++, it is sufficient to forward declare the compiledentities to the interpreter to be able to use them. Then, the executioninformation from the different calls to process is stored in a generic Cling `Value` object which is used to exchange information between compiled andinterpreted code.
- `callInterpretedFn` – Complementing `callCompiledFn`, compiled code cancall an interpreted function by asking Cling to form a function pointer froma given mangled name. Then the call uses the standard C++ syntax.
- `modifyCompiledValue` – Cling has full understanding of C++ and thus we cansupport complex low-level operations on stack-allocated memory. In theexample we ask the compiler for the memory address of the local variable locand ask the interpreter, at runtime, to square its value.

```cpp
// cling-demo.cpp// g++ ... cling-demo.cpp; ./cling-demo#include <cling/Interpreter/Interpreter.h>#include <cling/Interpreter/Value.h>#include <cling/Utils/Casting.h>#include <iostream>#include <string>#include <sstream>/// Definitions of declarations injected also into cling./// NOTE: this could also stay in a header #included here and into cling, but/// for the sake of simplicity we just redeclare them here.int aGlobal = 42;static float anotherGlobal = 3.141;float getAnotherGlobal() { return anotherGlobal; }void setAnotherGlobal(float val) { anotherGlobal = val; }///\brief Call compiled functions from the interpreter.void callCompiledFn(cling::Interpreter& interp) { // We could use a header, too... interp.declare("int aGlobal;\n" "float getAnotherGlobal();\n" "void setAnotherGlobal(float val);\n"); cling::Value res; // Will hold the result of the expression evaluation. interp.process("aGlobal;", &res); std::cout << "aGlobal is " << res.getAs<long long>() << '\n'; interp.process("getAnotherGlobal();", &res); std::cout << "getAnotherGlobal() returned " << res.getAs<float>() << '\n'; setAnotherGlobal(1.); // We modify the compiled value, interp.process("getAnotherGlobal();", &res); // does the interpreter see it? std::cout << "getAnotherGlobal() returned " << res.getAs<float>() << '\n'; // We modify using the interpreter, now the binary sees the new value. interp.process("setAnotherGlobal(7.777); getAnotherGlobal();"); std::cout << "getAnotherGlobal() returned " << getAnotherGlobal() << '\n';}/// Call an interpreted function using its symbol address.void callInterpretedFn(cling::Interpreter& interp) { // Declare a function to the interpreter. Make it extern "C" to remove // mangling from the game. interp.declare("extern \"C\" int plutification(int siss, int sat) " "{ return siss * sat; }"); void* addr = interp.getAddressOfGlobal("plutification"); using func_t = int(int, int); func_t* pFunc = cling::utils::VoidToFunctionPtr<func_t*>(addr); std::cout << "7 * 8 = " << pFunc(7, 8) << '\n';}/// Pass a pointer into cling as a string.void modifyCompiledValue(cling::Interpreter& interp) { int loc = 17; // The value that will be modified // Update the value of loc by passing it to the interpreter. std::ostringstream sstr; // on Windows, to prefix the hexadecimal value of a pointer with '0x', // one need to write: std::hex << std::showbase << (size_t)pointer sstr << "int& ref = *(int*)" << std::hex << std::showbase << (size_t)&loc << ';'; sstr << "ref = ref * ref;"; interp.process(sstr.str()); std::cout << "The square of 17 is " << loc << '\n';}int main(int argc, const char* const* argv) { // Create the Interpreter. LLVMDIR is provided as -D during compilation. cling::Interpreter interp(argc, argv, LLVMDIR); callCompiledFn(interp); callInterpretedFn(interp); modifyCompiledValue(interp); return 0;}
```

Output:

```bash
./cling-demoaGlobal is 42getAnotherGlobal() returned 3.141getAnotherGlobal() returned 1getAnotherGlobal() returned 7.7777 * 8 = 56The square of 17 is 289
```

Crossing the compiled-interpreted boundary relies on the stability of Clang’simplementation of the host’s application binary interface (ABI). Over the yearsit has been very reliable for both Unix and Windows however, Cling is heavilyused to interact with GCC-compiled codebases and is sensitive to ABIincompatibilities between GCC and Clang with respect to the Itanium ABIspecification.

## Extensions

Just like Clang, Cling can be extended by plugins. The next example demonstratesembedded use of Cling’s extension for automatic differentiation, [Clad](https://compiler-research.org/clad/). Clad transforms the clang’s AST toproduce derivatives and gradients of mathematical functions. When creating theCling instance we specify -fplugin and the path to the plugin itself. Then wedefine a target function, pow2, and ask for its derivative with respect to itsfirst argument.

```cpp
#include <cling/Interpreter/Interpreter.h>#include <cling/Interpreter/Value.h>// Derivatives as a service.void gimme_pow2dx(cling::Interpreter &interp) { // Definitions of declarations injected also into cling. interp.declare("double pow2(double x) { return x*x; }"); interp.declare("#include <clad/Differentiator/Differentiator.h>"); interp.declare("auto dfdx = clad::differentiate(pow2, 0);"); cling::Value res; // Will hold the evaluation result. interp.process("dfdx.getFunctionPtr();", &res); using func_t = double(double); func_t* pFunc = res.getAs<func_t*>(); printf("dfdx at 1 = %f\n", pFunc(1));}int main(int argc, const char* const* argv) { std::vector<const char*> argvExt(argv, argv+argc); argvExt.push_back("-fplugin=etc/cling/plugins/lib/clad.dylib"); // Create cling. LLVMDIR is provided as -D during compilation. cling::Interpreter interp(argvExt.size(), &argvExt[0], LLVMDIR); gimme_pow2dx(interp); return 0;}
```

Output:

```bash
./clad-demodfdx at 1 = 2.000000
```

## Conclusion

We have demonstrated Cling’s capabilities for template instantiation on demand;incorporating an interpreter in third-party code; and facilitating interpreterextension. The lazy template instantiation in an embedded interpreter provides aservice which is very suitable for interoperability with C++. Extending such aservice with domain-specific capabilities such as automatic differentiation canbe an key enabler for various science cases and for other broader communities.

## Acknowledgements

The author would like to thank Sylvain Corlay, Simeon Ehrig, David Lange,Chris Lattner, Javier Lopez Gomez, Wim Lavrijsen, Axel Naumann, Alexander Penev,Xavier Valls Pla, Richard Smith, Martin Vassilev, Ioana Ifrim who contributed tothis post.

You can find out more about our activities at [https://root.cern/cling/](https://root.cern/cling/) and [https://compiler-research.org](https://compiler-research.org).
