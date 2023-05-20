---
layout: post
title: How To Build A NPM Package, A Production Ready Tutorial
date: October 06, 2021
author: Álvaro José Agámez Licha
overview: npm is the world's largest software registry. Open source developers from every continent use npm to share and borrow packages, and many organizations use npm to manage private development as well.
permalink: how-to-build-a-npm-package.html
tags:
- Node.js
- NPM
- Software Development
- Package
- Tutorial
---

# About npm

<a href="https://www.npmjs.com/" target="_blank">npm</a> is the world's largest software registry. Open source developers from every continent use npm to share and borrow packages, and many organizations use npm to manage private development as well.

npm consists of three distinct components:

* The website
* The Command Line Interface (CLI)
* The registry

Use the <a href="https://npmjs.com/" target="_blank">website</a> to discover packages, set up profiles, and manage other aspects of your npm experience. For example, you can set up organizations to manage access to public or private packages.

The <a href="https://docs.npmjs.com/cli/npm" target="_blank">CLI</a> runs from a terminal, and is how most developers interact with npm.

The <a href="https://docs.npmjs.com/misc/registry" target="_blank">registry</a> is a large public database of JavaScript software and the meta-information surrounding it.

In this tutorial I will teach you how to create a npm package with TypeScript, TDD, test coverage, coding standard, coding linting, good practices and basic CI with Github Actions.

First of all let me make a summary of all the tools that we will use:

* <a href="https://github.com/" target="_blank">GitHub</a>: Is a provider of Internet hosting for software development and version control using Git. It offers the distributed version control and source code management functionality of Git, plus its own features.
* <a href="https://www.typescriptlang.org/" target="_blank">TypeScript</a>: TypeScript is a strongly typed programming language which builds on JavaScript giving you better tooling at any scale.
* <a href="https://github.com/standard/ts-standard" target="_blank">TS Standard</a>: TypeScript Style Guide, with linter and automatic code fixer based on <a href="https://github.com/standard/ts-standard" target="_blank">StandardJS</a>.
* <a href="https://github.com/avajs/ava" target="_blank">AVA</a>: A test runner for Node.js with a concise API, detailed error output, embrace of new language features and process isolation that lets you develop with confidence.
* <a href="https://istanbul.js.org/" target="_blank">Istanbul</a>: Is a test coverage tool that works with many different frameworks. It tracks which parts of your code are executed by your unit tests.
* <a href="https://www.conventionalcommits.org/" target="_blank">Conventional Commits</a>: A specification for adding human and machine readable meaning to commit messages.
* <a href="https://github.com/typicode/husky" target="_blank">husky</a>: Modern native Git hooks made easy, husky improves your commits and more 🐶 woof!.
* <a href="https://github.com/conventional-changelog/standard-version#readme" target="_blank">Standard Version</a>: A utility for versioning using <a href="https://semver.org/" target="_blank">semver</a> and CHANGELOG generation powered by <a href="https://www.conventionalcommits.org/" target="_blank">Conventional Commits</a>.
* <a href="https://verdaccio.org/" target="_blank">Verdaccio</a>: A lightweight private npm proxy registry.
* <a href="https://editorconfig.org/" target="_blank">EditorConfig</a>: Helps maintain consistent coding styles for multiple developers working on the same project across various editors and IDEs.

Ok, this may seem like a fairly extensive list to build an npm package, but trust me, you will see that when we start writing the code everything will fit in easily, since several of these tools we will use without realizing it, such as the linter and others we will use them at specific times, such as when making our commits or releases.

# What we will build?
I will teach you how I build one of my own npm packages, a set of utilities that help me in my other projects.

The package will consist of a set of small functions written in TypeScript, and for which we will always start by writing the corresponding unit test, following the TDD development process.

# Setup

This config is for Node.js v16.13.0 and NPM 7.0.0 or higher.

## Git Repository

To manage and publish the code of our package we will use <a href="https://github.com/" target="_blank">GitHub</a>, so we must create a new repository.  It's important to add a `README` file, a `.gitignore` choosing the `Node template` and a `license` (I use MIT). After the repository is created, we must clone it in our development environment.

```bash
$ git clone git@github.com:your_user/utils.git
```

## Create Package

To create out package scaffold we use `npm init`; we can also create our package under a scope or organization, I personally always use this approach and to do so we must use the `--scope` param.

```bash
npm init -y --scope=@my-org
```

After creating our package, which basically means creating our `package.json`, we need to edit this last file to add our information to the author key, modify the license type and add the keywords that will make the npm registry able to list our package in a better way.

## EditorConfig

To generate my configuration file I use a VS Code extension called <a href="https://marketplace.visualstudio.com/items?itemName=nepaul.editorconfiggenerator" target="_blank">EditorConfigGenerator</a>, which basically is in charge of generating a configuration file with the default values that are the ones that I usually use.  You can change these values to whatever you want, but the truth is that the default values are the most recommended.

## NPM Packages Dependencies

Now we will proceed to install the development dependencies that we will use for our package.

```bash
$ npm i -D typescript @types/node ts-standard ava nyc standard-version @commitlint/{cli,config-conventional,format} husky
```

