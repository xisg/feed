---
title: 'The Upspin manifesto: On the ownership and sharing of data'
url: https://commandcenter.blogspot.com/2017/10/the-upspin-manifesto-on-ownership-and.html
published: "2017-10-26T20:00:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-195062566919629025
---

# The Upspin manifesto: On the ownership and sharing of data

Here follows the original "manifesto" from late 2014 proposing the idea for what became [Upspin](https://upspin.io/). The text has been lightly edited to remove a couple of references to Google-internal systems, with no loss of meaning.

I'd like to thank Eduardo Pinheiro, Eric Grosse, Dave Presotto and Andrew Gerrand for helping me turn this into a working system, in retrospect remarkably close to the original vision.

[![Augie](https://upspin.io/images/augie.jpg)](https://upspin.io/images/augie.jpg)Augie image Copyright © 2017 Renee French

### **Manifesto**

Outside our laptops, most of us today have no shared file system at work. (There was a time when we did, but it's long gone.) The world took away our /home folders and moved us to databases, which are not file systems. Thus I can no longer (at least not without clumsy hackery) make my true home directory be where my files are any more. Instead, I am expected to work on some local machine using some web app talking to some database or other external repository to do my actual work. This is mobile phone user interfaces brought to engineering workstations, which has its advantages but also some deep flaws. Most important is the philosophy it represents.

You don't own your data any more. One argument is that companies own it, but from a strict user perspective, your "apps" own it. Each item you use in the modern mobile world is coupled to the program that manipulates it. Your Twitter, Facebook, or Google+ program is the only way to access the corresponding feed. Sometimes the boundary is softened within a company—photos in Google+ are available in other Google products—but that is the exception that proves the rule. _You_ don't control your data, the programs do.

Yet there are many reasons to want to access data from multiple programs. That is, almost by definition, the Unix model. Unix's model is largely irrelevant today, but there are still legitimate ways to think about data that are made much too hard by today's way of working. It's not necessarily impossible to share data among programs (although it's often very difficult), but it's never _natural_. There are workarounds like plugin architectures and documented workflows but they just demonstrate that the fundamental idea—sharing data among programs—is not the way we work any more.

This is backwards. It's a reversal of the old way we used to work, with a file system that all programs could access equally. The very notion of "download" and "upload" is plain stupid. Data should simply be available to any program with authenticated access rights. And of course for any person with those rights. Sharing between programs and people can be, technologically at least, the same problem, and a solved one.

This document proposes a modern way to achieve the good properties of the old world: consistent access, understandable data flow, and easy sharing without workarounds. To do this, we go back to the old notion of a file system and make it uniform and global. The result should be a data space in which all people can use all their programs, sharing and collaborating at will in a consistent, intuitive way.

Not downloading, uploading, mailing, tarring, gzipping, plugging in and copying around. Just using. Conceptually: If I want to post a picture on Twitter, I just name the file that holds it. If I want to edit a picture on Twitter using Photoshop, I use the File>Open menu in Photoshop and name the file stored on Twitter, which is easy to discover or even know a priori. (There are security and access questions here, and we'll come back to those.)

**Working in a file system.**

I want my home directory to be where all my data is. Not just my local files, not just my source code, not just my photos, not just my mail. _All_ my data. My "phone" should be able to access the same data as my laptop, which should be able to access the same data as the servers. (Ignore access control for the moment.) $HOME should be my home for everything: work, play, life; toy, phone, work, cluster.

This was how things worked in the old single-machine days but we lost sight of that when networking became universally available. There were network file systems and some research systems used them to provide basically this model, but the arrival of consumer devices, portable computing, and then smartphones eroded the model until every device is its own fiefdom and every program manages its own data through networking. We have a network _connecting_ devices instead of a network _composed_ of devices.

The knowledge of how to achieve the old way still exists, and networks are fast and ubiquitous enough to restore the model. From a human point of view, the data is all we care about: my pictures, my mail, my documents. Put those into a globally addressable file system and I can see all my data with equal facility from any device I control. And then, when I want to share with another person, I can name the file (or files or directory) that holds the information I want to share, grant access, and the other person can access it.

The essence here is that the data (if it's in a single file) has one name that is globally usable to anyone who knows the name and has the permission to evaluate it. Those names might be long and clumsy, but simple name space techniques can make the data work smoothly using local aliasing so that I live in "my" world, you live in your world (also called "my" world from your machines), and the longer, globally unique names only arise when we share, which can be done with a trivial, transparent, easy to use file-system interface.

Note that the goal here is not a new file system to use alongside the existing world. Its purpose is to be the only file system to use. Obviously there will be underlying storage systems, but from the user's point of view all access is through this system. I edit a source file, compile it, find a problem, point a friend at it; she accesses _the same file_, not a copy, to help me understand it. (If she wants a copy, there's always cp!).

This is not a simple thing to do, but I believe it is possible. Here is how I see it being assembled. This discussion will be idealized and skate over a lot of hard stuff. That's OK; this is a vision, not a design document.

**Everyone has a name.**

Each person is identified by a name. To make things simple here, let's just use an e-mail address. There may be a better idea, but this is sufficient for discussion. It is not a problem to have multiple names (addresses) in this model, since the sharing and access rights will treat the two names as distinct users with whatever sharing rights they choose to use.

Everyone has stable storage in the network.

Each person needs a way to make data accessible to the network, so the storage must live in the network. The easiest way to think of this is like the old network file systems, with per-user storage in the server farm. At a high level, it doesn't matter what that storage is or how it is structured, as long as it can be used to provide the storage layer for a file-system-like API.

Everyone's storage server has a name identified by the user's name.

The storage in the server farm is identified by the user's name.

Everyone has local storage, but it's just a cache.

It's too expensive to send all file access to the servers, so the local device, whatever it is—laptop, desktop, phone, watch—caches what it needs and goes to the server as needed. Cache protocols are an important part of the implementation; for the point of this discussion, let's just say they can be designed to work well. That is a critical piece and I have ideas, but put that point aside for now.

The server always knows what devices have cached copies of the files on local storage.

The cache always knows what the associated server is for each directory file in its cache and maintains consistency within reasonable time boundaries.

The cache implements the API of a full file system. The user lives in this file system for all the user's own files. As the user moves between devices, caching protocols keep things working.

**Everyone's cache can talk to multiple servers.**

A user may have multiple servers, perhaps from multiple providers. The same cache and therefore same file system API refers to them all equivalently. Similarly, if a user accesses a different user's files, the exact same protocol is used, and the result is cached in the same cache the same way. This is federation as architecture.

**Every file has a globally unique name.**

Every file is named by this triple: (host address, user name, file path). Access rights aside, any user can address any other user's file by evaluating the triple. The real access method will be nicer in practice, of course, but this is the gist.

**Every file has a potentially unique ACL.**

Although the user interface for access control needs to be very easy, the effect is that each file or directory has an access control list (ACL) that mediates all access to its contents. It will need to be very fine-grained with respect to each of users, files, and rights.

**Every user has a local name space.**

The cache/file-system layer contains functionality to bind things, typically directories, identified by such triples into locally nicer-to-use names. An obvious way to think about this is like an NFS mount point for /home, where the remote binding attaches to /home/XXX the component or components in the network that the local user wishes to identify by XXX. For example, Joe might establish /home/jane as a place to see all the (accessible to Joe) pieces of Jane's world. But it can be much finer-grained than that, to the level of pulling in a single file.

The NFS analogy only goes so far. First, the binding is a lazily-evaluated, multi-valued recipe, not a Unix-like mount. Also, the binding may itself be programmatic, so that there is an element of auto-discovery. Perhaps most important, one can ask any file in the cached local system what its triple is and get its unique name, so when a user wishes to share an item, the triple can be exposed and the remote user can use her locally-defined recipe to construct the renaming to make the item locally accessible. This is not as mysterious or as confusing in practice as it sounds; Plan 9 pretty much worked like this, although not as dynamically.

**Everyone's data becomes addressable.**

Twitter gives you (or anyone you permit) access to your Twitter data by implementing the API, just as the regular, more file-like servers do. The same story applies to any entity that has data it wants to make usable. At some scaling point, it becomes wrong not to play.

**Everyone's data is secure.**

It remains to be figured out how to do that, I admit, but with a simple, coherent data model that should be achievable.

**Is this a product?**

The protocols and some of the pieces, particularly what runs on local devices, should certainly be open source, as should a reference server implementation. Companies should be free to provide proprietary implementations to access their data, and should also be free to charge for hosting. A cloud provider could charge hosting fees for the service, perhaps with some free or very cheap tier that would satisfy the common user. There's money in this space.

**What is this again?**

What Google Drive should be. What Dropbox should be. What file systems can be. The way we unify our data access across companies, services, programs, and people. The way I want to live and work.

Never again should someone need to manually copy/upload/download/plugin/workflow/transfer data from one machine to another.
