# IOTA Feature Tourtorial
( back to [Tourtorial home](index.md) )<hr>
## Fast, easy script authoring
IOTA uses an easy-to-learn YAML-based DSL, so you can get started quickly.  JavaScript is also supported for utility and API calls.

Here's a taste:

```yaml
# scenarios.yaml

testdata:                                                       # ⬅ test data definition

  url:                                                          # ⬅ object, referenced using dot notation
    duckduckgo: https://duckduckgo.com/
  searchPhrase: design debt economics                           # ⬅ simple variable / value
    
pages:                                                          # ⬅ page object definitions

  DuckDuckGo search page:                                       # ⬅ page
    elements:
      Search field:                                             # ⬅ element
        locator:
          id: search_form_input_homepage                        # ⬅ locator (i.e. id, xpath etc)
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

**Next: [Running scenarios](Running.md)**
