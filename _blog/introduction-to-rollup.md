---
title: Introduction to Rollup
date: 2018-11-01 08:52:00
tags: [javascript, vue]
excerpt: Let's build a reusable and beautiful chart component for Vue.js using the Chart.js library.
image_thumb: ../img/blog/vue-chart-component-with-chartjs/featured-thumb.png
image:
  path: ../img/blog/vue-chart-component-with-chartjs/og.png
  width: 1200
  height: 720
---
Similar to Webpack or Browserify, Rollup is a module bundler for JavaScript. It allows us to use the modern ES module system and transform it into another module system: CommonJS, AMD, or the UMD. It can also bundle our module and wrap it inside the IIFE (Immediately-Invoked Function Expression).

Though people usually use Rollup to bundle a library. It's possible to bundle an application too.

## Table of Contents
{:.no_toc}

* TOC
{:toc}

## Installation

To install Rollup globally, run the following command on your terminal:

```bash
$ npm install -g rollup

# Or if you prefer to use Yarn
$ yarn global add rollup
```

You can now run the following command to print the Rollup CLI help:

```bash
$ rollup -h
```

## Quick Start

To see Rollup in action, let's create a simple ES module:

```js
// main.js
const eat = food => console.log(`I eat ${food}.`);

export default eat;
```

It just a simple function that will print `I eat something` on the console or the terminal. Save the file as `main.js`. On the terminal type the following command to transform our ES module into a CommonJS module:

```bash
$ rollup main.js --file bundle.js --format cjs
```

* `--file`: The path for the output file
* `--format`: The targetted output format, in our case the `cjs` is for the CommonJS module.

Along with the `main.js` file, we should now have the `bundle.js` file. If we open the `bundle.js` file, we'll see that our `eat` function is now using the CommonJS module.

```js
// bundle.js
'use strict';

const eat = food => console.log(`I eat ${food}.`);

module.exports = eat;
```

If you're familiar with Node.js, that's how you usually export a module.

## Exploring Other Output Formats

Rollup offers five output formats for your bundle:

* `cjs`: The CommonJS module that typically targetted for the Node.js environment.
* `amd`: The AMD module which usually used in the browser.
* `umd`: The UMD module which often use to target both the Node.js and the browser environments.
* `es`: The ES module itself.
* `iife`: Which will wrap our bundle within the IIFE (Immediately-Invoked Function Expression) for browser usage.

### AMD Output

Let's bundle our `main.js` module into an AMD module:

```bash
$ rollup main.js --file amd.js --format amd
```

Here's the generated `amd.js` file looks like:

```js
// amd.js
define(function () { 'use strict';

    const eat = food => console.log(`I eat ${food}.`);

    return eat;

});
```

