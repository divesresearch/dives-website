---
title: Should you use Typescript?
authors: ['ugur']
---

Just as JavaScript, TypeScript gets more and more common as each day passes.
Here is a google trends chart of the keyword TypeScript:

![trends](/img/typescript/trends.png)

We especially see a breakthrough in terms of interest after 2022. Does this
imply that you should get into the TypeScript train as well? Probably not. Let
me explain my you my reasoning.

# So, should you?

TLDR: Sometimes, but mostly not!

Why?

When deciding whether to use a certain technology or not, it is important to ask
whether or not we can solve the problem at hand with a less complex, more direct
approach without using the tool that is suggested. If someone says that "you
should use X" to solve a problem, you should always ask: "What is the problem
with the current tools we have, and what does this tool you mention provides in
addition to our initial tool so that it justifies adding aditional
complexity/dependency to our stack?" In essence, if someone is claiming that you
should use typescript because it is more productive compared to javascript, the
burden of proof belongs to the them.

As far as I know there is not a methodical research comparing typescript and
javascript in terms of productivity (if you know one, please let me know). Which
means that the best we can do is to gover over some of the most popular
arguments regarding the matter. I will first go over some of the
usual claims that are presented for the sake of using typescript, and then
provide some arguments on why costs of using typescript will surpass the
benefits of it.

# Why use TypeScript?

Now let's examine one of the most popular claims that are presented for the sake
of using TypeScript.

## 1. Types reduce bugs

This is one of the most popular claims for why you should use TypeScript.

Types does not reduce all kinds of bugs. Types reduce type-bugs. Which is only a
very small portion of actual bugs. If using statically typed language would be
sufficient to reduce bugs, then we would see programs written in typed languages
having less bugs in general. But we don't. ([see](https://labs.ig.com/static-typing-promise))

In many cases bugs seems to occur not because of just being typless, but more
probably, a complex design combined with a lack of a proper testing paradigm. If
you want to be sure that your program won't collapse during runtime, then it's
better for you to use a TDD (or a similar) approach combined with code reviews.
This will not only decrease other kinds of bugs in your code, but also the
type-bugs, which makes it less necessary to use statically typed languages.

But still, if type constraints will make it less likely to bugs to occur, what's
the harm of using a typed language anyways? Well, I agree that if you are working
on a project and it has REALLY REALLY important parts, it might be a good idea
to restrain certain functions with working certain types of input and
output. However keep in mind that you can still implement type contraining
mechanism on JavaScript depending on your needs.

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
you are forced to use types for the entire project when you need types for only
a small part of it.

If you are OK with writing types for all the variables and objects in the
project, than you can choose to go with typescript, but if only a small portion
of your codebase requires such a tidiness it might better for you to just come
up with a similar solution as mentioned above.

## 2. Types make your code more readable

I agree that sometimes types might make your code more readable in the sense
that it is more explicit and it helps you to understand what the inputs and
outputs for a function are supposed to be. Type constraints, in a way, serves
the mentality of 'Code as Documentation' too, which frees you from the liability
of writing documents.

However also keep in mind that most of the time, types also adds a syntax noise
to your code, that is, you are often FORCED to put redundant stuff in the code.

Instead of just writing:

```javascript
let a = 5;
```

you are now suppoed to write:

```typescript
let a : int = 5;
```

the latter, in any was is not more readable compared to the first one. Because
the first one is already simple enough.

You might say that "Well, types does not improve readability when your types are
simple, but when they are complex. In a way, type constraints make your code more
readable in projects that are at larger scale."

Then consider the following examples:

### Example 1.

