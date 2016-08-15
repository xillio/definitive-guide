# Developing a Plugin

In this chapter we will get to work and actually write some code. We will start by setting up a project and then get to making the actual plugin. 

You can do this with any IDE of your choice. As long as it has maven integration.

## Setting up the Project

Let's get our hands dirty and prepare a project for our code. The main supported way of building your plugin is to use *Maven*. Please note that the examples below might not use the latest versions of the Xill API.

If you get a little confused or you want to see an example you can head over to this guide's [GitHub](https://github.com/xillio/definitive-guide/tree/master/project-example) repository. There you will find a Maven project that supports this guide.

### Step 1: Setting up the build

1. Make sure you have [Maven 3+](https://maven.apache.org/download.cgi) installed or you are running an 
   IDE with an embedded maven installation.
2. Create a `pom.xml` file containing the configuration below.

    ```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                       http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <parent>
            <groupId>nl.xillio.xill</groupId>
            <artifactId>xill-plugins-parent</artifactId>
            <!-- Replace this with the latest API version -->
            <version>3.04.00</version>
        </parent>

        <artifactId>plugin-guide</artifactId>

        <repositories>
          <repository>
            <id>jcenter</id>
            <url>https://jcenter.bintray.com</url>
          </repository>
          <repository>
            <id>xillio-release</id>
            <url>https://maven.xillio.com/artifactory/libs-release</url>
          </repository>
        </repositories>
        
        <dependencies>
            <!-- Here you declare your maven dependencies -->
        </dependencies>

        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-assembly-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>	
    ```

3. Import your project into your IDE.

### Step 2: Package Implementation

Now to create your plugin you should create a subclass of the `XillPlugin` class. The Maven build will automatically pick it up and present it to the plugin framework.

```java
package nl.xillio.xill.plugins.guide;

import nl.xillio.plugins.XillPlugin;

/**
 * This class represents the Guide Xill package.
 */
public class GuideXillPlugin extends XillPlugin {

}

```

This class is empty for now because we do not have any configuration that we have to do here.

## The Project Structure

A plugin exists within its own package. In the example above we created the `nl.xillio.xill.plugins.guide` package which is the root for the `GuideXillPlugin`. Later we will be adding *constructs* to this package. If we place them in the `nl.xillio.xill.plugins.guide.constructs` package they will be automatically loaded by the abstract `XillPackage` implementation.

This concludes the initial project setup. We are now ready to start adding functionality to the plugin.

## MetaExpression

Before we create a construct let's take a look at the most important class in the Xill API. The `MetaExpression` represents all expressions in Xill. It is the main container of data.

### Expression Types in Xill

We do not want to bother the Xill programmer with variable types. This means that the type system used by Xill might not be entirely intuitive for you if you are more used Java's static type system.

The way we solve this problem is by implementing a conversion for every type in the Xill language to another type. As a result you can interpret most expressions as the type you want them to be.

For example. Say I have the expression `"300.5"`. Clearly this is a `String`. Yet, if you pass this value to the `Math.round` construct you will notice that it will work regardless. This is because the MetaExpression allows you to call the `MetaExpression#getNumberValue()` method which formats that string to a Java `Number`.

Of course conversion is not always possible. If I have the expression `"Hello World"` then when I call `getNumberValue()`, the return value will be `Double.NaN`.

Let's take a look at the different internal types you'll run into when working with the MetaExpression.

#### String

You can build a `String` `MetaExpression` from either Java or Xill.

| Create expression from | Code                                               |
| ---------------------- | -------------------------------------------------- |
| Xill                   | `"Hello World"`                                    |
| Java                   | `ExpressionBuilderHelper.fromValue("Hello World")` |

It will convert to almost all other types.

| **Expression**          | `"2356264"`     | `""`          | `"Hello World"`  |
| ----------------------- | --------------- | ------------- | ---------------- |
| **Description**         | A Number String | Empty String  | Any Other String |
| **`getBooleanValue()`** | `true`          | `false`       | `true`           |
| **`getNumberValue()`**  | `"2356264"`     | `Double.NaN`  | `Double.NaN`     |
| **`getBinaryValue()`**  | No Data         | No Data       | No Data          |

#### Boolean

You can build a `Boolean` `MetaExpression` from either Java or Xill.

| Create expression from | Code                                               |
| ---------------------- | -------------------------------------------------- |
| Xill                   | `true`                                             |
| Java                   | `ExpressionBuilderHelper.fromValue(true)`        |

It will convert to almost all other types.

| **Expression**          | `true`   | `false`   |
| ----------------------- | -------- | --------- |
| **Description**         | True     | False     |
| **`getStringValue()`**  | `"true"` | `"false"` |
| **`getNumberValue()`**  | `1`      | `0`       |
| **`getBinaryValue()`**  | No Data  | No Data   |

#### Number

You can build a `Number` `MetaExpression` from either Java or Xill.

| Create expression from | Code                                               |
| ---------------------- | -------------------------------------------------- |
| Xill                   | `5246.3`                                             |
| Java                   | `ExpressionBuilderHelper.fromValue(5246.3)`        |

It will convert to almost all other types.

| **Expression**          | `153`      | `0`     | `23.3`   |
| ----------------------- | ---------- | ------- | -------- |
| **Description**         | An Integer | Zero    | A Double |
| **`getStringValue()`**  | `"153"`    | `"0"`   | `"23.3"` |
| **`getBooleanValue()`** | `true`     | `false` | `true`   |
| **`getBinaryValue()`**  | No Data    | No Data | No Data  |


#### Binary

The binary data type is special. It represents a source or target of streamed data. An example would be the result of the `File.openRead` construct. 

| Create expression from | Code                                                         |
| ---------------------- | ------------------------------------------------------------ |
| Xill                   | `File.openWrite`                                             |
| Java                   | `ExpressionBuilderHelper.fromValue(new SimpleIOStream(...))` |

It will convert to almost all other types.

| **Expression**          | Any Stream                   |
| ----------------------- | ---------------------------- |
| **Description**         | Any Stream                   |
| **`getStringValue()`**  | Source or Target Description |
| **`getBooleanValue()`** | `true`                       |
| **`getNumberValue()`**  | `Double.NaN`                 |


### Data Structures in Xill

We can arrange these expressions in different ways. In Xill we support three basic structures: `ATOMIC`, `LIST` and `OBJECT`.

#### ATOMIC
The `ATOMIC` structure is the most basic. It represents a single expression. Some examples could be `"Hello World"`, `true` and `null`.

#### LIST

The `LIST` structure represents a collection of expressions in a fixed order. These expressions can be of any type or structure.

> **Note:** The Xill `LIST` works like `java.util.List` but has a nice syntax wrapped around it.

To create a list in Xill you use the *bracket notation*.

```javascript
// Create the list
var myList = [
    0,
    2,
    "Hello World",
    [1,2,3]
];

// Add an item to the end of the list
myList[] = 5;

// Set an item on a specific index
myList[1] = 5;
```

To create a `LIST` in Java you call `fromValue` with a `List<MetaExpression>` as the value.

```java
// Create list elements
MetaExpression item1 = fromValue(0);
MetaExpression item2 = fromValue("Element");

// Create the list
List<MetaExpression> list = new ArrayList<>();
list.add(item1);
list.add(item2);

// Build the expression
MetaExpression listExpression = ExpressionBuilderHelper.fromValue(list);
```

#### OBJECT

The `OBJECT` structure is like `LIST` but has `String` indexes.

> **Note:** The Xill `OBJECT` works like `java.util.Map` but attempts to preserve the insertion order. For this reason you have to pass the concrete `LinkedHashMap` instead of a `Map` when building the expression.

To create an `OBJECT` in Xill you use the *brace notation* (you might recognize this as JSON).

```javascript
// Create an object
var myObject = {
    "message": "Hello World"
};

// Add or override elements
myObject.more = "More...";
```

To create an `OBJECT` in Java you call `fromValue` with a `LinkedHashMap<String, MetaExpression>` as the value.

```java
// Create the elements
MetaExpression item1 = fromValue(0);
MetaExpression item2 = fromValue("Element");

// Create the map
LinkedHashMap<String, MetaExpression> map = new LinkedHashMap<>();
map.put("message", item2);
map.put("count" , item1);

// Build the expression
MetaExpression objectExpression = ExpressionBuilderHelper.fromValue(map);
```
## Creating Constructs

That should be enough theory for now. It's time to get our hands dirty. The first thing we will do is create a construct that performs a simple task. Then we will get a little bit more in-depth by doing a little bit more useful operation.

### Hello World

Let's create our `GreetConstruct`. This construct should print **Hello World** when you call it.

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
        
        /*
         Note that here we return NULL and not null.
         NULL represents a null value in Xill, null 
         represents a null value in Java.
        */
        return NULL;
    }
}
```

Let's take a step back and see what's going on here. We created a subclass of `Construct` and placed it in the `constructs` subpackage. Next we have to add the `prepareProcess(ConstructContext)` method.
It returns a  `ConstructProcessor`, which contains all the information required for executing the construct. We will pass it a process method and the input parameters.

In the `process(ConstructContext, MetaExpression)` method you can see how we grab the *logger* and use it to print a message. Something you might notice is how we always return `NULL`. This is because every Xill construct requires an output value.

> **Note:** For the more experienced software developers, you might notice that the `GreetConstruct` class is a `ConstructProcessor` factory.

One interesting piece of the code that you should take a closer look at, is the constructor of the `Argument` class.

```java
new Argument("name", fromValue("World"), ATOMIC)
```

This is where we define how an argument behaves. First we give it a name by passing `"name"` as the first parameter, then we give it a default value by passing a `MetaExpression` as the second parameter. This is optional. And finally we pass any number of `ExpressionDataTypes` that determine what type of data structure you will accept. In our case we accept `ATOMIC` values.

### Performing an Operation

The example above only demonstrates how to create your construct. It does not do anything that would be considered useful, so let's create a construct that is a little bit more advanced. Let's download a resource from the internet and return it to the user. 

The resource we will be downloading is a web api that returns yes, no or maybe with a random distribution. We can find it on [https://apis.rtainc.co/twitchbot/8ball](https://apis.rtainc.co/twitchbot/8ball). Go ahead, try it in your browser. You will find that every time you refresh your browser you will see a random response. This is exactly what we will make our next construct do.

First off we create the `EightBallConstruct` class like we did before with the `GreetConstruct`.

```java
public class EightBallConstruct extends Construct {
  @Override
  public ConstructProcessor prepareProcess() {
    return new ConstructProcessor(
      this::process
    );
  }
  
