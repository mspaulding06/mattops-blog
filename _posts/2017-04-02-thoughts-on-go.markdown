---
layout: post
title: Thoughts on Go
date: '2017-04-02 03:23:21'
comments: true
tags:
- programming
- go
---

About six month ago I decided it was time to learn a new language and picked up [The Go Programming Language](https://www.amazon.com/Programming-Language-Addison-Wesley-Professional-Computing/dp/0134190440/ref=sr_1_1?ie=UTF8&qid=1491096913&sr=8-1&keywords=programming+go).  Go is a language I've been thinking about learning for some time but didn't have a good reason.  I ended up writing a configuration management tool, something like Terraform.  Typically, when learning a new programming language I prefer to read documentation and a good intermediate to advanced level book on the subject.  Introductory books tend to be too slow and lose my interest as they describe the syntax of variables, conditionals, and data types, which are fairly similar among imperative languages.  Kernighan's book opens strong by going through a series of typical modern programming tasks which includes building a web server.  If you're interested in learning Go this is an obligatory companion to the [Go Documentation](https://golang.org/pkg/).  Those are the only two things you need.

#### First Impressions

One thing of note is that Go does not have classes, at least not in the traditional sense of a language like C++.  In Go you must associate functions with a structure.  The declaration of the function includes the structure's data type so that these functions are automatically available from an instance.  There is no requirement to _bless_ the structure as in Perl to bind the class to the data structure -- it is all automatic.

```go
package main

import "fmt"

type Greeter struct {
  name string
}

func NewGreeter() *Greeter {
  return &Greeter{}
}

func (g *Greeter) SetName(name string) {
  g.name = name
}

func (g Greeter) Greet() error {
  if g.name == "" {
    return fmt.Errorf("No Name Set!")
  }
  fmt.Printf("Hello, %s\n", g.name)
  return nil
}

func main() {
  g = NewGreeter()
  g.SetName("Grace")
  if err := g.Greet(); err != nil {
    panic(err)
  }
}
```

In the above example (as in Go as a whole) there is no exception handling.  Go does not throw exceptions, it only returns error objects.  If you are interested in dealing with errors then you must take the error object and in the case where it is not `nil` you can choose how to handle it.  While at first I thought this was a good idea I'm now not so sure.  Oftentimes programs will crash and it turns out that you failed to handle an error case somewhere deep inside the code.  If you had exceptions you would get an automatic stack trace on failure that pinpoints where the problem occurred.  In Go you must sift through your code until you figured out where you failed to check the returned error.

You might also notice that there isn't a constructor or initialization function for the structure that is created.  You must create your own constructor.  You'll want to give it a descriptive name since it's likely you'll have more than one struct in your package, since the constructor isn't called using the name of the struct but the package from which the constructor is called.  You also might notice that the constructor function returns a pointer to the structure.  Yes, Go has the concept of pointers, but there is no pointer arithmetic and there is no need to dereference a pointer to call functions on the object.  Pointers are also used when declaring functions for a structure. In the example above the `Greet()` function is called on a `Greet` object while `SetName()` is called on a `*Greet`.  That is because in the case of `SetName()` we intend to modify the object and so it must be mutable.  If a pointer is used instead then changes to the object will persist.  If instead like with the `Greet()` method we attempt to modify the object those changes would not persist because a copy of the object is used in the function and so it is immutable.

#### Convention

For a systems programming lanaguage a Go project is easier to manage than something like C or C++.  Each code file includes a package declaration at the top of the file which defines the namespace for the file.  Go also enforces that all code files in a directory must have the same package declaration.  This allows for writing maintainable code with minimal effort.  Organization of files makes sense as do most things with the language.  Go is opinionated and enforces language conventions.  Where some languages allow for doing whatever the programmer desires -- such as with Perl -- with Go the programmer must do things the Go way.  For instance, there is no need for a Go style guide because there is one enforced style for all Go code and the compiler will reformat your code for you if you run `go fmt mycode.go`.  You are also required to use all imported packages in your code or it is considered a compilation error.

#### To Be Continued

Hopefully this has given you an idea of what Go is all about.  I'll continue this in a future post and cover topics such as dependency management, goroutines, and other useful language features.