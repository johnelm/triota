# IOTA Feature Tourtorial
( back to [Tourtorial home](index.md) )<hr>
## Handling 'flaky tests'

When running queues of a thousand or more scenarios, it's likely that some failures will result from flakiness of the network or other dependencies.  IOTA provides a few retry mechanisms for dealing with this.

### Retry for individual scenario steps

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

### Requeueing scenarios

Similarly, you can configure IOTA to *requeue* an entire scenario when it fails because of known issues:

```yaml
# config.yaml

config:
  requeuable failures:
    - Navigate in your browser to                       # ⬅ probably means the page didn't even load
    - Cannot contact reCAPTCHA. Check your connection   # ⬅ occasional when hitting google via http proxy
```
Requeueing returns the scenario to the queue for subsequent re-execution.  When a scenario fails and is requeued, all logs and data from the attempt are retained for reporting and analysis.  A configurable requeue limit prevents the queue from endless attempts, when the issues persist.

### Retry decorators and Recaptcha detection

IOTA's retry mechanism can be enhanced by providing an optional *retry decorator*; a scenario or Javascript function which is attempted within IOTA's retry loop, adding side-affect to the retry attempts:

- Any errors within the decorator's execution are consumed silently
- If the decorator returns a truthy result (with no errors), the original error is retried.  
- Otherwise, the pre-existing, undecorated retry logic is used.

This has been used to detect and handle Recaptcha in IOTA tests, unblocking automated testing where the appearance of Recaptcha is undeterministic because it is controlled by opaque and/or fluid external policies.  

If Recaptcha has been encountered (and not handled) by IOTA, it will undoubtedly cause a scenario step to fail.  By simply providing the existing recaptcha scenario as a retry decorator, Recaptcha can be detected and handled wherever it happens, effectively Recaptcha-proofing the entire suite.  ( Recaptcha must be in 'test' checkbox-only mode - no streetsigns and storefronts - for this to work. )

Here's the full retry mechanism, including decoration, as used for Recaptcha detection:

![Retry Decoration](retry-decorator.png?raw=true "Retry Decoration")
<hr>
This is the end of the tourtorial for now.  

**Back to [Tourtorial home](index.md)**

