---
title: "Should you use TypeScript?"
date: 2022-11-22
authors: [ugur]
draft: false
---

Just as JavaScript, TypeScript gets more and more attention as each day passes.
Here is a google trends chart of the keyword TypeScript:

![trends](/img/typescript/trends.png)

We especially see a breakthrough in terms of interest after 2022. Does this
mean that you should get into the TypeScript train as well? Probably not. Let
me explain you my reasoning.

# So, should we use it or not?

TLDR: Depends on what you do, but most probably you don't need it!

Why?

When deciding whether to use a tool or not, it is important to ask whether or
not we can solve the problem at hand with a less complex, more direct approach
without using the tool that is suggested. If someone says that "you should use
X" to solve a problem, you should always ask: "What is the problem with the
current tools we have, and what does this tool you mention provides in addition
to our initial tool so that it justifies adding additional complexity/dependency
to our stack?" In essence, if someone is claiming that you should use typescript
because it is more productive compared to javascript, the the burden of proof
belongs to them.

I am not aware of any methodical research comparing typescript and javascript
when it comes to productivity (if you know one, please let me know). Which means
that the best we can do is to go over some of the most popular claims regarding
the matter. I will first examine some of the usual claims that are presented for
the sake of using typescript, and then provide some reasons about why one still
might not want to use it.

# Why use TypeScript?

## 1. Types reduce bugs

This is one of the most popular claims for why you should use TypeScript.

Types does not reduce all kinds of bugs. Types reduce type-bugs. Which is only a
very small portion of actual bugs. If using statically typed language would be
sufficient to reduce bugs, then we would see programs written in typed languages
having less bugs in general. But we don't. ([see](https://labs.ig.com/static-typing-promise))

In many cases bugs seems to occur not because of just being typless, but more
probably, a complex design combined with lack of a proper testing paradigm. If
you want to be sure that your program won't collapse during runtime, then it's
better for you to use a TDD (or a similar) approach combined with code reviews.
This will reduce other kinds of bugs alongside with type-bugs.

But still, if type constraints will make it less likely to bugs to occur, what's
the harm of using a typed language anyways? Well, I agree that if you are working
on a project and it has REALLY REALLY important parts, it might be a good idea
to restrain certain functions with working certain types of input and
output. However keep in mind that you can still implement certain type of type
constraining mechanisms on JavaScript depending on your needs.

Here is a very simple example of a way to create functions that checks whether
the given inputs satisfy certain properties by Eric Elliot.

```javascript
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
satisfy your needs. This might seem like an uggly solution at first but also
keep in mind that it might be better than putting yourself in a position where
you are forced to use types for the entire project when you only need it for a
small part of it.

If you are OK with writing types for all the variables and objects in the
project, then you can choose to go with typescript, but if only a small portion
of your codebase requires such a tidiness, it might be better for you to just
come up with a similar solution as mentioned above.

## 2. Types make your code more readable

I agree that sometimes types might make your code more readable in the sense
that it is more explicit and it helps you to understand what the inputs and
outputs for a function are supposed to be. Type constraints, in a way, also
serve the paradigm called 'Code as Documentation' which frees you from the
liability of writing documents.

So what's the problem? The problem is that most of the time, types also adds a
syntax noise to your code, that is, you are often FORCED to put redundant stuff
in the code.

Instead of just writing:

```javascript
let a = 5;
```

you are now suppoed to write:

```typescript
let a : number = 5;
```

the latter, in any was is not more readable compared to the first one. Because
the first one is already simple enough.

You might say "Well, types does not improve readability when your types are
simple, but when they are complex. In essence, types make your code more
readable in projects that are at larger scale." I agree with this, but I also
think that we should try to avoid such complexities as much as possible. That
is, we should strive to build things in a so simple and an explicit manner so
that a type is not a requirement for us anyways.

## 3. Big Companies are doing it

Here is a common way of thinking among some people: If TypeScript were bad, it
would not be used by big tech companies. big tech companies use TypeScript,
therefore TypeScript is not bad.

This argumentation is wrong for two reasons:

Firstly, you are not a big company, which means your requirements are probably not the
same as theirs.

Secondly, just Facebook is doing something does not mean it is good, this is a
fallacy in thinking known as 'appealing to authority'. There can be many cases
where big companies won't choose the better technology.

## 4. Dev Tools

The claim is as follows: Static Typing makes it possible to use great developer
tools that help us to become more productive such as jumping to definitions,
automatic refactoring, type inference and suggestions etc.

While it is true that Static Typing and Typescript has usually more dev tool
supports, there are two questions that are needed to be asked:

Are these tools really make you more productive, or make you feel like you
are more productive?

Are similar tools available for vanilla javascript as well? Is eslint +
autocomplete plugins + ES6 inference-capable tools not sufficient for your
workflow?

## 5. Types Provide a Good User Interface

I completely agree with this point.

The one thing I like about projects built using TypeScript is that they usually
have greate using experience. You can easily see the types of inputs and outputs
from documentation, or even look from the source code itself.

I believe that the effort and time spent building a typed interface justifies
itself since it pays off with the user experience (in this case, developers that
use the library).

So using Typescript is perfectly fine when building public libraries that are to be
used by many people which are not supposed to exactly understand what's going on
under the functions they use.

# Cons of TypeScript

## 1. Typescript is not a Script

Typescript is not actually a script. A script is a piece of code that is
executed directly by another program called interpreter rather than the computer
itself. In scripting languages, you do not need an additional step to compile
the program in order to run it. For nodejs, you just write 'node program.js'
where 'program.js' is the program you want to run. If you make a change on the
program, then you don't need to compile and run it again, you just run it.
However if you want to execute a typescript code, you first transpile it to
javascript, and then run it. This adds an additional step to your workflow,
which I personally don't like unless there is a HUGE benefit that justifies it.

## 2. Typed Javascript is still untyped

TypeScript code transpiles into javascript code nevertheless, which means,
although the source code you use have types, the actual code you run does not.
This is where typescript differs from truly statically typed languages like C
and C++ where different variable types are stored differently in memory.

Regardless of how carefull you design your types, there is always a chance that
a different value type might be assigned on a variable and can cause runtime
errors although typescript says its ok. Here is an interesting
[flood](https://twitter.com/devongovett/status/1157039444324577280?lang=en) by
the creator of parcel.js.

## 3. Slow Development

Will TypeScript make you more productive, or will it get in the way more than it
helps? When writing programs only in javascript it is sufficient for you to come
up with a solution and then build it. But in typescript, you also need to figure
out the types and type structures of the objects you are dealing with and tell
it to your code. If you want to follow "best practices" regarding TypeScript
this will mean that you will also spend good amount of time to how & where to
declare your types (this is generally true for most tools/paradigms).

## 4. Unnecessary Solutions

Types should not be a prerequisite for writing good code. You should strive to
build things in a so simple and an explicit manner so that a type is not a
requirement for them anyways.

# Conclusion

While using TypeScript might be preferable in certain cases like building public
libraries, it also comes with certain costs as well. Keep those costs in mind
whenever you are considering whether or not to use TypeScript.

# References

[You Might Not Need Typescript](https://medium.com/javascript-scene/you-might-not-need-typescript-or-static-types-aa7cb670a77b)  
[The Schocking Secret About Static Types](https://medium.com/javascript-scene/the-shocking-secret-about-static-types-514d39bf30a3)  
[Typescript is a waste of time, change my mind](https://dev.to/bettercodingacademy/typescript-is-a-waste-of-time-change-my-mind-pi8)  
[Typescript is not a script](https://www.youtube.com/watch?v=AkiLPpQ-6Wg)
