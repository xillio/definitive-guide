## Creating Constructs

That should be enough theory now. It's time to get our hands dirty. The first thing we will do is create a simple construct that performs a simple task. Then we will get a little bit more in-depth by doing a little bit more useful operation.

### Hello World

Let's create our `GreetConstruct`. This construct should print **Hello World** you call it.

> **Note:** You should create your constructs in the `constructs` package under the plugin package root. This is where the `XillPlugin` implementation will look for constructs and load them.

```java
package nl.xillio.xill.plugins.guide.constructs;

import ...;

public class GreetConstruct extends Construct {
    public ConstructProcessor prepareProcess(ConstructContext constructContext) {
        return new ConstructProcessor(
                name -> process(constructContext, name),
                new Argument("name", fromValue("World"), ATOMIC)
        );
    }

    private MetaExpression process(ConstructContext constructContext, 
            MetaExpression name) {
        Logger logger = constructContext.getRootLogger();

        logger.info("Hello " + name.getStringValue() + "!");

        return NULL;
    }
}
```

Let's take a step back and see what's going on here. We created a subclass of `Construct` and placed it in the `constructs` subpackage. Next we have to add the `prepareProcess(ConstructContext)` method.
This `ConstructProcessor` contains all the information required for executing the construct. We will pass it a process method and the input parameters.

In the `process(ConstructContext, MetaExpression)` method you can see how we grab the *logger* and use it to print a message. Something you might notice is how we always return `NULL`. This is because every Xill construct requires an output value.

> **Note:** For the more experienced software developers, you might notice that the `GreetConstruct` class is a `ConstructProcessor` factory.

One interesting piece of the code about that you should take a closer look at, is the constructor of the `Argument` class.

```java
new Argument("name", fromValue("World"), ATOMIC)
```

This is where we define how an argument behaves. First we give it a name by passing `"name"` as the first parameter, then we give it a default value by passing a `MetaExpression` as the second parameter. This is optional. And finally we pass any number of `ExpressionDataTypes` that determine what type of data structure you will accept. In our case we accept `ATOMIC` values.
