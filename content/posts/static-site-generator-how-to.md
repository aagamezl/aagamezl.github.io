---
layout: post
title: Static Site Generator - How I Created
date: September 12, 2021
author: Álvaro José Agámez Licha
overview: For a long time I had wanted to create my personal blog, I thought about using an online blogging system like dev.to, or using a static generator like Jekyl, Hugo or the like, but I don't know, none of them motivated me.
permalink: static-site-generator-how-to.html
tags:
- node.js
- static site generator
- site generator
- tools
---

# Inception


For a long time I had wanted to create my personal blog, I thought about using an online blogging system like dev.to, or using a static generator like Jekyl, Hugo or, the like, but I don't know, none of them motivated me.

So I decided to do what I almost always do, make my own tool to generate my personal blog and at the same time publish it with an Open Source project and a <a href="https://www.npmjs.com/package/@devnetic/static-site-generator" target="_blank">NPM package</a>.

The first thing I did was think about how I wanted this generator to work, what functionalities it would have, and how it should be used. Among the main functionality should be something that I give a lot of value, and that is simplicity, both in use and in installation, taking this as the main functionality, I made a list of what I wanted to have.

- Ease of use and installation.
- High speed of compilation/generation of content.
- Small size.
- Few dependencies.
- Written entirely in JavaScript.
- Support for different themes.
- Support for different template systems.
- Live reload.
- A CLI to manage the generator.

With this list of functionalities, I started the mental design of the project and when I say mental design I mean that I try to imagine how the application will be organized, what packages I can use to solve some problems and I begin to imagine how I would like to use the application if I were a user.

After a few hours of mental process, I started to write some lines of code, the basic structure of the project, I added some functions that I knew I would need, but it was time to start working on the basis of the project, which is basically converting content (markdown in my case) in a series of HTML, CSS and JavaScript files and here I must confess that I had no idea how other similar systems worked or solved the problems that I wanted to solve, so I started to investigate some of the static content generators most popular, and I focused on Jekyl and Hugo.

# Discovery

Almost immediately I discovered that there was something called Frontmatter which is used to add metadata in Markdown files, so I had to add a new item to my list of functionalities since this metadata block is something that adds a lot of important information to the content that includes it as well as for the system in general.

At first, I tried to parse both the Markdown and the Frontmatter on my own, but after valuable hours spent on this task, I realized that it was not worth investing time and energy on this, since it was not within the scope of the project, so after some time searching in NPM I decided on two quite popular packages to perform these tasks, well, actually I chose three packages since I had to use an additional one to format the code blocks:

