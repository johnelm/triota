### Easy, flexible API testing

IOTA allows you to define API requests in YAML as templates

For making utility calls, assertions, or API-only testing

Serialized requests easily imported from Postman


```yaml

login:
  method: POST
  url: https://accounts.yourco.com/access_client/sign_in
  headers:
    cache-control: no-cache
    offeringid: ${offerings.offeringId}
    content-type: application/json
  body:
    username: ${userID}
    password: ${password}
  json: true
```



- request templates

if no values given, will fall back to values provided in testdata


```
request templates
	again, can use test data here
	API testing
		serialization of requests
		injection of variables / values
		used for API testing
		and performing actions via API within flows
			assertions on API
			creation of new users
```

```
also javascript
	Javascript DSL
		compiled on the file using Babel
		using IOTA’s babel configuration, saving the author the trouble
		ScenarioFunctionActions should be able to extend any Type defined
			should provide hooks for
				before
				after
	cookie injection
```



As of [v1.7.0](../releases/tag/1.7.0), IOTA supports API Testing, and the ability to call remote services amid IOTA scenarios.

- uses 'request'
- serialization format
- Postman export
- parameterizing requests
- using requests