## MetaExpression

Before we create a construct let's take a look at the most important class in the Xill API that you will need to understand. The `MetaExpression` represents all expressions in Xill. It is the main container of data.

### Expression Types in Xill

We do not want to bother the Xill programmer with typing. This means that the typing system used by Xill might not be entirely intuitive for you if you are more used Java's static type system.

The way we solve the problem is by implementing a conversion for every type in the Xill language to an other type. As a result you can interpret most expressions as the type you want them to be.

For example. Say I have the expression `"300.5"`. Clearly this is a `String`. But, yet if you pass this value to the `Math.round` construct you will notice that it will still work. This is because the MetaExpression allows you to call the `MetaExpression#getNumberValue()` method which formats that string to a Java `Number`.

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
| Java                   | `ExpressionBuilderHelper.fromValue(true)`          |

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
| Xill                   | `true`                                             |
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

The expressions that we looked at before can be arranged in different structures. In Xill we support three basic structures: `ATOMIC`, `LIST` and `OBJECT`.

#### ATOMIC
The `ATOMIC` structure is the most basic. It represents a single expression. Some examples could be `"Hello World"`, `true` and `null`.

#### LIST

The `LIST` structure represents a collection of expressions in a fixed order. These expressions can by of any type or structure.

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

To create an `OBJECT` in Xill you use the *brace notation*.

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
