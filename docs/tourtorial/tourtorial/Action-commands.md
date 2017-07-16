Here's the list of commands currently supported in IOTA 'action' and 'results' sequences (within scenarios):

While it's most typical to use *action* commands in the action block of a scenario and *assertions* in the results section, there are no rules against using assertions in action blocks.

###Actions:
- `webdriver.get(url)` (not element based)
- `sleep(millis)` (not element based)
- `waitForEnabled()`
- `waitForDisplay()`
- `click()`
- `sendKeys()`
- `uploadFile()`
- `clear()`
- `check()`
- `uncheck()`

###Assertions:
- `isDisplayed()`
- `isVisible()`
- `isEnabled()`
- `isNotEnabled()`
- `isNotDisplayed()`
- `count().equals(numArg)`
- `getText().contains(arg)`
- `getAttribute(attrname, attrvalue)`

> Note: `isNotEnabled()` and `isNotDisplayed()` are synonyms for `!isEnabled()` and `!isDisplayed()`.  They were added for convenience - when composing scenarios in YAML, quotes around the action expression are usually not required, unless they start with a bang.  The synonyms can be used to avoid the confusion around when quotes are required.


scenarios:
  Test the new assertions:
    actions:
      - webdriver.get('https://www.google.com')
      - Google Search Page:
        - Text field:
          - sendKeys('booger nuggets')
        - Search button:
          - '!isDisplayed()'
        - Text field:
          - sendSpecialKeys('Enter')
        - Result stats:
          - jsonContainsPath('this.that.other')
          - valueAtJsonPathEquals('this.that.other', 'looking for this text!')
          - isDisplayed()
          - '!getText().contains("booger")'
          - sleep(2000)
          - getText().contains('results')
          - '!getText().equals("results")'