## TypeScript Setup

First we must create our configuration file for TypeScript:

```bash
$ touch tsconfig.json
```

Now we must complete the configuration file with a subset of options that will help us to develop our package:

```json
{
  "compilerOptions": {
    "allowJs": false,
    "declaration": true,
    "declarationDir": "./lib/types",
    "esModuleInterop": true,
    "lib": [
      "ES2021"
    ],
    "module": "commonjs",
    "moduleResolution": "node",
    "noImplicitAny": true,
    "outDir": "./lib",
    "rootDir": "./src",
    "sourceMap": true,
    "strict": true,
    "target": "ES2021"
  },
  "include": [
    "src/**/*.ts"
  ],
  "exclude": [
    "lib",
    "./node_modules"
  ]
}
```

Finally we must create a couple of npm scripts to compile our TypeScript code:

```bash
$ npm set-script clean "rm -rf lib"
$ npm set-script build "npm run clean && tsc"
$ npm set-script build:watch "npm run build -- -w"
```

## Linter Configuration

The only settings we need to define for `ts-standard` are the files we don't want it to check; unfortunately the only way to configure `ts-standard` is through the `package.json`, so we must add the following configuration.

```json
// package.json
"ts-standard": {
  "ignore": [
    "lib",
    "test"
  ]
},
```

Now, we need to add our npm scripts to execute the linter:

```bash
$ npm set-script lint "ts-standard"
$ npm set-script lint:fix "ts-standard --fix"
```

The `lib` directory is the directory where our JavaScript is generated after the compilation process.

## Test Runner Configuration

We can configure ava in two different ways, using the `package.json` or using a configuration file, but for our package we are going to use the configuration file to make our `package.json` file as clean as possible.

```bash
$ touch ava.config.js
$ npm set-script test "ava"
$ npm set-script test:watch "ava --watch"
$ npm set-script coverage "nyc ava"
```

Now we can define our test runner settings, by the way, ava doesn't have TypeScript support by default, but this is not a problem, we can activate it ourselves very easily.

```javascript
export default {
  files: [
    'test/**/*.spec.ts'
  ],
  extensions: [
    'ts'
  ],
  require: [
    'ts-node/register'
  ]
}
```

## Configure Commitlint

Commitlint configuration is very simple, we just have to create its configuration file in the root directory of our package.

```bash
# Configure commitlint to use conventional config
$ echo "module.exports = { extends: ['@commitlint/config-conventional'] }" > commitlint.config.js
```

## Configure Husky Hooks

```bash
$ npm set-script prepare "husky install"
$ npm set-script release "standard-version"
$ npm run prepare
$ npx husky add .husky/pre-commit "npm run lint"
$ npx husky add .husky/pre-commit "npm test"
$ npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
$ git add .husky/pre-commit

# Testing our pre-commit hook
$ git commit -m "Keep calm and commit"
# `npm run lint` and `npm test` will run every time you commit
```

# Time To Code

Ok enough setup, it is time for us to start writing the code of our package, as we will use TDD (Test Driven Development), first we will write the first test for the `getType` function, this function must return the type of the value passed as a parameter.

Since we are using TDD to develop our package, we are going to start with the minimum test that our function must pass.

```javascript
// test/utils.spec.ts
import test from 'ava'

import * as utils from './../src'

test('should returns the correct type', t => {
  t.is(utils.getType(), 'Undefined')
})
```

To make our development faster, let's start our test runner in `watch` mode so that it is monitoring the changes in our tests and code; to do this we must execute the command `npm run test:watch`.

Obviously our test will fail because we haven't defined our `getType()` function yet, so let's do that first.

```javascript
// src/utils.ts
export const getType = (value: unknown = undefined): string => { }
```

Ok, we have some progress, now we need to make our first test pass, so let's return the expected value.

```javascript
// src/utils.ts
export const getType = (value: unknown = undefined): string => {
  return 'Undefined'
}
```

Perfect, our first test is passing, although it is quite useless from a practical point of view, so let's speed things up a bit to get to move to the next sections of this tutorial.

```javascript
// test/utils.spec.ts
test('should returns the correct type', t => {
  t.is(utils.getType({}), 'Object')
  t.is(utils.getType(Object.create(null)), 'Object')
  t.is(utils.getType([]), 'Array')
  t.is(utils.getType(new Array()), 'Array')
  t.is(utils.getType(new Date()), 'Date')
  t.is(utils.getType(String()), 'String')
  t.is(utils.getType(''), 'String')
  t.is(utils.getType('123'), 'String')
  t.is(utils.getType(123), 'Number')
  t.is(utils.getType(123.4), 'Number')
  t.is(utils.getType(true), 'Boolean')
  t.is(utils.getType(false), 'Boolean')
  t.is(utils.getType(BigInt(1)), 'BigInt')
  t.is(utils.getType(() => {}), 'Function')
  t.is(utils.getType(BigInt), 'Function')
  t.is(utils.getType(undefined), 'Undefined')
  t.is(utils.getType(), 'Undefined')
})
```

```javascript
// src/utils.ts
export const getType = (value: unknown = undefined): string => {
  return 'Undefined'
}
```