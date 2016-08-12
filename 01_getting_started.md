# Getting Started

In this chapter we will give an overview of the Xill platform and what components it consists of. This way we can gain a better understanding of what the responsibility of the plugin is and how we can maximize its potential.

## Components of Xill

![The Xill Ecosystem](images/xill_components.png)

At the core of Xill is an extensible framework with modules that have clear responsibilities. 

### Language

The language module defines the Xill syntax and converts scripts to token trees. These trees contain all tokens required for processing. This module does no processing other than syntax and sanity checking of the scripts.

### Processor

The processor module is where the magic happens. This module executes scripts. It will take a syntax tree and parse it into an object which can be processed. It manages all resources and handles errors and debugging.

### API

The API contains all shared classes. This module provide a shared interface for plugins to talk to. It contains essential classes like the [`MetaExpression`](#metaexpression) and all the shared data types like XML, Date and more.

### Plugins

Now this is what this guide is about. The plugins add actual functionality to Xill. They allow calls to other Java libraries or perform specific operations. These plugins consist of a collection of *constructs* available to the Xill programmer.

A construct in Xill is a function that takes a fixed amount of expressions as input and performs a specific operation. All these constructs can communicate with each other in their own sandbox inside the plugin package. This means the will not affect *constructs* in other plugins.

> **Note:** For the more experienced Java developers, when we talk about a sandbox what we really mean is that the plugins have their own class loader.

Xill processor module manages these plugins.

```javascript
use System;
// In this case `System` is a plugin package.

// And System.print is a construct.
System.print("Hello World");
```

## What Makes a Plugin

Let's take a more detailed look at the things a plugin contains. We saw earlier that it contains a *package* and some *constructs*, but there is a little bit more to it than that. If you want to know how to set up these specific parts then skip to [Developing a Plugin](#developing-a-plugin).

### Package Declaration

The first part of a plugin is its *package declaration*. This declaration specifies where the Processor can find the plugin so the framework can load it. If you use our maven build this declaration will be automatically generated.

### Package Implementation

The second part is the actual implementation of the package. This implementation is the class referenced by the package declaration and must be a subclass of the `XillPlugin` class.

### Constructs

Now strictly speaking we do not need *constructs* to form a plugin but without any your plugin will be boring. These *constructs* are implementations of the abstract `Construct` class. They provide a way for the processor module to perform any operation. The package implementation loads them and keeps in the same sandbox as the other constructs in that package.
