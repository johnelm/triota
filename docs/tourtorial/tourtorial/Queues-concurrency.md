# IOTA Feature Tourtorial
( back to [Tourtorial home](index.md) )<hr>
## Queues and concurrency

Thus far, we've run our scenarios one at a time, using the `iota scenario` command.  This works well when we're authoring our scripts and troubleshooting or debugging.

But as we author more scenarios and build our test coverage, we need a way to schedule and run them all, i.e. for regression testing.  While we could simply write shell scripts that sequence individual scenario commands, they quickly become hard to maintain, especially since we'd need a separate command for each scenario / platform combination.

IOTA offers a much more flexible and powerful way to handle this: *queues*.  Here's how it works:

### configsets and suites

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

We can use more `enqueue` commands to continue populating the queue with as many suites as we wish.

But let's say we're done populating the queue, and proceed with our example.
### Running queued scenarios with concurrency

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

**Next: [Handling flaky tests](Flaky.md)**
