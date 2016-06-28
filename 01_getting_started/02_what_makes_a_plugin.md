## What Makes a Plugin

Let's take a more detailed look at the things a plugin contains. We saw earlier that it contains a *package* and some *constructs*, but there is a little bit more to it than that. If you want to know how to set up these specific parts then skip to [Developing a Plugin](#developing-a-plugin).

### Package Declaration

The first part of a plugin is its *package declaration*. This declaration specifies where the Processor can find the plugin so the framework can load it. If you use our maven build this declaration will be automatically generated.

### Package Implementation

The second part is the actual implementation of the package. This implementation is the class referenced by the package declaration and must be a subclass of the `XillPlugin` class.

### Constructs

Now strictly speaking we do not need *constructs* to form a plugin but without any your plugin will be boring. These *constructs* are implementations of the abstract `Construct` class. They provide a way for the processor module to perform any operation. The package implementation loads them and keeps in the same sandbox as the other constructs in that package.
