minimal-json
============

[![License](https://img.shields.io/github/license/ralfstx/minimal-json.svg)](https://github.com/ralfstx/minimal-json/blob/master/LICENSE)
[![Maven Central](https://img.shields.io/maven-central/v/com.eclipsesource.minimal-json/minimal-json.svg)](http://search.maven.org/#search|ga|1|g%3A%22com.eclipsesource.minimal-json%22%20a%3A%22minimal-json%22)
[![Build Status](https://img.shields.io/travis/ralfstx/minimal-json.svg)](http://travis-ci.org/ralfstx/minimal-json)

A fast and minimal JSON parser and writer for Java.
It's not an object mapper, but a bare-bones library that aims at being

* **minimal**: no dependencies, single package with just a few classes, small download size (< 25kB)
* **fast**: high performance comparable with other state-of-the-art parsers (see below)
* **leightweight**: object representation with minimal memory footprint (e.g. no HashMaps)
* **simple**: reading, writing and modifying JSON with minimal code (short names, fluent style)

Minimal-json is fully covered by unit tests, and field-tested by the [Eclipse RAP project](http://eclipse.org/rap) and others (see below). The JAR contains a **valid OSGi** bundle manifest and can be used in OSGi environments without modifications.

Code Examples
-------------

### Read JSON from a String or a Reader

Reading is buffered already, so you *don't* need to wrap your reader in a BufferedReader.

```java
JsonObject jsonObject = JsonObject.readFrom( string );
JsonArray jsonArray = JsonArray.readFrom( reader );
```

### Access the contents of a JSON object

The `get` method returns a `JsonValue` if the element exists. Depending on the type, you can get corresponding Java values using the methods `asString`, `asBoolean`, etc.

```java
String name = jsonObject.get( "name" ).asString();
int age = jsonObject.get( "age" ).asInt(); // asLong(), asFloat(), asDouble(), ...
```

you can use a default value if the element does not exist:

```java
String name = jsonObject.getString( "name", "unknown" );
int age = jsonObject.getInt( "age", -1 );
```

you can also iterate over the members:

```java
for( Member member : jsonObject ) {
  String name = member.getName();
  JsonValue value = member.getValue();
  // ...
}
```

### Access the contents of a JSON array

```java
String name = jsonArray.get( 0 ).asString();
int age = jsonArray.get( 1 ).asInt(); // asLong(), asFloat(), asDouble(), ...

// or iterate over the values:
for( JsonValue value : jsonArray ) {
  // ...
}
```

### Access nested contents

```java
// Example: { "friends": [ { "name": "John", "age": 23 }, ... ], ... }
JsonArray friends = jsonObject.get( "friends" ).asArray();
String name = friends.get( 0 ).asObject().get( "name" ).asString();
int age = friends.get( 0 ).asObject().get( "age" ).asInt();
```

### Create JSON objects and arrays

```java
JsonObject jsonObject = new JsonObject().add( "name", "John" ).add( "age", 23 );
// -> { "name": "John", "age", 23 }

JsonArray jsonArray = new JsonArray().add( "John" ).add( 23 );
// -> [ "John", 23 ]
```

### Modify JSON objects and arrays

```java
jsonObject.set( "age", 24 );
jsonArray.set( 1, 24 ); // access element by index

jsonObject.remove( "age" );
jsonArray.remove( 1 );
```

### Write JSON to a Writer or a String

These methods can be used on all JsonValues:

```java
jsonValue.writeTo( writer );
String json = jsonValue.toString();
```

Enable formatted output:

```java
jsonValue.writeTo( writer, WriterConfig.PRETTY_PRINT );
String json = jsonValue.toString( WriterConfig.PRETTY_PRINT );
```

Concurrency
-----------

The JSON structures in this library (`JsonObject` and `JsonArray`) are deliberately **not thread-safe** to keep them fast and simple. In the rare case that JSON data structures must be accessed from multiple threads, while at least one of these threads modifies the contents, the application must ensure proper synchronization.

Iterators will throw a `ConcurrentModificationException` when the contents of
a JSON structure have been modified after the creation of the iterator.

Performance
-----------

I've spent days benchmarking and tuning minimal-json's reading and writing performance. You can see from the charts below that it performs quite reasonably. But damn, it's not quite as fast as Jackson! However, given that Jackson is a very complex machine compared to minimal-json, I think for the minimalism of this parser, the results are quite good.

Below is the result of a performance comparison with other parsers, namely
[org.json](http://www.json.org/java/index.html) 20141113,
[Gson](http://code.google.com/p/google-gson/) 2.3.1,
[Jackson](http://wiki.fasterxml.com/JacksonHome) 2.5.0, and
[JSON.simple](https://code.google.com/p/json-simple/) 1.1.1.
In this benchmark, two example JSON texts are parsed into a Java object and then serialized to JSON again.
All benchmarks can be found in [com.eclipsesource.json.performancetest](https://github.com/ralfstx/minimal-json/tree/master/com.eclipsesource.json.performancetest).

[rap.json](https://github.com/ralfstx/minimal-json/blob/master/com.eclipsesource.json.performancetest/src/main/resources/input/rap.json), ~ 30kB

![Read/Write performance compared to other parsers](https://raw.github.com/ralfstx/minimal-json/master/com.eclipsesource.json.performancetest/performance-rap.png "Read/Write performance compared to other parsers")

[caliper.json](https://github.com/ralfstx/minimal-json/blob/master/com.eclipsesource.json.performancetest/src/main/resources/input/caliper.json), ~ 83kB

![Read/Write performance compared to other parsers](https://raw.github.com/ralfstx/minimal-json/master/com.eclipsesource.json.performancetest/performance-caliper.png "Read/Write performance compared to other parsers")

Who is using minimal-json?
--------------------------

Here are some projects that use minimal-json:

* [Hazelcast](http://hazelcast.org/)
* [Eclipse RAP](http://eclipse.org/rap)
* [Box.com Java SDK](http://opensource.box.com/box-java-sdk/)
* [jshint-eclipse](https://github.com/eclipsesource/jshint-eclipse)
* [tern.java](https://github.com/angelozerr/tern.java)

Include
-------

You can include minimal-json from Maven Central by adding this dependency to your `pom.xml`:

```xml
<dependency>
  <groupId>com.eclipsesource.minimal-json</groupId>
  <artifactId>minimal-json</artifactId>
  <version>0.9.2</version>
</dependency>
```

Build
-----

To build minimal-json on your machine, checkout the repository, `cd` into it, and call:
```
mvn clean install
```
A continuous integration build is running at [Travis-CI](https://travis-ci.org/ralfstx/minimal-json).

License
-------

The code is available under the terms of the [MIT License](http://opensource.org/licenses/MIT).