This one from [Erik
Westra](https://medium.com/kaliberinteractive/were-not-writing-our-code-in-typescript-why-4a4babc1de2e),

```javascript
// JavaScript
function convert(a, { includedTypes }) {
  return a.reduce(
    (result, x) => includedTypes.includes(x.type) ? { ...result, [x.id]: x } : result,
    null
  )
}
```

```typescript
// TypeScript
type ConvertInput = { id: string, type: string }

function convert<T extends ConvertInput>( a: Array<T>, o: { includedTypes: Array<string> } ) {
  return a.reduce<null | { [id: string]: T }>(
    (result, x) => o.includedTypes.includes(x.type) ? { ...result, [x.id]: x } : result,
    null
  )
}

```

The convert function accepts a list objects with id and type properties than
returns a mapping from id to the object itself which contains the proper types
(that are, in the includedTypes array). Needless to say, the latter one in any
way is not better than the first one in terms of readibility because it has lots
of redundant stuff (noise) that is not actually relevant to the semantic of the
code.

### Example 2
This is an example is from [Better Coding
Academy](https://dev.to/bettercodingacademy/typescript-is-a-waste-of-time-change-my-mind-pi8)

```javascript
import React from 'react';
import ApolloClient from 'apollo-client';

let apolloContext;

export function getApolloContext() {
  if (!apolloContext) {
    apolloContext = React.createContext({});
  }
  return apolloContext;
}

export function resetApolloContext() {
  apolloContext = React.createContext({});
}
```

```typescript
import React from 'react';
import ApolloClient from 'apollo-client';

export interface ApolloContextValue {
  client?: ApolloClient<object>;
  renderPromises?: Record<any, any>;
}

let apolloContext: React.Context<ApolloContextValue>;

export function getApolloContext() {
  if (!apolloContext) {
    apolloContext = React.createContext<ApolloContextValue>({});
  }
  return apolloContext;
}

export function resetApolloContext() {
  apolloContext = React.createContext<ApolloContextValue>({});
}
```
### Example 3

example needed

## 3. Big Companies are doing it

Here is a common way of thinking among some people: If TypeScript were bad, it
would not be used by big tech companies. big tech companies use TypeScript,
therefore TypeScript is not bad.

This argumentation is wrong for two reasons:

You are not a big company, which means your requirements are probably not the
same as theirs.

Just Facebook is doing something does not mean it is good, this is a fallacy in
thinking known as 'appealing to authority'. There can be many cases where big
companies won't choose the better technology.

## 4. Dev Tools

The argument is as follows: Static Typing makes it possible to use great
developer tools such as jumping to definitions and automatic refactoring, hence
it helps us to be more productive.

While it is true that Static Typing and typescript has usually more dev tool
supports, there are two questions that are needed to be asked:

Are these tools really make you more productive, or make you feel like you
are more productive?

Are similar tools available for vanilla javascript as well? Is eslint +
autocomplete plugins + ES6 inference-capable tools not sufficient for your
workflow?


# Cons of TypeScript

Having mentioned the possible pros of using TypeScript, let's continue our
article by examining the possible cons of it.

## 1. Typescript is not a Script

Typescript is not actually a script. A script is a piece of code that is runned
directly by another program called interpreter rather than the computer itself.
In scripting languages, you do not need an additional step to compile the
program in order to run it. For nodejs, you just write 'node program.js' where
'program.js' is the program you want to run. If you make a change on the
program, then you don't need to compile and run it again, you just run it.
However if you want to execute a typescript code, you first transpile it to
javascript, and then run it. This adds an additional step to your workflow,
which I personally don't like, unless there is a HUGE benefit that justifies it.

## 2. Typed Javascript is still untyped

Typescript code transpiles into javascript code nevertheless, which means,
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
requirement for them anyways. In this sense, type is already an unnecessary
solution for writing understandable code.

# Conclusion

Although using static typing might be very handy in certain cases, it comes with
costs as well. And it's very probable that the PROs of using typescript
does not justify the price of it.

# References

[You Might Not Need Typescript](https://medium.com/javascript-scene/you-might-not-need-typescript-or-static-types-aa7cb670a77b)  
[The Schocking Secret About Static Types](https://medium.com/javascript-scene/the-shocking-secret-about-static-types-514d39bf30a3)  
[Typescript is a waste of time, change my mind](https://dev.to/bettercodingacademy/typescript-is-a-waste-of-time-change-my-mind-pi8)  
[Typescript is not a script](https://www.youtube.com/watch?v=AkiLPpQ-6Wg)
