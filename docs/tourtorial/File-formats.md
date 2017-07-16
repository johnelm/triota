## File Formats

IOTA test assets are authored in text files of these types: yaml, json, and Javascript (*.js).  In all cases, the test assets are made available to the IOTA engine by _exporting_ them in the way appropriate for the file type.

### YAML

YAML has been the most common format used for IOTA.  The YAML document's 'top level' objects are exported and consumed by the IOTA engine, which loads the assets into memory.

Example - exporting a scenario using yaml:
```yaml
scenarios:
  My awesome scenario:
    actions:
      - do something awesome and important
```

### JSON

Test assets can be authored in JSON format if desired - it works exactly the same as with YAML (top level objects are exported).  It hasn't been used much simply because YAML is more readable, but some assets may be well suited for the json format (i.e. testdata and webdriver capabilities).

Example - exporting testdata using json:
```json
{
  "testdata": {
    "user": "john_elm@intuit.com",
    "title": "two hecks of a guy",
    "org": "Dept. of Redundancy Department"
  }
}
```

### Javascript (*.js)

Since IOTA [v1.7.0](https://github.intuit.com/CTO-Dev-FDS-FDX/qa-iota/releases/tag/1.7.0), we can provide a .js file that is consumed like a node.js module file, and which exports test assets [like any other node.js module](https://nodejs.org/docs/latest/api/modules.html#modules_module_exports).

You can export any kind of IOTA test assets (i.e. testdata, scenarios, etc.) from a .js file.  If Javascript functions are exported as members of a `scenarios` export, those functions can be used in (or as) executable scenarios for your flows in IOTA.  The function will have the same access to the IOTA API as IOTA's internal Performables.

Example - exporting a `scenario function` from a Javascript module file:
```js
module.exports = {
  scenarios: {
    'My awesome scenario': function() {
      console.log('so awesome');
      // now do something useful in Javascript
    }
  }
}
```

## Loading test assets into IOTA

Test assets can be consumed as individual files, or, if a directory is specified, all test asset files in the directory will be loaded.  Multiple test asset sources can be specified when running IOTA - see (link TBD:running IOTA) for more details.

## General asset loading rules

* You may export anything you like from IOTA test asset files, but IOTA will ignore any objects that it doesn't recognize.
* The recognized entities are explained here (link TBD to the schema page).
* Within a single file, only one object of each type is allowed (export names must be unique).
* All test assets are _merged_ together when they are loaded by IOTA.  This means:
  * multiple files (potentially of multiple formats) may export the same asset types: 
    * file1.yaml, file2.yaml and file3.json may each export its own 'testdata' object.
    * in this case, the testdata graph available at runtime will be the result of merging all three exports together
  * the testdata values themselves will also be merged

Example:

`mytestassets/file1.yaml`:
```yaml
testdata:
  userid: john_elm@intuit.com
```
`mytestassets/file2.yaml`:
```yaml
testdata:
  app:
    url: https://myoffering-qal.intuit.net
```

At runtime, the available testdata will be:
```yaml
testdata:
  userid: john_elm@intuit.com
  app:
    url: https://myoffering-qal.intuit.net
```

## Overriding and inheritance
Since multiple test asset sources can be specified when running IOTA, we can author files in a way that affords some overriding capability: values that are loaded _last_ will override previously loaded values.

Example: say that in addition to the `mytestassets` folder in the examples above, I also have a `e2etestassets` folder.  I want to test my system's e2e endpoint without changing my scripts.

`e2etestassets/e2e.yaml`:
```yaml
testdata:
  app:
    url: https://myoffering-e2e.intuit.net
```

.. and say I run IOTA to load the test assets from both folders:

`iota <some iota command> -f ./mytestassets -f ./e2etestassets`

The resulting testdata at runtime will be:

```yaml
testdata:
  userid: john_elm@intuit.com
  app:
    url: https://myoffering-e2e.intuit.net
```






