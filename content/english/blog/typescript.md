---
title: "Should you use TypeScript?"
date: 2022-11-22
authors: [ugur]
draft: false
---

TypeScript is a programming language that is a superset of JavaScript, meaning
it supports all of the features of JavaScript but also includes additional
features.

Just like JavaScript, TypeScript gets more and more attention as each day passes.
Here is a google trends chart of the keyword TypeScript:

![trends](/img/TypeScript/trends.png)

You can see an important increase in interest in TypeScript, particularly after
2022. So should you get into the TypeScript train as well?

In this essay, I will examine the potential benefits and drawbacks of using
TypeScript in different contexts. The decision of whether or not to use
TypeScript will depend on the specific needs of the project at hand. I will
argue that, in most cases, those specific needs of a project will not justify
the use of TypeScript, unless it is being used for a specific purpose, such as
building a public library.

I will first examine and conclude whether some of the usual claims are presented
for the sake of using TypeScript and then provide some reasons why one still
might not want to use it.

# Why use TypeScript?

## 1. Types reduce bugs

This is one of the most popular claims for why you should use TypeScript.

Types do not reduce all kinds of bugs. Types reduce type-bugs. Which is only a
very small portion of actual bugs. If using statically typed language would be
sufficient to reduce bugs, then we would see programs written in typed languages
having fewer bugs in general. But we don’t. ([see](https://labs.ig.com/static-typing-promise))

In many cases bugs seems to occur not because of just being typless, but more
probably, a complex design combined with lack of a proper testing paradigm. If
you want to be sure that your program won't collapse during runtime, then it's
better for you to use a TDD (or a similar) approach combined with code reviews.
This will reduce other kinds of bugs alongside with type-bugs.

But still, if type constraints will make it less likely for bugs to occur,
what’s the harm of using a typed language anyways? Well, I agree that if you are
working on a project and it has important parts, it might be a good idea to
restrain certain functions by working on certain types of input and output.
However, keep in mind that you can still implement a certain type of type
constraining mechanism on JavaScript depending on your needs.

Here is a very simple example of a way to create functions that check whether
the given inputs satisfy certain properties by Eric Elliot.

```JavaScript
const fn = (fn, {required = []}) => (params = {}) => {
  const missing = required.filter(param => !(param in params));

  if (missing.length) {
    throw new Error(`${ fn.name }() Missing required parameter(s):
    ${ missing.join(', ') }`);
  }

  return fn(params);
};

const createEmployee = fn(({
    name = '',
    hireDate = Date.now(),
    title = 'Worker Drone'
  } = {}) => ({
    name, hireDate, title
  }),
  { required: ['name'] }
);

console.log(createEmployee({ name: 'foo' })); // works
createEmployee(); // createEmployee() Missing required parameter(s): name
```

Principally you can extend `fn` or come up with similar solutions in any way to
satisfy your needs. This might seem like an ugly solution at first but also
keep in mind that it might be better than putting yourself in a position where
you are forced to use types for the entire project when you only need them for a
small part of it.

If you are OK with writing types for all the variables and objects in the
project, then you can choose to go with TypeScript, but if only a small portion
of your codebase requires such a tidiness, it might be better for you to just
come up with a similar solution as mentioned above.

## 2. Types make your code more readable

I agree that sometimes types might make your code more readable in the sense
that it is more explicit and it helps you to understand what the inputs and
outputs for a function are supposed to be. Type constraints, in a way, also
serve the paradigm called 'Code as Documentation' which frees you from the
liability of writing documents.

So what's the problem? The problem is that most of the time, types also add a
syntax noise to your code, that is, you are often FORCED to put redundant stuff
in the code.

Instead of just writing:

```JavaScript
let a = 5;
```

you are now supposed to write:

```TypeScript
let a : number = 5;
```

the latter, in any way, is not more readable compared to the first one. Because
the first one is already simple enough.

You might say “Well, types do not improve readability when your types are
simple, but when they are complex. In essence, types make your code more
readable in projects that are at a larger scale.” I agree with this, but I also
think that we should try to avoid such complexities as much as possible. That
is, we should strive to build things in a so simple and explicit manner so that
a type is not a requirement for us anyways.

## 3. Big Companies are doing it

Here is a common way of thinking among some people: If TypeScript were bad, it
would not be used by big tech companies. big tech companies use TypeScript,
therefore TypeScript is not bad.

This argument is wrong for two reasons:

Firstly, you are not a big company, which means your requirements are probably not the
same as theirs.

Secondly, just Facebook is doing something does not mean it is good, this is a
fallacy in thinking known as 'appealing to authority'. There can be many cases
where big companies won't choose the better technology.

## 4. Dev Tools

The claim is as follows: Static Typing makes it possible to use great developer
tools that help us to become more productive such as jumping to definitions,
automatic refactoring, type inference, and suggestions, etc.

While it is true that Static Typing and Typescript has usually more dev tool
supports, two questions are needed to be asked:

Are these tools make you more productive, or make you feel like you are more
productive?

Are similar tools available for vanilla JavaScript as well? Is eslint +
autocomplete plugins + ES6 inference-capable tools not sufficient for your
workflow?

## 5. Types Provide a Good User Interface

I completely agree with the point that projects built using TypeScript often
have a great user experience. This is because the effort and time spent building
a typed interface pay off in the form of improved documentation and ease of
understanding for developers who use the library. The use of TypeScript is
especially beneficial when building public libraries that will be used by many
people who may not have a deep understanding of the inner workings of the
functions they are using. Overall, the use of TypeScript can greatly improve the
user experience and is therefore a worthwhile investment when building libraries
that will be widely used.

# Cons of TypeScript

## 1. Typescript is not a Script

Typescript is not a script. A script is a piece of code that is executed
directly by another program called an interpreter rather than the computer
itself. In scripting languages, you do not need an additional step to compile
the program to run it. For nodejs, you just write ‘node program.js’ where
‘program.js’ is the program you want to run. If you make a change to the
program, then you don’t need to compile and run it again, you just run it.
However, if you want to execute a TypeScript code, you first transpile it to
JavaScript and then run it. This adds an additional step to your workflow, which
I don’t like unless there is a HUGE benefit that justifies it.

## 2. Typed Javascript is still untyped

TypeScript code transpiles into JavaScript code nevertheless, which means,
although the source code you use has types, the actual code you run does not.
This is where TypeScript differs from truly statically typed languages like C
and C++ where different variable types are stored differently in memory.

Regardless of how careful you design your types, there is always a chance that
a different value type might be assigned on a variable and can cause runtime
errors although TypeScript says it's okay. Here is an interesting
[flood](https://twitter.com/devongovett/status/1157039444324577280?lang=en) by
the creator of parcel.js.

## 3. Slow Development

Will TypeScript make you more productive, or will it get in the way more than it
helps? When writing programs only in JavaScript it is sufficient for you to come
up with a solution and then build it. But in TypeScript, you also need to figure
out the types and type structures of the objects you are dealing with and tell
it to your code. If you want to follow "best practices" regarding TypeScript
this will mean that you will also spend a good amount of time on how & where to
declare your types (this is generally true for most tools/paradigms).

## 4. Unnecessary Solutions

Types should not be a prerequisite for writing good code. You should strive to
build things in a so simple and explicit manner so that a type is not a
requirement for them anyways.

# Conclusion

When deciding whether to use a tool, it's important to consider whether a
simpler, more direct approach can solve the problem without it. If someone
suggests using tool X to solve a problem, you should ask what the current tools
can't do and what tool X offers to justify adding complexity and dependencies.
If someone claims that TypeScript is more productive than JavaScript, the burden
of proof is on them.

Without methodical research comparing TypeScript and JavaScript in terms of
productivity (if you are aware of any relevant research, please let me know), we
must rely on our reasoning to decide the best approach.

Based on the arguments presented, it appears to me that using TypeScript is not
necessary in many cases. While it may have benefits in specific situations like
building public libraries, it also comes with costs to consider. It's important
to weigh these costs against the specific needs of the project before deciding
to use TypeScript. In essence, using TypeScript can be justified based on what
you do, but most probably you don't need it!

# References

[You Might Not Need Typescript](https://medium.com/JavaScript-scene/you-might-not-need-TypeScript-or-static-types-aa7cb670a77b)  
[The Schocking Secret About Static Types](https://medium.com/JavaScript-scene/the-shocking-secret-about-static-types-514d39bf30a3)  
[Typescript is a waste of time, change my mind](https://dev.to/bettercodingacademy/TypeScript-is-a-waste-of-time-change-my-mind-pi8)  
[Typescript is not a script](https://www.youtube.com/watch?v=AkiLPpQ-6Wg)