If you ever worked with [RequireJS](https://requirejs.org) or [curl.js](https://github.com/cujojs/curl) in the past. You might notice the `define` function above. It's how you register a factory function in the AMD format.

### UMD Output

Now, let's create the UMD version:

```bash
$ rollup main.js --file umd.js --format umd --name eat
```

For UMD format, we also need to pass the `--name` option. It's the identifier that will be used to expose our module. We use the same name as our `eat` function. We can use a different name though.

If we open up the `umd.js` file, the generated code will look like this:

```js
(function (global, factory) {
    typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
    typeof define === 'function' && define.amd ? define(factory) :
    (global.eat = factory());
}(this, (function () { 'use strict';

    const eat = food => console.log(`I eat ${food}.`);

    return eat;

})));
```

It slightly looks more complicated than the previous output formats. It's an IIFE that will check if the current environment supports a CommonJS or an AMD module format. If the current environment supports a CommonJS module, it will export our `eat` function.

```js
typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory()
```

Or if the current environment supports an AMD module, it will register a factory function for our `eat` module.

```js
typeof define === 'function' && define.amd ? define(factory)
```

Otherwise, it will add the `eat` function into the current `this` object:

```js
global.eat = factory()
```

For example, if we load our `umd.js` file in the browser, we can access our `eat` function from the `window` object:

```js
window.eat('🍕');
```

### ES Output

Rollup can also output the same exact ES module:

```js
rollup main.js --file es.js --format es
```

If we check the generated `es.js` file, we'll find the identical code:

```js
const eat = food => console.log(`I eat ${food}.`);

export default eat;
```

We use this option in case the users of our library want to use a tool that can leverage the ES module, like Webpack  2+.

### IIFE

The last format that we can generate with Rollup is IIFE (Immediately-Invoked Function Expression):

```js
rollup main.js --file iife.js --format iife --name eat
```

Just like the UMD format, we have to set the `--name` option. If we check the `iife.js` file, the `eat` function is wrapped within the IIFE block:

```js
var eat = (function () {
    'use strict';

    const eat = food => console.log(`I eat ${food}.`);

    return eat;

}());
```

## Rollup Configuration File

We can also use a configuration file to configure Rollup. Before continuing, let's delete the generated files. Then move the `main.js` file into the `src` directory.

```bash
# Delete generated files
$ rm bundle.js amd.js umd.js es.js iife.js

# Move main.js to src directory
$ mkdir src
$ mv main.js src
```

Create a new file named `rollup.config.js` and put the following configuration object:

```js
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'dist/cjs.js',
    format: 'cjs'
  }
};
```

Your current directory should now look like this:

```txt
|-- rollup.config.js
|-- src
|   |-- main.js
```

Run the following command to bundle our main.js file:

```bash
$ rollup -c
```

* `-c`: The config file path. If not set, it will look for the `rollup.config.js` file.

We should now have the generated CommonJS file stored at: `dist/cjs.js`.

### Multiple Output Formats

We can also assign an array to the `output` option to generate various output formats:

```js
// rollup.config.js
export default {
  input: 'src/main.js',
  output: [
    {
      file: 'dist/cjs.js',
      format: 'cjs'
    },
    {
      file: 'dist/umd.js',
      format: 'umd',
      name: 'eat'
    }
  ]
};
```

If you run the `rollup -c` command again, it will generate two bundles with different module format: `cjs` and `umd`.

### Multiple Inputs

It's also possible to build bundles from multiple inputs:

```js
// rollup.config.js
export default [
  {
    input: 'src/main.js',
    output: [
      {
        file: 'dist/cjs.js',
        format: 'cjs'
      },
      {
        file: 'dist/umd.js',
        format: 'umd',
        name: 'eat'
      }
    ]
  },
  {
    input: 'src/other.js',
    output: {
      file: 'dist/other.bundle.js',
      format: 'cjs'
    }
  }
];
```

## Tree-Shaking

One of the most amazing features on Rollup is its tree-shaking ability. RollUp can statically analyze our code and remove unused functions or modules from our bundle.

### Preparing Our Food Factory

Within the `src` directory create a new file named `food.js`:

```js
// src/food.js
const FRUITS = ['🍏', '🍉', '🍇'];

const FAST_FOODS = ['🍔', '🍟', '🍕'];

const randomFruit = () =>
  FRUITS[getRandomNumberBetween(0, FRUITS.length - 1)];

const randomFastFood = () =>
  FAST_FOODS[getRandomNumberBetween(0, FRUITS.length - 1)];

const getRandomNumberBetween = (start, end) =>
  Math.floor(Math.random() * end) + start;

export {
  FRUITS,
  FAST_FOODS,
  randomFruit,
  randomFastFood
};
```

In this `food` module, we export two constants and two functions:

* `FRUITS`: An array of fruit emojis.
* `FAST_FOODS`: An array of fast food emojis.
* `randomFruit`: Function to get a random fruit emoji.
* `randomFastFood`: Function to get a random fast food emoji.

### Tree-Shaking in Action

Let's get back to our `main.js` file. Rename the `eat` function to `eatFruit`. Use the `randomFruit` function from our `food` module to generate a random fruit emoji:

```js
// src/main.js
import * as food from './food';

const eatFruit = () => console.log(`I eat ${food.randomFruit()}.`);

export default eatFruit;
```

Type the following command on the terminal to bundle our `main.js` module into a CommonJS format.

```bash
$ rollup src/main.js --file dist/bundle.js --format cjs
```

Let's inspect the generated `bundle.js` file:

```js
// dist/bundle.js
'use strict';

const FRUITS = ['🍏', '🍉', '🍇'];

const randomFruit = () =>
  FRUITS[getRandomNumberBetween(0, FRUITS.length - 1)];

const getRandomNumberBetween = (start, end) =>
  Math.floor(Math.random() * end) + start;

const eatFruit = () => console.log(`I eat ${randomFruit()}.`);

module.exports = eatFruit;
```

Even though we import the whole `food` module into our code, Rollup is smart enough to strip any unused parts. Both the `FAST_FOODS` constant and the `randomFastFood` function are excluded from the final bundle. Isn't that awesome?

Even though Rollup is smart enough to exclude any unused functions or modules, it's a good practice to explicitly export things that we only need. Let's modify our `main.js` file to import the `randomFruit` function only:

```js
// src/main.js
import { randomFruit } from './food';

const eatFruit = () => console.log(`I eat ${randomFruit()}.`);

export default eatFruit;
```

Run the command to bundle our `main.js` file again, we should have the same exact output.

```bash
$ rollup src/main.js --file dist/bundle.js --format cjs
```

### Three-Shaking Gotcha

Suppose we use the `default` keyword to export the four items in the `food.js` module:

```js
// Omitted for brevity...

export default {
  FRUITS,
  FAST_FOODS,
  randomFruit,
  randomFastFood
};
```

And the `main.js` file looks like this to accommodate that change:

```js
// src/main.js
import food from './food';

const eatFruit = () => console.log(`I eat ${food.randomFruit()}.`);

export default eatFruit;
```

If we bundle that `main.js` file through Rollup, the output will look like this:

```js
// dist/bundle.js
'use strict';

const FRUITS = ['🍏', '🍉', '🍇'];

const FAST_FOODS = ['🍔', '🍟', '🍕'];

const randomFruit = () =>
  FRUITS[getRandomNumberBetween(0, FRUITS.length - 1)];

const randomFastFood = () =>
  FAST_FOODS[getRandomNumberBetween(0, FRUITS.length - 1)];

const getRandomNumberBetween = (start, end) =>
  Math.floor(Math.random() * end) + start;

var food = {
  FRUITS,
  FAST_FOODS,
  randomFruit,
  randomFastFood
};

const eatFruit = () => console.log(`I eat ${food.randomFruit()}.`);

module.exports = eatFruit;
```

Even though we're not using them, we ended up having both `FAST_FOODS` and `randomFastFood` in our bundle file.

There's [another three-shaking gotcha](#another-three-shaking-gotcha) explained on the next section that you should be aware of.

## Importing NPM Package

We often need to pull a third party library from NPM. For our example, let's pull the `lodash` library. Within the project directory, type the following command to create an empty `package.json` file:

```bash
$ echo {} > package.json
```

Then install the `lodash-es` package, it's the ES modules version for `lodash`:

```bash
$ npm install lodash-es

# Or if you prefer to use Yarn
$ yarn add lodash-es
```

Open up the `src/food.js` file. Let's replace the `getRandomNumberBetween` with the lodash's `random` function instead.

```js
// src/food.js
import { random } from 'lodash-es';

const FRUITS = ['🍏', '🍉', '🍇'];

const FAST_FOODS = ['🍔', '🍟', '🍕'];

const randomFruit = () =>
  FRUITS[random(0, FRUITS.length - 1)];

const randomFastFood = () =>
  FAST_FOODS[random(0, FRUITS.length - 1)];

export {
  FRUITS,
  FAST_FOODS,
  randomFruit,
  randomFastFood
};
```

This time we're going to use the Rollup configuration file. On your project directory, create a new file named `rollup.config.js`:

```js
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'dist/cjs.js',
    format: 'cjs'
  }
};
```

With this configuration file, we're going to generate a bundle with CommonJS format. Let's build our `main.js` file!

```bash
$ rollup -c
```

You'll get a warning that it can't resolve the `lodash-es`.

```bash
(!) Unresolved dependencies
https://rollupjs.org/guide/en#warning-treating-module-as-external-dependency
lodash-es (imported by src/food.js)
created dist/cjs.js in 32ms
```

If you check the generated `dist/cjs.js` file, you'll see that the lodash code is not included. It simply `require` the `lodash-es` module.

```js
'use strict';

var lodashEs = require('lodash-es');

const FRUITS = ['🍏', '🍉', '🍇'];

const randomFruit = () =>
  FRUITS[lodashEs.random(0, FRUITS.length - 1)];

const eatFruit = () => console.log(`I eat ${randomFruit()}.`);

module.exports = eatFruit;
```

The bundle will still work if the user of your library happens to have `lodash-es` installed on their project. Of course, it's not a reliable approach to build a library. So how can we solve this?

### Resolving Third Party Modules with Plugin

Luckily, there's already a plugin to assist Rollup resolving any third party modules installed through NPM: [`rollup-plugin-node-resolve`](https://github.com/rollup/rollup-plugin-node-resolve). Let's install this plugin as our dev-dependency:

```bash
$ npm install rollup-plugin-node-resolve -D

# If you prefer to use Yarn
$ yarn add rollup-plugin-node-resolve -D
```

Next, we need to register this plugin within our `rollup.config.js` file:

```js
// rollup.config.js
import resolve from 'rollup-plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/cjs.js',
    format: 'cjs'
  },
  plugins: [
    resolve()
  ]
};
```

Let's bundle our `main.js` file once again:

```js
$ rollup -c
```

There's no warning message this time! If you check the generated `dist/cjs.js` file, you should find the lodash code in the bundle.

But wait, why is there so many lodash codes in our bundle file? Though we only import the `random` function. 🤔

### Another Three-Shaking Gotcha

Performing static analysis in a dynamic programming language like JavaScript is hard. Rollup has to be careful in removing any unused code to guarantee that the final bundle still works correctly.

If an imported module appears to have some side-effects, Rollup needs to be conservative and includes those side-effects. Even though it may be a false positive, just like in our lodash case above.

In our lodash case, we can get around this by importing the submodule instead:

```js
// src/food.js
import random from 'lodash-es/random';

const FRUITS = ['🍏', '🍉', '🍇'];

const FAST_FOODS = ['🍔', '🍟', '🍕'];

const randomFruit = () =>
  FRUITS[random(0, FRUITS.length - 1)];

const randomFastFood = () =>
  FAST_FOODS[random(0, FRUITS.length - 1)];

export {
  FRUITS,
  FAST_FOODS,
  randomFruit,
  randomFastFood
};
```

### Importing CommonJS Module

The `lodash-es` package is using the ES module format. So Rollup can process it out of the box. Unfortunately, the majority of packages published on NPM are using the CommonJS format.

Let's replace the `lodash-es` package with its CommonJS counterpart: `lodash` and see if Rollup can handle it.

```bash
$ npm uninstall lodash-es
$ npm i lodash

# If you use Yarn
$ yarn remove lodash-es
$ yarn add lodash
```

Don't forget to update the import statement on the `food.js` module:

```js
// src/food.js
import random from 'lodash/random';

const FRUITS = ['🍏', '🍉', '🍇'];

const FAST_FOODS = ['🍔', '🍟', '🍕'];

const randomFruit = () =>
  FRUITS[random(0, FRUITS.length - 1)];

const randomFastFood = () =>
  FAST_FOODS[random(0, FRUITS.length - 1)];

export {
  FRUITS,
  FAST_FOODS,
  randomFruit,
  randomFastFood
};
```

Let's run the Rollup command to bundle our `main.js` file:

```bash
$ rollup -c
```

We'll end up having an error message like this:

```bash
src/main.js → dist2/cjs.js...
[!] Error: 'default' is not exported by node_modules/lodash/random.js
https://rollupjs.org/guide/en#error-name-is-not-exported-by-module-
src/food.js (1:7)
1: import random from 'lodash/random';
```

It turns out Rollup can't process the CommonJS format. Fortunately, there's already a plugin to solve this issue: [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs). This plugin will convert any CommonJS modules into the ES module format. That way Rollup can process them.

Let's install this plugin:

```bash
$ npm install rollup-plugin-commonjs -D

# Or if you use yarn
$ yarn add rollup-plugin-commonjs -D
```

Don't forget to register it on `rollup.config.js` file:

```js
// rollup.config.js
import commonjs from 'rollup-plugin-commonjs';
import resolve from 'rollup-plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/cjs.js',
    format: 'cjs'
  },
  plugins: [
    resolve(),
    commonjs()
  ]
};
```

Try to bundle our `main.js` file again. There should be no error this time.

```bash
$ rollup - c
```

> If you register other plugins that can also transform your module, make sure that the `rollup-plugin-commonjs` comes before any of that plugins. It's to prevent the other plugins from making changes that may break the CommonJS detection.

### Peer Dependencies
