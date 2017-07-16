# IOTA Feature Tourtorial

Let's have a closer look at IOTA's usage and features.

You can try most of the steps in the tourtorial yourself by cloning this repository locally, and navigating to the `examples/feature-tour` directory.  Note that for some parts, valid Saucelabs credentials are needed.

Basic knowledge of selenium and page objects is helpful, but not required.

## Table of Contents

Here's what we cover in the tourtorial so far:

- [Fast, easy script authoring](Authoring.md)
- [Running scenarios](Running.md)
  - [dry run: manual test instructions, for free](Running.md#dry-run-manual-test-instructions-for-free)
  - [in Chrome](Running.md#in-chrome)
  - [on other platforms, browsers and Selenium grids](Running.md#on-other-platforms-browsers-and-selenium-grids)
- [Queues and concurrency](Queues-concurrency.md#queues-and-concurrency)
  - [configsets and suites](Queues-concurrency.md#configsets-and-suites)
  - [Running queued scenarios with concurrency](Queues-concurrency.md#running-queued-scenarios-with-concurrency)
- [Handling 'flaky tests'](Flaky.md#handling-flaky-tests)
  - [Retry for individual scenario steps](Flaky.md#retry-for-individual-scenario-steps)
  - [Requeueing scenarios](Flaky.md#requeueing-scenarios)
  - [Retry decorators and Recaptcha detection](Flaky.md#retry-decorators-and-recaptcha-detection)

## Tourtorials in progress / coming soon
The following features are complete, but not yet covered in the tourtorial.  I'm working on it.  ¯\\(ツ)/¯

- Reporting
  - Supported formats
  - Code coverage with Istanbul
- Scripting in Javascript
  - Javascript scripting support
  - JSON Test assets
- API Testing
  - API request templates
  - Using request templates in Javascript
- Organization and selection / scoping of test assets
  - The -f option
  - Deep merge of test assets and overriding
  - Environment tweaks, NLS support
  - Label support for repeatable CLI arguments
- Architecture
  - Performer decoration and multiple inheritance via mixins
  - Extending IOTA with plugins
  - The road ahead

Back to the main IOTA [README](../../README.md)
