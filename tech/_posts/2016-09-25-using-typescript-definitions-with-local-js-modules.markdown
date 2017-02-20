---
layout:     post
title:      Using Typescript definitions with local Javascript modules
date:       2016-09-25 15:31:19
categories: tech
---

Let's dive into a common scenario of [creating Typescript definitions](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html) for local Javascript modules. We will create a simple Javascript module that contains a bunch of 'noises', and then try to use that code in a Typescript file.

I've found that the best way to get deep knowledge of a language or environment - aside from actually creating things with those tools - is to answer questions about those very things. I'm a firm believer that if you cannot teach something, then it probably means you don't know it well enough yet! You could say that I [learn by teaching](https://en.wikipedia.org/wiki/Learning_by_teaching).

Unfortunately, I'm a software engineer, not a teacher, so the oppurtunity for explanation is not as great. Luckily there is [Stack Overflow](https://stackoverflow.com/users/6137718/gnalck), where I've answered a couple of Typescript questions related to modules and definition files. I think these are as frequent as they are interesting, conceptually, because in the realm of inter-operability, you _have_ to know both sides of the equation very well, if you are to make sense of it all.

## Creating the Javascript module

If you've done anything in Javascript before, you are probably familiar with the module pattern, especially the [CommonJS](http://requirejs.org/docs/commonjs.html) variation. Essentially, in the file you want to create the module, you need to assign the various properties to the `module.exports` variable, and you can later get this in other files, whether by require.js, or by the Typescript `import` statement.

In this example, we want to create a module of various noises, so we simply assign a new object to `module.exports`, and have the various properties represent the noises we want to open for use:

```javascript
// noises.js
module.exports = {
  bark: 'ruff ruff',
  meow: 'meeeoooowwww'
  // etc...
}
```


## Creating the Typescript definition file

Let's create a definition file for our new module. This is done in Typescript by `export`-ing the variables with [type annotations](https://basarat.gitbooks.io/typescript/content/docs/types/type-system.html) we created in Javascript. Think of it of creating the 'signatures' of all exposed properties via type annotations, but leaving the implmentation to the actual javascript code.

In this case, we have our `module` that is called `noises` (its file is named that), and various string properties that are contained within it.

```typescript
// noises.d.ts
export module noises {
  export var bark: string;
  export var meow: string;
}
```

The above looks correct, but there is one fatal flaw - it doesn't match the [topography](https://en.wikipedia.org/wiki/Topography) of our original `noises.js` module! Here we are stating that the `noises.js` module exports an object called `noises`, and that object has two nested properties, `bark` and `meow`. In reality, our module exports `bark` and `meow` directly, without any `noises` variable containing them.

To reflect this, we can flatten our definition another level:

```typescript
// noises.d.ts
export var bark: string;
export var meow: string; 
```


## Using the Javascript module in Typescript

Now that we have our internal module created, let's use it from Typescript. In this case, we can use the import statement:

```typescript
// app.ts
import {bark, meow} from './noises';
console.log(bark);
```

Note that you can also get code completion here, if you type `import {} from './noises'` before filling in the objects you want to import within the brackets. Our new type definition file acts as a [contract](https://en.wikipedia.org/wiki/Design_by_contract) with our untyped javascript module, and the compiler uses it to verify our import statement is valid before generating the actual javascript that is run:

```javascript
// app.js (compiled output of app.ts)
"use strict";
var noises_1 = require('./noises');
console.log(noises_1.barrk);
```

This can be both a good and bad thing - if we have extra properties exposed in our `noises.js` file that is **not** in our `noises.d.ts` file, Typescript will fail to compile our `app.ts` if we try to import that property! On the other hand, if there is a typo in our `noises.d.ts` file, compile will be fine, but we will get a runtime error when we try to import it.

This may seem like a big deal, but in reality, we are still better off than we were with no typing at all. A javascript modules API should rarely change (within the standerd of [semver](http://semver.org/)), so you only have to get the definition right once, and be atleast backwards compatibile until the next major change (i.e., `1.0.1` updated to `2.0.0`).

## Further reading

This was merely an overview of the Typescript module world. If you want to learn more, I recommend the following documentations:

* [Namespaces and Modules - Typescript](https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html)
* [Writing Definition files](https://typescript.codeplex.com/wikipage?title=Writing%20Definition%20%28.d.ts%29%20Files)