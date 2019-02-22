# Registering Types with Unity
Unity does not require `Type` to be registered to resolve it. 

Any concrete, constructable `Type` could be resolved by Unity without any prior preparation. It will even create and supply parameters if required. In other words, if you can create a `Type` with `new` operator:
```cs
var value = new SomeClass(new SomeOtherClass());
```
you can resolve it from Unity:
```cs
var value = container.Resolve<SomeClass>();
```
A lot of times, if type does not require any custom steps, it is faster to resolve `Type` from the container if it is not registered. You should only register a `Type` if one of the following is required:

* Lifetime policy other than transient
* Nondefault constructor should be selected
* Properties or fields not marked with appropriate attributes should be injected
* Method should be called during initialization on the created object
* Mapping between service and implementation types 

## What is a Registration and how it works
When you register a `Type`, you are instructing Unity how to create and initialize an instance of that `Type` during instantiation. You also instruct Unity how to deal with the crated instance during its lifetime.

Once registration is complete, Unity creates a blueprint of the type factory where it stores implementation details (name, to and from types, etc.), information about what members to inject and how, and lifetime manager responsible for managing the instance. <br>
At the later time, when that `Type` is requested, Unity uses this blueprint to create a pipeline (resolver delegate) to be used to create type. 

Each Unity container exposes a [collection](xref:Unity.IUnityContainer#Unity_IUnityContainer_Registrations) of available registrations presented as an enumeration of [IContainerRegistration](xref:Unity.IContainerRegistration) objects. This collection could be used to filter and select certain registrations as well as to [check if the `Type` is registered](xref:#Unity.IUnityContainer#Unity_IUnityContainer_IsRegistered_System_Type_System_String_) and how.

## Different types of registrations
Unity recognizes three different scenarios of how instances and types are created:
#### Instances created outside of Unity
A lot of times parts of system's infrastructure require to be available to clients and services of the application. These entities are created and managed outside of Unity but should be accessible by consumers of the framework during subsequent resolutions. <br>
To enable access to these objects Unity provides a way to register instances. For more information see [Instance Registration](instance.md).
#### Instances created by registered Type factories
Although Unity is quite capable in creating instance of types, sometimes it is more efficient or desireable to use `Type` factories. Unity satisfies this requirement by providing API to register a special factory delegate. For more information see [Factory Registration](factory.md)  
#### Instances created by Unity
This is the most common scenario where objects are created by the container itself. Creation process is controlled by either `Type` registrations associated with the container or implicit Unity defaults if none registered. For more information how `Type` can be registered see [Type Registration](type.md)

## Registration Hierarchies
Unity container provides a way to create child containers (other systems refer to it as resolution scopes) and allows building sophisticated scope trees of registrations. There are just a few simple rules to follow when dealing with container hierarchies:

* **Types registered in predecessor containers always available in descendant containers**<br>
This is a very simple concept, each registration is like a public virtual declaration in cs types. Every descendant inherits it and can use at will.

* **Types registered in descendant containers override the same registration of predecessors**<br>
Following the same analogy with public virtual declarations, each override registration installs its own declaration and hides the one in predecessor containers.