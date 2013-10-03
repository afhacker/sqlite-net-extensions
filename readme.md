# SQLite-Net Extensions

[SQLite-Net Extensions](https://bitbucket.org/twincoders/sqlite-net-extensions) is a very simple ORM that provides **one-to-one**, **one-to-many**, **many-to-one**, **many-to-many**, **inverse** and **text-blobbed** relationships on top of the [sqlite-net library](https://github.com/praeclarum/sqlite-net).

sqlite-net is an open source, minimal library to allow .NET and Mono applications to store data in [SQLite 3 databases](http://www.sqlite.org). SQLite-Net Extensions extends its funcionality to help the user handle relationships between sqlite-net entities.


## How it works
SQLite-Net Extensions provides attributes for specifying the relationships in different ways and uses reflection to read and write the objects at Runtime.

SQLite-Net Extensions doesn't create any table or columns in your database and doesn't persist obscure elements to handle the database. For this reason, you have full control over the database schema used to persist your entities. SQLite-Net Extensions only requires you to specify the foreign keys used to handle the relationships and it will find out the rest by itself.

SQLite-Net Extensions doesn't modify or override any method behavior of SQLite.Net. Instead, it extends `SQLiteConnection` class with methods to handle the relationships.

For example `GetChildren` finds all the relationship properties that have been specified in the element, finds the required foreign keys and fills the properties automatically for you.

Complementarily `UpdateWithChildren` looks at the relationships that you have set, updates all the foreign keys and save the changes in the database. This is particulary helpful for one-to-many, many-to-many or one-to-one relationships where the foreign key is in the destination table.

You can update foreign keys manually if you feel more comfortable handling some relationships by yourself and let the SQLite-Net extensions handle the rest for you. You can even add or remove SQLite-Net extensions of any project at any time without changes to your database.

## Some code
Theories are for the conspiracists, lets see some code.

#### sqlite-net version
This is how you usually specify a relationship in **sqlite-net** (extracted from [sqlite-net wiki](https://github.com/praeclarum/sqlite-net/wiki/GettingStarted)):

    public class Stock

Then you query like this:

    return db.Query<Valuation> ("select * from Valuation where StockId = ?", stock.Id);
    
#### SQLite-Net Extensions version

With SQLite-Net extensions, no more need to write the queries manually, just specify the relationships in the entities:

    public class Stock

SQLite-Net Extensions will find all the properties with a relationship attribute and then find the foreign keys and inverse attributes for them.

#### Read and write operations
Here's how we'll create, read and update the entities:

    var db = new SQLiteConnection("database.sqlitedb");

Ok, maybe that was a horrible example, because one stock may have many valuations, so it's not a one-to-one, but you get the point.

## Some action
Probably *one-to-one* relationships aren't the reason that you are reading this, so let's prepare a more complete scenario using all the different kind of relationships:

  
    public class Student

Now try to imagine fetching and storing all these entities manually using **sqlite-net**...


## Features
SQLite-Net extensions is built on top of [sqlite-net library](https://github.com/praeclarum/sqlite-net), so obviously all the [features of **sqlite-net**](https://github.com/praeclarum/sqlite-net/wiki) are present in it.

If you don't know why a foreign key or an intermediate table is required at some point or what a relationship represents, take a look at [this article](http://www.onlamp.com/pub/a/onlamp/2001/03/20/aboutSQL.html) that explains pretty simple how relationships are stored in a database.

### One to one
The foreign key for a one-to-one relationship may be defined in any entity or even in **both** entities. In the latter case SQLite-Net extensions will automatically update inverse foreign keys when needed.

The **inverse** relationship for a **one-to-one** property is also a **one-to-one** relationship. SQLite-Net extensions will automatically load one-to-one inverse relationships, because the object is already loaded into memory and it has no DB overhead.

Example:

    public class Passport
        [PrimaryKey, AutoIncrement]

### One to many
The foreign key for a one-to-many relationship must be defined in the *many* end of the relationship.

The **inverse** relationship for a **one-to-many** property is a **many-to-one** relationship. SQLite-Net extensions will automatically load one-to-many inverse relationships, because the object is already loaded into memory and it has no DB overhead.

One-to-many relationships currently support `List` and `Array` of entities. They might be used indistinctly.

Order of the elements is not guarranteed and should be considered as totally random. If sorting is required it should be performed after the elements are loaded.

Example:

    public class Bus

### Many to one
Many-to-one is the opposite to a one-to-many relationship. They represent exactly the same relationship seen from opposite entities. It can also be seen as a one-to-one relationship with no inverse restrictions.

The foreign key for a many-to-one relationship must be defined in the *many* end of the relationship.

The **inverse** relationship for a **many-to-one** property is a **one-to-many** relationship. SQLite-Net extensions will not automatically load many-to-one inverse relationships.

Example:

    public class Bus


### Many to many
Many-to-many relationships cannot be expressed using a foreign key in one of the entities, because foreign keys represent X-to-one relationships. Instead, an intermediate entity is required. This entity is never used directly in the application, but for clarity's shake, SQLite-Net Extensions will never create a table that the user hasn't defined explicitly.

The foreign keys for a many-to-many relationship are thus declared in a intermediate entity.

The **inverse** relationship for a **many-to-many** property is a **many-to-many** relationship. SQLite-Net extensions will not automatically load many-to-many inverse relationships.

Example:

    public class Student

### Inverse relationships
Inverse relationship are automatically discovered on runtime using reflection by matching the type of the origin entity with the type of the relationship of the opposite entity.

You may also explicitly declare the inverse property in the relationship attribute. This is required when more than one relationship with the same type is declared.

Example:

    public class Bus

### Text blobbed properties
Text-blobbed properties are serialized into a text property when saved and deserialized when loaded. This allows storing simple objects in the same table in a single column.

Text-blobbed properties have a small overhead of serializing and deserializing the objects and some limitations, but are the best way to store simple objects like `List` or `Dictionary` of basic types or simple relationships.

Text-blobbed properties require a declared `string` property where the serialized object is stored.

The serializer used to store and load the elements can be customized by implementing the simple `ITextBlobSerializer` interface:

    public interface ITextBlobSerializer

A JSON-based serializer is used if no other serializer has been specified using `TextBlobOperations.SetTextSerializer` method. To use the JSON serializer, a reference to [Newtonsoft Json.Net library](http://james.newtonking.com/projects/json-net.aspx) must be included in the project, also available as a [NuGet package](http://www.nuget.org/packages/newtonsoft.json/).

Text-blobbed properties cannot have relationships to other objects nor inverse relationship to its parent.

Example:

    public class Address
### Foreign keys
Foreign keys for a relationship are discovered on runtime using reflection matching the type of the relationship with the type specified in the `ForeignKey` attribute. 

You may also explicitly declare the foreign key in the relationship attribute. Explicitly declaring the foreign key is required when more than one relationship with the same type is declared.

Foreign key must have the same type as the identifier of the entity that it's referencing.

## Limitations
SQLite-Net it's not a fully featured ORM and because of its conception and implementation, it has some limitations:

#### Only objects with ID can be used in a relationship
If you use `[AutoIncrement]` in your *Primary Key* this also means that you need to insert the object in the database first to be assigned a primary key.

Relationships are based in *foreign keys*. Foreign keys are just a reference to the primary key of another table. If the referenced object doesn't have an assigned primary key, there's no way to store this relationship in the database. We could *dectect* somehow if the object hasn't been persisted yet and then insert it to the database, but we would be losing control and again that is one of our main principles.

Before calling `UpdateWithChildren` make sure you have inserted all referenced objects in the database and you will be fine.

#### Inverse relationships are not loaded automatically on many-to-* relationships
Because of *many-to-many* and *many-to-one* relationships inverse may reference to **a lot** of objects, they have to be loaded manually using `GetChildren` method. Only *one-to-one* and *one-to-many* inverse relationships will be loaded automatically.

#### Foreign keys are not updated automatically for removed references
When you call to `UpdateWithChildren` method, it refreshes all the foreign keys based on the current relationships (including the inverse relationships), and stores the changes in the database. But if you remove a reference to an object, there's no way for SQLite-Net Extensions to know that you have an object in memory that wasn't referenced before. If you later call to `UpdateWithChildren` to the referenced object, you may find that the removed reference is unintentionally restored.

To keep a reference to the removed object it's recommended to reload the referenced object from the database.

## I want [moar](http://slappersonly.com/wordpress/wp-content/uploads/2013/02/moar.jpg)!!
If you want to see some code in action, you can take a look at the Integration Tests. They are packed into a Xamarin.iOS project using NUnit Lite. Run it as a normal Xamarin.iOS App to execute the tests.

> Those are my principles. If you don't like them I have others.


We'd like to hear from you. Don't hesitate in contacting us for suggestions, feature requests, bug reports or anything you need.

## License

Copyright (C) 2013 TwinCoders S.L.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.