1. Markdown parser: [markdown-it](https://www.npmjs.com/package/markdown-it)
2. Frontmatter parser: [gray-matter](https://www.npmjs.com/package/gray-matter)
3. Syntax highlighter: [highlight.js](https://www.npmjs.com/package/highlight.js)

# Development

Ok, at this point I had already solved the content generation part, now there was only a little code left to work with the file system and for this, I chose [glob](https://www.npmjs.com/package/glob ), as it is perhaps the best option for working with file search patterns and automatic search depth in paths.

After a few hours of development, trial and error, debugging, etc. I managed to get the base of the generator, the part that takes care of almost all the "magic" by calling it somehow:

```javascript
/**
 * Generate the site structure, compile the markdown and compile the content
 *
 * @param {object} data
 * @param {object} config
 * @return {object}
 */
const generateSite = async (data, config) => {
  data = Array.isArray(data) ? data : [data]

  return data.map(parseFrontmatter)
    .map(compileMarkdown)
    .filter(page => page.layout !== undefined)
    .sort(sortContent)
    .reduce(compileContent, {})
}
```

It seems like little code for a static content generator, and yes, there is a lot more code that makes everything work properly to get our content compiled correctly, but that function is the heart of the system.

A more general view of the system would be the `buildSite` function, which is responsible for orchestrating all the operations necessary to generate the content.

```javascript
/**
 * Build the site; this function is called by the the command line option and by
 * chokidar initalization when the build directory is empty.
 *
 * @param {object} config
 * @return {boolean}
 */
const buildSite = async (config) => {
  // Get the content data to be compiled
  const data = await getData(
    getPath(config.content.path),
    getGlobPattern(config.content.files)
  )

  const site = await generateSite(data, config)

  await writeContent({ ...site, ...config.site }, config)
  await copyAssets(config)

  return true
}
```

The whole system is made up of around 900 lines of code (including JSDoc blocks, so the number of actual lines of code is much less); If you want to take a look at all the code of the system you can do it in its [GitHub Repository](https://github.com/aagamezl/static-site-generator).

Another key point of the system is the CLI to handle all the available options, such as generating the initial scaffolding, compiling the content, or launching the development server with live reloading. For this task, I chose [@devnetic/cli](https://www.npmjs.com/package/@devnetic/cli) one of my own NPM packages, which makes building CLI applications really easy and fast.

```javascript
const { format, getParams, usage } = require('@devnetic/cli')

// Get the command line params
const params = getParams()

// Show the help menu when the command line tool is called with -h or --help param
if (params.help || params.h || Object.keys(params).length === 0) {
    usage('Usage: ssg <option> [modifier]')
    .option(['-b', '--build'], '\t\t\tRun the build process')
    .option(['-c', '--clean [-y, --yes]'], '\tClean the build directory')
    .option(['-n', '--new'], '\t\t\tGenerate a new site')
    .option(['-v', '--version'], '\t\tDisplay version')
    .option(['-w', '--watch'], '\t\t\tRun the build process watching changes')
    .option(['-h', '--help'], '\t\t\tShow this help')
    .epilog(`Copyright ${new Date().getFullYear()} - Static Site Generator`)
    .show()

    process.exit(0)
}

// Create a text formmater
const errorLog = format.bold().red

console.log(errorLog('This error message will be red and bold'))
```

Last but not least, for the tasks managing the filesystem, real-time file change monitoring, and development server with live reload I chose the NPM packages [Chokidar](https://www.npmjs.com/package/chokidar), [fs-extra](https://www.npmjs.com/package/fs-extra) and [live-server](https://www.npmjs.com/package/live-server)  respectively.

# Usage

Ok, now that I have outlined how I built and composed the system, let's see how it would be installed and used; Being an NPM package we can install it like any other:

```bash
$ npm i -D @devnetic/static-site-generator
```

Now we have access to the command `ssg` (an acronym for static site generator), with which we can perform all the available tasks in the system. The first thing we would like to do is generate the initial scaffolding for our site, for which we will use the `npx ssg -n` command.

This will generate the following scaffolding:

```bash
- content (or whatever name that you provide in the creation step)
|-- pages
|-- posts
- public (or whatever name that you provide in the creation step)
- template-system
- themes
|-- default
```

Some of these directories are generated by convention and cannot be changed, such as `template-system` and` themes`, while the others can be changed at the time of the initial scaffolding creation or later manually, renaming the directories and changing the corresponding values in the configuration file.

The content that the generator will use for our sites must be added in 2 different directories, each of which has a well-defined function; these directories are pages and post respectively.

- `page directory`: This directory contains the main pages, such as home, about, contact, projects, or any main content that should generate a navigation element in the main menu.

- `post directory`: This directory contains the posts or secondary content of the site and it will not generate a menu in the main menu.

Now we can generate the site with the command `npx ssg -b` or with the command` npx ssg -w`; the last one I will run the developer's server and it will generate the site as long as the build directory is empty, otherwise it will start with the content that already exists.

To clean all the contents of the build directory, we can use the command `npx ssg -c`; this operation is destructive and cannot be undone, so confirmation will be required before the operation is performed. In case we want to skip the commit, we can use the `-y` modify, so the full command would be `npx ssg -c -y`.