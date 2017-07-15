# IOTA

<!-- TOC -->

- [Introduction](#introduction)
  - [Key features](#key-features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [as a devDependency](#as-a-devdependency)
  - [for the IOTA fiddler or contributor](#for-the-iota-fiddler-or-contributor)
- [Feature tour-torial](#feature-tour-torial)
  - [Fast, easy script authoring](#fast-easy-script-authoring)
  - [Running scenarios](#running-scenarios)
    - [dry run: manual test instructions, for free](#dry-run-manual-test-instructions-for-free)
    - [in Chrome](#in-chrome)
    - [on other platforms, browsers and Selenium grids](#on-other-platforms-browsers-and-selenium-grids)
  - [Queues and concurrency](#queues-and-concurrency)
    - [configsets and suites](#configsets-and-suites)
    - [Running queued scenarios with concurrency](#running-queued-scenarios-with-concurrency)
  - [Handling 'flaky tests'](#handling-flaky-tests)
    - [Retry for individual scenario steps](#retry-for-individual-scenario-steps)
    - [Requeueing scenarios](#requeueing-scenarios)
    - [Retry decorators and Recaptcha detection](#retry-decorators-and-recaptcha-detection)
  - [More features and advanced usage](#more-features-and-advanced-usage)
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

Basic installation is typical:

```bash
npm install -g @fdx/iota --registry=https://registry.npmjs.intuit.net` # ( on Intuit network or VPN )
```

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

## Feature tour-torial

Let's have a closer look at IOTA's usage and features.  

You can follow along by cloning this repository locally, and navigating to the `examples/feature-tour` directory.  Basic knowledge of selenium and page objects is helpful, but not required.

### Fast, easy script authoring
IOTA uses an easy-to-learn YAML-based DSL, so you can get started quickly.  JavaScript is also supported for utility and API calls.

Here's a taste:

```yaml
# scenarios.yaml

testdata:                                                       # ⬅ test data definition:

  url:                                                          # ⬅ object, referenced using dot notation
    duckduckgo: https://duckduckgo.com/
  searchPhrase: design debt economics                           # ⬅ simple variable / value
    
pages:.                                                         # page object definitions:

  DuckDuckGo search page:                                       # ⬅ page
    elements:
      Search field:                                             # ⬅ element
        locator:
          id: search_form_input_homepage                           # ⬅ locator (i.e. id, xpath etc)
      Search button:
        locator:
          id: search_button_homepage

  DuckDuckGo results page:
    elements:
      Top search result:
        locator:
          id: r1-0

scenarios:                                                      # ⬅ scenario definitions:

  Navigate to the DuckDuckGo search page:
    actions:
      - browser.get( url.duckduckgo )                           # ⬅ url.duckduckgo is from testdata

  Search for something using DuckDuckGo:
    actions:
      - { Scenario: 'Navigate to the DuckDuckGo search page' }  # ⬅ scenarios can include other scenarios
      - DuckDuckGo search page:                                 # ⬅ page
        - Search field:                                         # ⬅ element
          - sendKeys( '${searchPhrase} article' )                 
        - Search button:
          - click()
      - DuckDuckGo results page:                         
        - Top search result:                         
          - isDisplayed()                                       # ⬅ actions and assertions are performed
          - getText().containsIgnoreCase( searchPhrase )        # ⬅ on the containing element
 
```
See if you can pick out these concepts in the example:

- user actions and assertions are captured in `scenarios`, which can be included in other composite scenarios.  You can easily factor your scenarios as reusable building blocks to model your user flows and avoid duplication.
- `pages`: page objects are authored as simple collections of elements and their locators.  
- `testdata` is managed as a graph of objects and values which can be used like variables (and in template strings) throughout scenarios and other test assets.
- page and element names are nested hierarchically in scenarios; commands and assertions like `click()` and `isDisplayed()` are performed on their containing elements

### Running scenarios

Running an IOTA scenario locally is easy.  Let's see how it's done.

#### dry run: manual test instructions, for free
Descriptive object names (with spaces) are supported, and encouraged.  This enables the generation of *plain English* manual test instructions, for sharing, test case documentation, and for Steps to Reproduce in bug reports.

Try this `iota scenario` command:

```bash
# [ pwd: examples/feature-tour ]:
$ iota scenario 'Search for something using DuckDuckGo' --dry-run           # note the dry-run option

INFO: performing the scenario: "Search for something using DuckDuckGo"
SCRIPT: Test case steps:
( start scenario steps )
- Search for something using DuckDuckGo
  - Navigate to the DuckDuckGo search page
    - Navigate in your browser to https://duckduckgo.com/
  - Type 'design debt economics article' into the Search field
  - Click the Search button
  - The Top search result should be displayed
  - The Top search result should contain the text, 'design debt economics' (ignoring case)
( end scenario steps )
```
The `--dry-run` option produces the manual script for our example 'Search for something' scenario, without actually running it.  Note how the scenario, page and element names from our DuckDuckGo example are used in the output.

#### in Chrome
Now, drop the `--dry-run` option from the command.  This will run the example scenario in Chrome on your local machine (assuming IOTA is properly installed).

```bash
# [ pwd: examples/feature-tour ]:
$ iota scenario 'Search for something using DuckDuckGo'

INFO: performing the scenario: "Search for something using DuckDuckGo"
SCRIPT: Test case steps:
( start scenario steps )
- Search for something using DuckDuckGo
  - Navigate to the DuckDuckGo search page
    - Navigate in your browser to https://duckduckgo.com/
  - Type 'design debt economics article' into the Search field
  - Click the Search button
  - The Top search result should be displayed
  - The Top search result should contain the text, 'design debt economics' (ignoring case)
( end scenario steps )
SCRIPT: ✓ Navigate in your browser to https://duckduckgo.com/
SCRIPT: ✓ Type 'design debt economics article' into the Search field
SCRIPT: ✓ Click the Search button
SCRIPT: ✓ The Top search result should be displayed
SCRIPT: ✓ The Top search result should contain the text, 'design debt economics' (ignoring case)
INFO:

IOTA Execution Statistics:

| tests | passes | pending | failures | duration |
| ----- | ------ | ------- | -------- | -------- |
| 5     | 5      | 0       | 0        | 3508     |
```

#### on other platforms, browsers and Selenium grids
Your local Chrome browser is used when no webdriver configuration is provided.  If we provide a configuration, it's easy to run the same scenario on other browsers and platforms ( typically on a selenium grid like Saucelabs ).

For example, given the following Internet Explorer configuration:

```yaml
# sauce-msie.yaml

configurations:
  sauce_msie:
    platform: Windows 10
    browserName: internet explorer

    # selenium grid endpoint and creds:
    username:  <YOUR SAUCELABS USERNAME>                  # ⬅ replace with your valid 
    accessKey: <YOUR SAUCELABS ACCESS TOKEN>              # ⬅ saucelabs creds!
    usingServer: http://ondemand.saucelabs.com:80/wd/hub
```

.. if we run the scenario again using the `--platform` option:

```bash
# [ pwd: examples/feature-tour ]:
$ iota scenario 'Search for something using DuckDuckGo' --platform sauce_msie

INFO: performing the scenario: "Search for something using DuckDuckGo"
SCRIPT: Test case steps:
( start scenario steps )
- Search for something using DuckDuckGo
  - Navigate to the DuckDuckGo search page
    - Navigate in your browser to https://duckduckgo.com/
  - Type 'design debt economics article' into the Search field
  - Click the Search button
  - The Top search result should be displayed
  - The Top search result should contain the text, 'design debt economics' (ignoring case)
( end scenario steps )
SCRIPT: ✓ Navigate in your browser to https://duckduckgo.com/
INFO: https://saucelabs.com/beta/tests/43f8f42dd42f46d880cb07c91dd513a8/watch     # ⬅ shows live execution while scenario runs
SCRIPT: ✓ Type 'design debt economics article' into the Search field
SCRIPT: ✓ Click the Search button
SCRIPT: ✓ The Top search result should be displayed
SCRIPT: ✓ The Top search result should contain the text, 'design debt economics' (ignoring case)
INFO:


IOTA Scenario Execution Results:

| platform   | name                                  | dirs                                         | sauce                                                                   |
| ---------- | ------------------------------------- | -------------------------------------------- | ----------------------------------------------------------------------- |
| sauce_msie | Search for something using DuckDuckGo | /Users/jelm1/code/iota/examples/feature-tour | https://saucelabs.com/beta/tests/43f8f42dd42f46d880cb07c91dd513a8/watch |


IOTA Execution Statistics:

| tests | passes | pending | failures | duration |
| ----- | ------ | ------- | -------- | -------- |
| 5     | 5      | 0       | 0        | 22328    |
```

The scenario is executed in SauceLabs' cloud grid.  A link to the video replay is included in the output.

### Queues and concurrency

Thus far, we've run our scenarios one at a time, using the `iota scenario` command.  This works well when we're authoring our scripts and troubleshooting or debugging.

But as we author more scenarios and build our test coverage, we need a way to schedule and run them all, i.e. for regression testing.  While we could simply write shell scripts that sequence individual scenario commands, they quickly become hard to maintain, especially since we'd need a separate command for each scenario / platform combination.

IOTA offers a much more flexible and powerful way to handle this: *queues*.  Here's how it works:

#### configsets and suites

In `platforms.yaml`, I've provided seven (7) Saucelabs browser configurations - similar to the 'sauce_msie' example from before, but for the latest version of Chrome, Firefox, Safari, Internet Explorer, and Edge, on both Mac and Windows platforms.  For brevity I'm not showing them here, but feel free to have a look.

All seven configurations are included in a `configset` called `sauce_all`:

```yaml
# platforms.yaml (excerpt)
configsets:
  sauce_all:
    - chrome_sauce_windows
    - chrome_sauce_mac
    - firefox_sauce_windows
    - firefox_sauce_mac
    - safari_sauce
    - msie_sauce
    - edge_sauce

```

> To make our example more interesting, I've added an additional scenario to `scenarios.yaml`, which performs the same search as our first one, but using IxQuick ( another search engine ) instead.  Again, the scenario isn't shown here, but feel free to have a look in the yaml file.

configsets are used in conjunction with `suites`.  A suite in iota is what you'd expect - a collection of scenarios - but it also specifies which platforms will be used, via a configset like the `sauce_all` one we defined in the previous step.  

Here's a suite that includes both of our search scenarios, for all seven of our platforms:

```yaml
# scenarios.yaml (excerpt)
suites:
  search_engines_sauce_all:                      # ⬅ suite definition
    sauce_all:                                   # ⬅ sauce_all configset defined previously in platforms.yaml (previous step)
      - Search for something using DuckDuckGo
      - Search for something using IxQuick
```
.. and here's how iota uses it to *populate a queue*, when we run the `iota enqueue` command:

```bash
# [ pwd: examples/feature-tour ]:
$ iota enqueue search_engines_sauce_all
INFO: 14 documents inserted from suite search_engines_sauce_all into queue at /Users/jelm1/code/iota/examples/feature-tour/.queue
INFO: total documents: 14
```
> the queue is populated with the **product** of the enumerated scenarios and the configset's platforms.
<br>
<br>
> under the covers, iota is built to manage platforms as a **variability**.  the groundwork is there to populate queues with the products of other variabilities, like alternate flows, testdata variations, etc., making it possible to **generate coverage** for all permutations of defined variabilities.

At this point, we can use more `enqueue` commands to continue populating the queue with as many suites as we wish.

Let's proceed with our example, and tell iota to execute the scenarios we have in the queue now.
#### Running queued scenarios with concurrency

The command to run the queue is very simple: `iota queued`:

```bash
# [ pwd: examples/feature-tour ]:
$ iota queued --engines 20

INFO: scenarios 0% waiting: 14 running: 0 passed: 0 failed: 0 requeues: 0 engines target: 14 running: 0 time elapsed: 2 sec user: 0 sec
INFO: scenarios 0% waiting: 13 running: 1 passed: 0 failed: 0 requeues: 0 engines target: 14 running: 1 time elapsed: 4 sec user: 2 sec
INFO: scenarios 0% waiting: 12 running: 2 passed: 0 failed: 0 requeues: 0 engines target: 14 running: 2 time elapsed: 6 sec user: 6 sec
INFO: scenarios 0% waiting: 11 running: 3 passed: 0 failed: 0 requeues: 0 engines target: 14 running: 3 time elapsed: 8 sec user: 12 sec
# ..skipping about 20 lines..
INFO: scenarios 93% waiting: 0 running: 1 passed: 13 failed: 0 requeues: 0 engines target: 1 running: 1 time elapsed: 52 sec user: 4 min, 34 sec
INFO: scenarios 93% waiting: 0 running: 1 passed: 13 failed: 0 requeues: 0 engines target: 1 running: 1 time elapsed: 54 sec user: 4 min, 36 sec
INFO: scenarios 100% waiting: 0 running: 0 passed: 14 failed: 0 requeues: 0 engines target: 0 running: 0 time elapsed: 56 sec user: 4 min, 36 sec
INFO:

IOTA Queue Statistics:

| total | queued | running | success | failed | incomplete |
| ----- | ------ | ------- | ------- | ------ | ---------- |
| 14    | 0      | 0       | 14      | 0      | 0          |
```
New Node.js v8 engines are spawned as child processes.  The engines are ephemeral - each one draws one scenario from the queue, executes it, records the results, and then terminates.  

IOTA keeps spawning new engines, to the maximum concurrency specified by the `--engines` option, until the queue is complete.  Running stats are streamed to the console, showing queue progress, pass/fail numbers, and more.

### Handling 'flaky tests'

When running queues of a thousand or more scenarios, it becomes likely that some failures will result from flakiness of the network or other dependencies.  IOTA provides a few retry mechanisms for dealing with this.

#### Retry for individual scenario steps

You can configure IOTA to retry specific errors in the execution of any scenario step.  When a 'retryable' error occurs, IOTA retries the failed step a configurable number of times, with a configurable wait between retries.

Retryable errors are recognized by matching strings in the error's message and stacktrace.  Here's a retry config that allows resilience to common errors caused by animations in the DOM:

```yaml
# config.yaml

config:
  retryable errors:
    - StaleElement                  # ⬅ common errors that result
    - Element is not clickable      # ⬅ from animations in our 
    - InvalidElementStateError      # ⬅ web apps
```
Here's how it works in a flowchart:
![IOTA scenario step retry mechanism](iota-retry-mechanism.png?raw=true "Scenario Step Retry")

#### Requeueing scenarios

Similarly, you can configure IOTA to *requeue* an entire scenario when it fails because of known issues:

```yaml
# config.yaml

config:
  requeuable failures:
    - Navigate in your browser to                       # ⬅ probably means the page didn't even load
    - Cannot contact reCAPTCHA. Check your connection   # ⬅ occasional when hitting google via http proxy
```
Requeueing returns the scenario to the queue for subsequent re-execution.  When a scenario fails and is requeued, all logs and data from the attempt are retained for reporting and analysis.  A configurable requeue limit prevents the queue from endless attempts, when the issues persist.

#### Retry decorators and Recaptcha detection

IOTA's retry mechanism can be enhanced by providing an optional *retry decorator*; a scenario or Javascript function which is attempted within IOTA's retry loop, adding side-affect to the retry attempts:

- Any errors within the decorator's execution are consumed silently
- If the decorator returns a truthy result (with no errors), the original error is retried.  
- Otherwise, the pre-existing, undecorated retry logic is used.

This has been used to detect and handle Recaptcha in IOTA tests, unblocking automated testing where the appearance of Recaptcha is undeterministic because it is controlled by opaque and/or fluid external policies.  

If Recaptcha has been encountered (and not handled) by IOTA, it will undoubtedly cause a scenario step to fail.  By simply providing the existing recaptcha scenario as a retry decorator, Recaptcha can be detected and handled wherever it happens, effectively Recaptcha-proofing the entire suite.  ( Recaptcha must be in 'test' checkbox-only mode - no streetsigns and storefronts - for this to work. )

Here's the full retry mechanism, including decoration, as used for Recaptcha detection:

![Retry Decoration](retry-decorator.png?raw=true "Retry Decoration")

### More features and advanced usage

IOTA has many other features.  I'll provide separate documentation and/or tutorials in the wiki, but here's a list of features that are worth mentioning - feel free to ask me about them here .

- easy, flexible API testing and utility calls using 'request templates'
- Javascript DSL
- flexible organization and separation of test assets
- deep merging of test assets for overriding, easy environment tweaks, NLS support for testdata
- extensible, pluggable architecture
- performer decoration and multiple inheritance via mixins
- reporting in plain-text, html, markdown, with email/slack integration
- Code coverage support using Istanbul

## Feature Backlog and Issues

IOTA Bugs and Design Debt are tracked [here](https://github.intuit.com/CTO-Dev-FDS-FDX/qa-iota/issues).


