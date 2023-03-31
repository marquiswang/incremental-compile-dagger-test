This is a small project that reproduces how Dagger breaks incremental compilation with Maven.

First of all, the correct behavior when you run `mvn compile`, is that Dagger generates a `Foo_Factory.java`,
and both compiles both `Foo.java` and `Foo_Factory.java`.

If you run:

```
$ mvn clean compile
$ find target -name "*.class"
target/classes/com/marquiswang/test/Foo_Factory$InstanceHolder.class
target/classes/com/marquiswang/test/Foo.class
target/classes/com/marquiswang/test/Foo_Factory.class
```

You see that it correctly generates the Factory and compiles both into class files.

Furthermore, if you run

```
$ mvn clean compile
$ mvn compile
```

In the second call to `mvn compile`, the `maven-compiler-plugin` is able to successfully detect that
nothing has changed, and so it emits:

```
[INFO] --- compiler:3.11.0:compile (default-compile) @ incremental-compile-dagger-test ---
[INFO] Nothing to compile - all classes are up to date
```

You can also see that the class files are still all there. 

However, if you modify (or just touch) `Foo.java`:

```
mvn clean compile
touch src/main/java/com/marquiswang/test/Foo.java
mvn compile
mvn compile
```

Then, for every subsequent call to `mvn compile`, it will always try to recompile everything.

```
[INFO] --- compiler:3.11.0:compile (default-compile) @ incremental-compile-dagger-test ---
[INFO] Changes detected - recompiling the module! :source
[INFO] Compiling 1 source file with javac [debug target 1.7] to target/classes
[WARNING] bootstrap class path not set in conjunction with -source 7
[WARNING] Implicitly compiled files were not subject to annotation processing.
  Use -proc:none to disable annotation processing or -implicit to specify a policy for implicit compilation.
``` 

If you add some debug flags,

```
mvn compile -Dmaven.compiler.showCompilationChanges=true -Dmaven.compiler.debug=true
```

Then you see that it says:

```
[INFO] Stale source detected: /path/to/incremental-compile-dagger-test/target/generated-sources/annotations/com/marquiswang/test/Foo_Factory.java
```

Finally you use find to look at the class files:

```
$ find target -name "*.class" 
target/classes/com/marquiswang/test/Foo.class
```

You see that the Dagger-generated classes have disappeared from the compiled class files.
