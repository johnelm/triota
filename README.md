# IOTA

<!-- TOC -->

- [Introduction](#introduction)
  - [Key features](#key-features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [as a devDependency](#as-a-devdependency)
  - [for the IOTA fiddler or contributor](#for-the-iota-fiddler-or-contributor)
- [Feature Tourtorial](#feature-tourtorial)
- [Feature Backlog and Issues](#feature-backlog-and-issues)

<!-- /TOC -->
## Introduction
IOTA<sup>*</sup> is a functional test automation framework based on Selenium [WebDriverJS](http://seleniumhq.github.io/selenium/docs/api/javascript/index.html) and Node.js.  Its main goals are rapid scripting, adaptability and maintainability.

Feel free to reach out to me here with any questions, I'd love to hear from you!

### Key features
- Simple text based DSL for authoring tests without code (Javascript is also supported)
- Easy reuse of flows and other test assets
- Automatic generation of manual test instructions
- Queueing and concurrency support
- Configurable retry logic for handling network and dependency flakiness
- Recaptcha detection and handling
- Easy API testing and utility calls
- Extensible, pluggable architecture
- Reporting, including code coverage

> \* Headsup: `iota` will soon be renamed to `triota` due to an npm naming conflict.

## Prerequisites

IOTA requires Node.js v7.4.0 or newer.

It has not been used or tested on Windows.

## Installation

Basic installation is the typical:

```bash
npm install -g @fdx/iota --registry=https://registry.npmjs.intuit.net` # ( on Intuit network or VPN )

```
This will install the framework and all its dependencies, including the Selenium server jar file and Chromedriver.  No other steps are required to drive your local Chrome, or any browser via a remote Selenium server or grid.

### as a devDependency
IOTA can be used as a devDependency in your existing node.js project, so you can use it like any other test framework in your npm scripts.

```bash
# [ pwd: your own node project's root ]
npm install --save-dev @fdx/iota --registry=https://registry.npmjs.intuit.net
```

### for the IOTA fiddler or contributor

If you're making changes in a local clone of IOTA, a few steps are necessary to make sure your latest changes are used when you run it.

```bash
# first unlink any previous global iota installation via `npm unlink -g iota`
# [ pwd: your IOTA clone's root ]
npm install
npm link
```

If you're also using IOTA as a dev dependency in another Node.js project, go there run this command:

```bash
# [ pwd: your own project's root ]
npm link iota
```

Note: IOTA's launcher script automatically compiles any changed *.js files, so you don't have to remember to run babel.  You're welcome.  :-)

## Feature Tourtorial

For a closer look at IOTA's usage and features, head on over to the [Feature Tourtorial](docs/tourtorial/index.md).

## Feature Backlog and Issues

IOTA Bugs and Design Debt are tracked [here](https://github.intuit.com/CTO-Dev-FDS-FDX/qa-iota/issues).


