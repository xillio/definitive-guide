# Advanced Topics

Up until now the examples used in this guide have been straightforward to avoid any complex structures. In this chapter we will take a look as some of these complex structures and provide examples on how to use advanced features.

## Passing Data Between Constructs

Xill is not an object oriented language, but sometimes you still want to work with an object oriented model in the back-end. This is possible through what we call the `MetadataExpression` (don't confuse it with `MetaExpression`).
This `MetadataExpression` represents an empty marker interface that allows you to add hidden Java objects to a Xill expression. For instance: If I make a connection to a database that I want to keep alive I can create a `MyDB.connect` construct that will return a connection object. This object will look like a string such as `"user@host:database_name"` but in the construct we will push the actual Java  connection into the expression. After that we can pull this connection object from the expression and use it in other constructs in the package.

```
use MyDB;

// Use a construct to build the connection object
var connection = MyDB.connect("localhost");


```

## Testing Your Plugin

