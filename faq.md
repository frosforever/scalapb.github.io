---
title: "ScalaPB: Frequently Asked Questions"
layout: page
---

# Frequently Asked Questions

## IntelliJ complains on duplicate files ("class is already defined")

If you are using sbt-protoc this should not happen. Please file a bug.

If you are still using sbt-scalapb, please switch to sbt-protoc as described
in the installation instruction.

sbt-protobuf which sbt-scalapb relies on defaults to generating the case
classes in `target/src_managed/compiled_protobuf/`.  This leads to a situation
where both `target/src_managed/compiled_protobuf/` and its parent, `target/src_managed/`,
are considered source directories and the source files are seen twice. To
eliminate this problem, let's tell sbt-protobuf to generate the sources into
the parent directory. Add this to your `build.sbt`:

{%highlight scala%}
scalaSource in PB.protobufConfig := sourceManaged.value
{%endhighlight%}

If you generate Java sources, add,

{%highlight scala%}
javaSource in PB.protobufConfig := sourceManaged.value
{%endhighlight%}

## How do I use ScalaPB from the command line?

Check out [ScalaPBC]({{site.baseurl}}/scalapbc.html).

## How do I use ScalaPB with Maven?

ScalaPB can be invoked in your Maven build by calling ScalaPBC, a standalone
Java application that generates code. See an [example project](https://github.com/thesamet/scalapb-maven-example).

The relevant parts are marked with "Add the generated folder as a source" and
"Compile the proto file(s)".

## How do I get grpc, java conversions, flat packages, etc with Maven?

The example maven project invokes ScalaPBC. To get these ScalaPB features, you need to pass a
generator parameter to ScalaPBC. See the supported generator parameters and how to use them in 
[ScalaPBC]({{site.baseurl}}/scalapbc.html) documentation.

## I am getting "Import was not found or had errors"

If you are using sbt-protoc and importing protos like `scalapb/scalapb.proto`,
or common protocol buffers like `google/protobuf/wrappers.proto`:

Add the following to your `build.sbt`:

    libraryDependencies += "com.thesamet.scalapb" %% "scalapb-runtime" % scalapb.compiler.Version.scalapbVersion % "protobuf"

This tells `sbt-protoc` to extract protos from this jar (and all its
dependencies, which includes Google's common protos), and make them available
in the include path that is passed to protoc.

If you are not using sbt (for example, spbc), then you need to make those
files available on the file system.

## How do I generate Scala code for protos from another jar?

Include the jar as a `protobuf` dependency in your libraryDependencies:

    libraryDependencies += "com.somepackage" %% "that-has-jar" % "1.0" % "protobuf"

This will tell sbt-protoc to extract the protos from that jar into
`target/scala-2.vv/protobuf-external`. This makes it possible to `import`
those protos from your local protos. sbt-protoc looks for protocol buffers to
compile in the directories listed in `PB.protoSources`. There you need to
add a line like this to your `build.sbt`:

    PB.protoSources in Compile += target.value / "protobuf_external"

You may find other protos under `protobuf_external` that you do not wish to
compile. You can exclude them by adding an `includeFilter`:

    includeFilter in PB.generate := new SimpleFileFilter(
      (f: File) =>  f.getParent.endsWith("com/thesamet/protos"))

See [full example here](https://github.com/thesamet/sbt-protoc/tree/master/examples/multi-with-external-jar).

## How do I represent `Option[T]` in proto3 for scalar fields?

Scalar fields are the various numeric types, `bool`, `string`, `byte` and `enum` -
everything except of messages. In the proto2 wire format, there is a distinction between
not setting a value (`None`), or setting it to its default value (`Some(0)` or
`Some("")`).

In proto3, this distinction has been removed in the wire format. Whenever the value
of a scalar type is zero, it does not get serialized. Therefore, the parser is not
able to distinguish between `Some(0)` or `None`. The semantics is that a [zero
has been received](https://developers.google.com/protocol-buffers/docs/proto3#default).

Optional message types are still wrapped in `Option[]` since there is a
distinction in the wire format between leaving out a message (which is
represent by `None` or sending one, even if all its fields are unset (or
assigned default values). If you wish to have `Option[]` around scalar
types in proto3, you can use this fact to your advantage by using [primitive wrappers]({{site.baseurl}}/customizations.html#primitive-wrappers)
