# litjava

litjava is a literate programming tool in Java for Java. It is designed as follows:

1. the literate format is a Markdown file, so that it's rendered directly in Github.
1. it supports test-driven development, where the tests themselves are also literate programs.
1. it is usable in a continous integration setting, in particular in Travis.

## Contributing

litjava is open-source and welcomes contributions as pull-requests. Questions and bug reports are welcome on the [litjava Github issue tracker](https://github.com/monperrus/litjava/issues).

## Writing a program in litjava

Simply add Java code in a README.md file, as you would do in any documentation file on Github, for instance.

### Getting started

```java
int i = 3+4;
```
The code has to be enclosed by <code>\`\`\`java</code> and <code>\`\`\`</code> to be recognised as litjava code.

The litjava code does not have to be a complete java method or a complete java class. litjava always creates the necessary wrapper code if required or reports an error it if fails to understand what you mean.

### Writing tests

Now let's write tests:
```java
void test() {
  LitJava litjava = new LitJava('README.md');
  Section s0 = litjava.getSection(0);
  assertEquals("int i = 3+4;", s0.getText());
}
```
Any method that starts with `test*` is considered as a test method. You don't have to specify a test class, or to add the `@Test` annotation. By default, all test methods are put in a class called `FooTest` where `Foo` is the name of the literate program file. Since you're reading `README.md`, this test will be put in `READMETest.java`.

If the same literate program contains several test methods with the same name, it adds a unique suffix to the method name.

You can also directly write an assertion. If a snippet contains a call to a Junit assertion method, it's considered as a test and wrapped in a test method. 
```java
assertEquals(7, 3+4);
```

### Writing functions

To write a function, put it directly in a code section.
```java
void add(int x, int y) {
  return x+y;
}
```
litjava automicaly considers it as static (even if you don't add write it as such), and adds it to a class called `Functions.java`. 

And we can test it:
```java
assertEquals(7, add(3,4));
```

### Writing classes

One can write a class directly.
```java
class Point {
  int x;
  int y;
}
```

However, sometimes we want to explain each method in a literate way, that is we want to split the class in several snippets. To do this; simply starts the class with a special tag:

```java
@Literate
class Car {
  int speed = 0;
}
```
(you still have to close with a bracket, because any Litjava section has to be parsable.)

Then, one explains the methods, eg a car can accelerate


```java
@Literate(class="Car")
void accelerate() {
  this.speed += 2;
}
```

The annotation `@Literate` tells that this method is part of class `Car`.

And a car can stop
```java
@Literate(class="Car")
void stop() {
  this.speed = 0;
}
```

And we write the corresponding tests:
```java
Car c = new Car();
assertEquals(0, c.speed);
c.accelerate();
assertEquals(2, c.speed);
c.stop();
assertEquals(0, c.speed);
```

Other contracts:

* Contrary to tests, two classes cannot have the same name.
* if a method or a field is package-protected it's made public in the generated source code. we do this to encourage readable code.

### Imports

You never write an import in litjava, litjava automically finds the corresponding class in the classpath.
If two or more classes with the same name exists, litjava tries to import the right one based on the usage in the code, and reports an error if it fails to detect the correct class.

### Using libraries

litjava uses Markdown metadata to specify the libraries used by your code (that are also [shown by default in Github](https://github.com/blog/1647-viewing-yaml-metadata-in-your-documents)). The metadata is in YAML.

```yaml
---
dependency: ./lib/foo.jar
---
```
A dependency can have two formats, a local path to a jar file, as in the example above, or a reference to a Maven central artifact, following the Gradle convention `<group>:<artifact>:<version`, such as:

```yaml
---
dependency: fr.inria.gforge.spoon:spoon-core:5.4.0
---
```

### Using packages

By default, all classes are put in the package named from the file name of the Markdown literate program. However, you can specify an alternate top-level package, in the program metadata:

```yaml
---
package: litjava
---
```

If you want to put a specific class in given package, the `@Literate` annotation takes an optional parameter `package`.

```java
@Literate(package="foo.bar")
class Bird {}
```

## Usage

### Command-line usage
To download litjava:
```sh
$ wget https://raw.githubusercontent.com/monperrus/litjava/master/download-litjava.sh | bash
```
To create the java files from the literate program:
```sh
java -cp litjava.jar litjava.LitJava README.md
```

This creates all application classes and test classes files in folder `src`. In addition, it generates a default gradle file to build and test the application.
```sh
java -cp litjava.jar litjava.LitJava README.md
cd litjava
gradle test
```

### Integration into Travis

Activate Travis for your repository, and adds a file `.travis.yml` which contains the following content:

```yaml
language:java
script: wget https://raw.githubusercontent.com/monperrus/litjava/master/litjava-travis.sh | bash
```


## Related work

[Corrode](https://github.com/jameysharp/corrode/blob/master/src/Language/Rust/Corrode/C.md) is literate haskell rendered in Github. It's the principal source of inspiration of litjava.

[Rambutan](https://www.tug.org/TUGboat/tb23-3-4/tb75saha.pdf) is a literate programming tool for java, which  addresses Java specificities such as as classes and imports and produces nice documents.

## Tests

This is the test suite of litjava itself.

```java
LitJava litjava = new LitJava('README.md');
assertEquals("litjava", litjava.getPackageName());
assertEquals("foo.bar", litjava.getClass("Bird").getPackage().getQualifiedName());
```

