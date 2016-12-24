# Markdown Magic

✨ Add a little magic to your markdown ✨

## About

Markdown magic uses comment blocks in markdown files to automatically sync or transform it's contents. These code comments are hidden in markdown and when viewed as HTML.

- Automatically keep markdown files up to date from local or remote code sources
- Transform markdown content with custom transform functions
- Render markdown with any template engine
- Automatically generate a table of contents

This `README.md` is generated with `markdown-magic` [view the raw file](https://raw.githubusercontent.com/DavidWells/markdown-magic/master/README.md) to see how.

[Video demo](http://www.youtube.com/watch?v=4V2utrvxwJ8) • [Example Repo](https://github.com/DavidWells/repo-using-markdown-magic)

## Table of Contents
<!-- ⛔️ AUTO-GENERATED-CONTENT:START (TOC) -->
- [About](#about)
- [Install](#install)
- [Usage](#usage)
  * [API](#api)
  * [Configuration Options](#configuration-options)
- [Transforms](#transforms)
  * [`CODE`](#code)
  * [`REMOTE`](#remote)
  * [`TOC`](#toc)
- [Custom Transforms](#custom-transforms)
- [Plugin Example:](#plugin-example)
- [Other usage examples](#other-usage-examples)
- [Custom Transform Demo](#custom-transform-demo)
- [Prior Art](#prior-art)
<!-- ⛔️ AUTO-GENERATED-CONTENT:END -->

## Install

```bash
npm install markdown-magic --save-dev
```

## Usage
<!-- ⛔️ AUTO-GENERATED-CONTENT:START (CODE:src=./examples/basic-usage.js&order=last) -->
<!-- The below code snippet is automatically added from ./examples/basic-usage.js -->
```js
import markdownMagic from 'markdown-magic'
import path from 'path'

const markdownPath = path.join(__dirname, 'README.md')
markdownMagic(markdownPath)
```
<!-- ⛔️ AUTO-GENERATED-CONTENT:END *-->

<!-- ⛔️ AUTO-GENERATED-CONTENT:START (RENDERDOCS:path=./index.js)
- Do not remove or modify this section -->
### API
```js
markdownMagic(filePath, config, callback)
```
- `filePaths` - *String or Array* - Path or glob pattern. Uses [globby patterns](https://github.com/sindresorhus/multimatch/blob/master/test.js)
- `config` - See configuration options below
- `callback` - callback to run after markdown updates
<!-- ⛔️ AUTO-GENERATED-CONTENT:END - Do not remove or modify this section -->

<!-- ⛔️ AUTO-GENERATED-CONTENT:START (RENDERDOCS:path=./lib/processFile.js)
- Do not remove or modify this section -->
### Configuration Options

`transforms` - *Object* - (optional) Custom commands to transform block contents, see transforms & custom transforms sections below.

`outputDir` - *String* - (optional) Change output path of new content. Default behavior is replacing the original file

`matchWord` - *String* - (optional) Comment pattern to look for & replace inner contents. Default `AUTO-GENERATED-CONTENT`

`DEBUG` - *Boolean* - (optional) set debug flag to `true` to inspect the process
<!-- ⛔️ AUTO-GENERATED-CONTENT:END - Do not remove or modify this section -->


## Transforms

Markdown Magic comes with a couple of built in transforms for you to use or you can extend it with your own transforms. See 'Custom Transforms' below.

<!-- ⛔️ AUTO-GENERATED-CONTENT:START (RENDERDOCS:path=./lib/transforms/index.js) - Do not remove or modify this section -->
### `CODE`

Get code from file or URL and put in markdown

**Options:**
- `src`: The relative path to the code to pull in, or the `URL` where the raw code lives
- `syntax` (optional): Syntax will be inferred by fileType if not specified

**Example:**
```md
<-- MATCHWORD:START (CODE:src=./relative/path/to/code.js) -->
This content will be dynamically replaced with code from the file
<-- MATCHWORD:END -->
```
---

### `REMOTE`

Get any remote Data and put in markdown

**Options:**
- `url`: The URL of the remote content to pull in

**Example:**
```md
<-- MATCHWORD:START (REMOTE:url=http://url-to-raw-md.md) -->
This content will be dynamically replace from the remote url
<-- MATCHWORD:END -->
```
---

### `TOC`

Generate table of contents from markdown file

**Example:**
```md
<-- MATCHWORD:START (TOC) -->
toc will be generated here
<-- MATCHWORD:END -->
```
---
<!-- ⛔️ AUTO-GENERATED-CONTENT:END - Do not remove or modify this section -->

## Custom Transforms

Markdown Magic is extendable via plugins.

Plugins allow developers to add new transforms, use different rendering engines or any other logic you might want in `config.commands`.

This code is used to generate **this markdown file**:

<!-- ⛔️ AUTO-GENERATED-CONTENT:START (CODE:src=./examples/generate-readme.js) -->
<!-- The below code snippet is automatically added from ./examples/generate-readme.js -->
```js
const fs = require('fs')
const path = require('path')
const execSync = require('child_process').execSync
const markdownMagic = require('../index') // 'markdown-magic'

const config = {
  transforms: {
    /* Update the content in comment matching:
       AUTO-GENERATED-CONTENT (customTransform:optionOne=hi&optionOne=DUDE)
    */
    customTransform(content, options) {
      console.log('original innerContent', content)
      console.log(options) // { optionOne: hi, optionOne: DUDE}
      return `This will replace all the contents of inside the comment ${options.optionOne}`
    },
    /* Update the content in comment matching:
      AUTO-GENERATED-CONTENT (RENDERDOCS:path=../file.js)
    */
    RENDERDOCS(content, options) {
      const contents = fs.readFileSync(options.path, 'utf8')
      const docBlocs = require('dox').parseComments(contents, { raw: true, skipSingleStar: true })
      let updatedContent = ''
      docBlocs.forEach((data) => {
        updatedContent += `${data.description.full}\n\n`
      })
      return updatedContent.replace(/^\s+|\s+$/g, '')
    },
    pluginExample: require('./plugin-example')({ addNewLine: true })
  }
}

/* This example callback automatically updates Readme.md and commits the changes */
const callback = function autoGitCommit(err, output) {
  // output is array of file information
  output.forEach(function(data) {
    const mdPath = data.outputFilePath
    const gitAdd = execSync(`git add ${mdPath}`, {}, (error) => {
      if (error) console.warn(error)
      console.log('git add complete')
      const msg = `${mdPath} automatically updated by markdown-magic`
      const gitCommitCommand = `git commit -m '${msg}' --no-verify`
      execSync(gitCommitCommand, {}, (err) => {
        if (err) console.warn(err)
        console.log('git commit automatically ran. Push up your changes!')
      })
    })
  })
}

const markdownPath = path.join(__dirname, '..', 'README.md')
markdownMagic(markdownPath, config, callback)
```
<!-- ⛔️ AUTO-GENERATED-CONTENT:END -->

## Plugin Example:

<!-- ⛔️ AUTO-GENERATED-CONTENT:START (CODE:src=./examples/plugin-example.js) -->
<!-- The below code snippet is automatically added from ./examples/plugin-example.js -->
```js
/**
 * Custom Transform Plugin example
 */
const merge = require('deepmerge')

module.exports = function customPlugin(pluginOptions) {
  // set plugin defaults
  const defaultOptions = {
    addNewLine: false
  }
  const userOptions = pluginOptions || {}
  const pluginConfig = merge(defaultOptions, userOptions)
  // return the transform function
  return function (content, options) {
    const newLine = (pluginConfig.addNewLine) ? '\n' : ''
    const updatedContent = content + newLine
    return updatedContent
  }
}
```
<!-- ⛔️ AUTO-GENERATED-CONTENT:END -->

[View the raw file](https://raw.githubusercontent.com/DavidWells/markdown-magic/master/README.md) file and run `npm run docs` to see this plugin run
<!-- ⛔️ AUTO-GENERATED-CONTENT:START (pluginExample) DO not edit ⛔️ -->
This content is altered by the `pluginExample` plugin registered in `examples/generate-readme.js`

<!-- ⛔️ AUTO-GENERATED-CONTENT:END -->

## Other usage examples

- [Serverless Community Plugin Repo](https://github.com/serverless/community-plugins/blob/master/generate-readme.js) this example takes a `json` file and converts it into a github flavored markdown table

## Custom Transform Demo

View the raw source of this `README.md` file to see the comment block and see how the `customTransform` function in `examples/generate-readme.js` works

<!-- ⛔️ AUTO-GENERATED-CONTENT:START (customTransform:optionOne=hi&optionOne=DUDE) - Do not remove or modify this section -->
This will replace all the contents of inside the comment DUDE
<!-- ⛔️ AUTO-GENERATED-CONTENT:END - Do not remove or modify this section -->

## Prior Art

This was inspired by [Kent C Dodds](https://twitter.com/kentcdodds) and [jfmengels](https://github.com/jfmengels)'s [all contributors cli](https://github.com/jfmengels/all-contributors-cli) project.
