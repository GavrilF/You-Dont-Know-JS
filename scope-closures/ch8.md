# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# Chapter 8: The Module Pattern

| NOTE: |
| :--- |
| Work in progress |

We've examined every angle of lexical scope, from the breadth of the global scope down through nested block scopes, into the intricasies of the variable lifecycle. Then we leveraged lexical scope to understand the power of closure. Throughout all the discussion so far, the underlying thread has been that understanding and mastering scope and closure is central to properly structuring and organizing our code, especially where we store information in variables.

Take a moment to reflect on how far you've come in this journey so far; you've taken big steps in getting to know JS more deeply!

In this chapter, we wrap up the main text of the book by exploring one of the most important code organization patterns in all of programming: the module. As we'll see, modules are inherently built from what we've already covered. Modules are the payoff for your efforts in learning lexical scope and closure.

To whatever extent these topics may still feel a bit abstract or mostly academic, our goal here is to appreciate where modules elevate our thinking to concrete, practical impact on how we write programs.

## Encapsulation and POLE

Encapsulation is often cited as a principle of object-oriented (OO) programming, but it's more fundamental and broadly applicable than that. The goal of encapsulation is the bundling or co-location of information (data) and behavior (functions) that serve a common purpose.

Independent of any syntax or code mechanisms, the spirit of encapsulation can be realized in something as simple as organizing bits of the program with common purpose into separate files. If we bundle everything that powers a list of search results into a single file called "search-list.js", we're encapsulating that part of the program.

A modern trend in front-end programming to organize applications around Component architecture pushes encapsulation even further. It feels natural to consolidate everything that constitutes the search results list -- even beyond code, to include presentational markup and styling -- into a single unit of program logic, something tangible we can interact with. So we label that collection the "SearchList" component.

Another key goal is the control of visibility of certain aspects of the encapsulated data and functionality. Recall the POLE principle from Chapter 6, which seeks to defensively guard against various *dangers* of over-exposure; these apply not only to variables but also functions. In JS, we most often accomplish visibility control through the mechanics of lexical scope.

The idea is to group alike program bits together, and selectively limit programmatic access to certain parts which can reasonably be described as *private* details. What's not considered *private* is then by default *public*, accessible to the whole program.

The natural effect of this effort is better code organization. It's easier to build and maintain software when we know where things are, with clear and obvious boundaries and connection points. It's also easier to maintain quality if we avoid the pitfalls of over-exposed data and functionality.

These are some of the main benefits of organizing JS programs into modules.

## What is a Module?

A module is a collection of related data and functions (often referred to as methods in this context), characterized by a division between hidden *private* details and *public*, accessible details, usually called the "public API".

Modules should be stateful -- they need to maintain some information -- otherwise, it's not really a module. If all you have is a collection of related functions, but no data, then you don't have encapsulation and thus you don't have a module.

The better term for a collection of *stateless* functions is a namespace:

```js
// namespace, not module
var Utils = {
    cancelEvt(evt) {
        evt.preventDefault();
        evt.stopPropagation();
        evt.stopImmediatePropagation();
    },
    wait(ms) {
        return new Promise(function c(res){
            setTimeout(res,ms);
        });
    },
    isValidEmail(email) {
        return /[^@]+@[^@.]+\.[^@.]+/.test(email);
    }
};
```

`Utils` here is a useful collection of utilities, yet they're all state-independent functions. Gathering functionality together is generally good pratice, but that doesn't make this a module. Rather, we've defined a `Utils` namespace and organized the functions under it.

Further, even if you have data and stateful functions together, if you're not limiting the visibility of any of it, then it's falling short of the POLE side of encapsulation and is probably not useful to label as a module:

```js
var Student = {
    records: [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ],
    getName(studentID) {
        var student = this.records.find(
            student => student.id == studentID
        );
        return student.name;
    }
};

Student.getName(73);
// Suzy
```

Since `records` is publicly accessible data, not hidden behind a public API, `Student` here isn't truly a module. What is it? `Student` does have the data-plus-functionality aspect of encapsulation, but not the visibility-control aspect, so it's an instance of a general data structure.

| NOTE: |
| :--- |
| Another aspect of the spirit of the module pattern is embracing modularization through loose-coupling. In this chapter, we're focusing more narrowly on implementing modules, so we'll defer the bigger-picture discussion of how modules can be designed to interact well with each other to Appendix A. |

### Classic Modules

Let's turn `Student` into a module. We'll start with what's called the "classic module pattern", which also used to be referred to as the "revealing module pattern" when it first emerged in the early 2000's.

```js
var Student = (function defineStudent(){
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
})();

Student.getName(73);
// Suzy
```

`Student` is now an instance of a module. It features a public API with a single method: `getName(..)`. This method is able to access the private hidden `records` data.

| WARNING: |
| :--- |
| I should point out that the explicit student data being hard-coded into this module definition is just for our illustration purposes. A typical module in your program will receive this data from an outside source, typically loaded from databases, JSON data files, Ajax calls, etc. The data is then injected into the module instance typically through some method(s) of the module's public API. |

How does it work?

Notice that the instance of the module is created by the `defineStudent()` IIFE being executed. This IIFE returns the object (named `publicAPI`) which has a property on it referencing the inner `getName(..)` function.

The naming of the object as `publicAPI` is purely stylistic preference on my part. The object can be named whatever you like (JS doesn't care), or you can just return an object directly without assigning it to any internal named variable. I like to assign it to `publicAPI` because it nicely labels what the object is, and also is a handy reference to the module's public API from inside itself, if needed.

From the outside, `Student.getName(..)` invokes this exposed inner function, which maintains access to the inner `records` variable via closure.

By the way, you don't *have* to return an object with a function as one of its properties. You could just return a function directly, in place of the object. That still satisfies all the core bits of a classic module.

The use of an IIFE implies that our module only ever needs a single central instance, which is commonly referred to as a "singleton". Indeed, this specific example is simple enough that there's no obvious reason we'd need anything more than just one instance of the `Student` module.

But if we did want to define a module that supported multiple instances in our program, we can slightly tweak the code:

```js
function defineStudent() {
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
}

var fullTime = defineStudent();
fullTime.getName(73);
// Suzy
```

Rather than specifying `defineStudent()` as an IIFE, we just define it as a normal standalone function, which is commonly referred to in this pattern as a "module factory" function.

We then call the module factory, producing an instance of the module that we name `fullTime`. `fullTime.getName(..)` can now invoke the method on that specific instance.

So to clarify what makes something a classic module:

* there must be an outer scope, typically from a module factory function running at least once.

* the module's inner scope must have at least one piece of hidden information that represents state for the module.

* the module must return on its public API a reference to at least one function that has closure over the hidden module state (so that this state is actually preserved).

### Module Variations

// TODO

## Modern ES Modules

// TODO