  private MetaExpression process() {
    ...
  }
}
```

Now we have to perform the download itself. To do this we will add another private method which returns a `String` and then we will parse that result to a `MetaExpression` in the `process` method.

```java
private String get8Ball() {
  try (InputStream stream = new URL(¨https://apis.rtainc.co/twitchbot/8ball¨).openStream()) {
    return IOUtils.toString(stream);
  } catch(IOException e) {
    throw new OperationFailedException(
      "download the ´future´",
      e.getMessage(),
      e
    );
  }
}

private MetaExpression process() {
  return fromValue(get8Ball());
}
```

Now let's take a look at how our constructs work in Xill IDE. To create the required jar file you can run the *package* phase in maven by entering `mvn package` on the command line or running the phase from your IDE. You should now have a jar file in your `target` folder. Copy that to the `plugins` folder in your installation of Xill IDE.

You can run the following script to verify your construct.

```
use System, Guide;

System.print("Does my construct work correctly?");
System.print(Guide.eightBall());
```
If everything went well, you will see two messages in your console: "Does my construct work correctly?" and a response from the 8-ball.

## Documenting Constructs

In Xill IDE you have access to a documentation section. This is where you can find documentation for all loaded constructs. To make your construct visible in there you can create an xml file that matches the name of the construct. For example, if you have a construct at `io.xill.guide.constructs.GreetConstruct` you can create an xml file at `/io/xill/guide/constructs/GreetConstruct.xml` and the documentation generator will load the file.

Below you can find a fully expressed example of the documentation model.

```xml
<?xml version="1.0" encoding="utf-8"?>
<function>
    <description>
Returns true if the value is contained in the given list or object. Otherwise false.
    </description>
    <examples>
        <example title="Usage">
            <code>
                use Collection;

                var list = ["a", "b", "c"];
                Collection.contains(list, "b"); // TRUE
                Collection.contains(list, "e"); // FALSE

                var object = {1:"a", 2:"b", 3:"c"};
                Collection.contains(object, "b"); // TRUE
                Collection.contains(object, "e"); // FALSE
            </code>
        </example>
    </examples>
    <searchTags>
        contains, value, filter, list, object
    </searchTags>
    <references>
      <reference>remove</reference>
    </references>
</function>
```

There are several components in this xml file so let's walk through them.

### Description

In the description section we have the main body of the documentation. Here you should describe how your construct works and what valid input and output is. You can go into great detail here and use markdown headings and tables to make your documentation more readable.

### Examples

It is extremely useful to a Xill user to have proper code examples. Not unlike this guide, you can read all the documentation you want but until you see some context it will be a lot more clear.

In this section you have two tools at your disposal: header and code. The code element should contain some Xill code that describes how to use that specific construct. The header tag can by used to give a title to a piece of code.

### Search Tags

Above the documentation panel there is a search box. To allow for better results you can add some search tags to your documentation. If any of those tags are then present in the search box, your construct will pop up.

### References

And finally, if you have constructs that work as a group or that are similar to each other you can reference them here. This will add a link to the documentation page to the referenced construct. You reference a construct by its name (like `System.print`). You can leave out the package name if you are referencing a construct in the same package as the documented construct.

## Using Services

In the `EightBallConstruct` example above we put all the functionality for the construct in a single spot. If we would like to create another construct that fetches a resource using http, we would have to duplicate our code.

A good way to combat code duplication like that is by using services. A service is a class that is responsible for performing a collection of specific operations. Let's take a look a service that performs an http request.

```java
public class HttpService {
   private String get(String url) {
    try (InputStream stream = new URL(url).openStream()) {
      return IOUtils.toString(stream);
    } catch(IOException e) {
      throw new OperationFailedException(
        "get " + url,
        e.getMessage(),
        e
      );
    }
  }
}
```

This small service fetches a resource from the internet using an http get request. We can now use this in our constructs. To do this we make use of [Guice](https://github.com/google/guice/wiki).

```java
public class EightBallConstruct extends Construct {
  private final HttpService httpService;
  
  // Note the @Inject annotation to instruct guice to use this constructor
  @Inject
  public EightBallConstruct(HttpService httpService) {
   this.httpService = httpService;
  }
  
  @Override
  public ConstructProcessor prepareProcess() {
    return new ConstructProcessor(
      this:process
    );
  }
  
  private MetaExpression process() {
    String result = httpService.get("https://apis.rtainc.co/twitchbot/8ball");
    return fromValue(result);
  }
}
```

Using services is a simple as that. Of course there are more advanced scenarios. For these I would like to refer you to the [Guice Wiki](https://github.com/google/guice/wiki).
