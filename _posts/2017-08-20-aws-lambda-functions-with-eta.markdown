---
layout: post
title: AWS Lambda Functions with Eta
date: '2017-08-20 15:00:00'
comments: true
tags:
- programming
- haskell
- eta
- aws
---

Recently I learned about the programming language called [Eta](http://eta-lang.org) from [an episode of the Functional Geekery Podcast](https://www.functionalgeekery.com/episode-83-rahul-muttineni/).  Eta is a dialect of Haskell that is compatible with the GHC Haskell compiler and that runs on top of the JVM.  This could be a helpful step in Haskell gaining more adoption in production environments.  Because of the popularity of the Java programming language and the JVM, Haskell is able to be used in many more existing applications since it is possible to mix Eta code with the code of other languages built on top of the JVM.  Eta provides the ability to [write bindings for existing Java libraries](http://eta-lang.org/docs/html/eta-user-guide.html#java-foreign-function-interface) to enable this interaction.

### Serverless Architectures

More recently I've been focusing my time on understanding serverless and specifically AWS Lambda.  Because of my interest in Haskell I have been trying to find ways to use it for serverless applications.  While AWS does not support using Haskell directly for lambdas it is possible to [use a JavaScript shim](https://github.com/abailly/aws-lambda-haskell/blob/master/run-tmpl.js) with NodeJS to execute a Haskell binary.  While I suppose this works I would prefer to have better integration than that.  This is where Eta comes in.

With Eta I was able to write bindings to the lambda core library and build a jar file that would run as a Java8 function in AWS.  While [the example that I wrote](https://github.com/mspaulding06/eta-aws-lambda) doesn't do much it at least proved to me that it is possible to use Eta for serverless applications.

### Writing the Bindings

First I started with a very simple request handler function written in Java like this:

```java
package com.example;

import java.util.Collections;
import java.util.Map;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;

public class LambdaTest implements RequestHandler<Map<String, Object>, Map<String, Object>> {
    @Override
    public Map<String, Object> handleRequest(Map<String, Object> input, Context context) {
        return input;
    }
}
```

The above function just takes the input received in the `Map` and returns it back as the response from the lambda function invocation.  When the function is invoked a JSON object is passed to it which is unmarshaled into the `Map` and when it is returned as the response it is again marshaled into a JSON object.  I figured this would be a good first test to at least prove out that it works.

To write bindings I needed to define a module in Haskell code and write import statements for the objects that I was going to use which are the `Context` and `RequestHandler` objects exposed by the lambda core library.

```haskell
data {-# CLASS "com.amazonaws.services.lambda.runtime.Context" #-} Context =
    Context (Object# Context) deriving Class
```

Defining a data type for the Context was simple as it only requires providing the fully qualified name of the object you want to use in the Java library and then creating a type constructor that holds a context object.

```haskell
data {-# CLASS "com.example.LambdaTest implements com.amazonaws.services.lambda.runtime.RequestHandler" #-} RequestHandler a b =
    RequestHandler (Object# (RequestHandler a b)) deriving Class
```

In the case of the request handler, since I needed to implement an interface, I had to define the name of a class that I wanted to use in my program (`com.example.LambdaTest`) that would implement the request handler.  Since a `RequestHandler` object takes parameters for the input and output types the type constructor holds a `RequestHandler` that is polymorphic with two arguments.

### Implementing the Handler

Now that the bindings were written the handler itself needed to be written and then exported so that it could be called.  

```haskell
handler :: (Map JString JString) -> Context -> Java (RequestHandler (Map JString JString) (Map JString JString)) (Map JString JString)
handler m _ = return m
```

In the above code there are two arguments to the handler function, the input which is a Java `Map` and the `Context` object.  Since we are using the Java monad which is made available by importing the Java module provided by Eta the return value uses a `Java` type constructor.  The type constructor takes two arguments, the first argument is the type of the object on which the handler is being called, our `RequestHandler` object, and the second argument is the return value of the function which is another Java `Map`.

```haskell
foreign export java "handler"
    handler :: (Map JString JString) -> Context -> Java (RequestHandler (Map JString JString) (Map JString JString)) (Map JString JString)
```

Now that there is a handler function it must be exported to be called as a lambda function.  When exporting the function only the function name `handler` needs to be specified instead of the fully qualified name.  This is because the handler function has a return type that includes the object on which the function is called.

### Conclusion

I was really impressed how little code I had to write to implement a lambda function with Eta and I think that it has a lot of potential for providing the benefits of functional programming to enterprises and production systems that are using the JVM.  As a next step to make writing lambdas with Eta more useful it would be very helpful to be able to call the AWS SDK.  Some work has [already been done](https://github.com/Jyothsnasrinivas/eta-aws-core) for the core of the SDK.  Hopefully as the community around Eta grows and work continues on [FFI binding generators](https://github.com/typelead/eta-ffi/tree/develop), having these libraries available for use will be much easier.

### Get the Code

[Eta AWS Lambda Sample on Github](https://github.com/mspaulding06/eta-aws-lambda)