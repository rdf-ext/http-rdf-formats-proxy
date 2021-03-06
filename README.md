# http rdf formats proxy

[Express](http://expressjs.com/) middleware for fetching and converting resources between RDF formats

## install

```bash
npm install http-rdf-formats-proxy
```

In [Express](http://expressjs.com/) on `GET /proxy`:
```js
const express = require('express')
const formatsProxy = require('http-rdf-formats-proxy')

const app = express()
app.get('/proxy', formatsProxy({
  // options with their default values:
  fetch: require('rdf-fetch-lite')
  // Fetch used for the proxy request
  // Must implement response.quadStream() returning
  //   data following RDFJS spec interfaces
  formats: require('rdf-formats-common')()
  // These formats are used for serialization
  // Formats could be taken also from provided fetch
}))
app.server = app.listen(8000)
```

## usage

Currently [rdf-formats-common](https://github.com/rdf-ext/rdf-formats-common/blob/51e26d7e83d8946a4f7078dbbd2992906cb07899/index.js#L18) formats are supported by default.
```bash
curl -H 'Accept: text/n3' 'http://localhost:8000/proxy?uri=http://example.com/resource.jsonld'
curl -H 'Accept: text/turtle' 'http://localhost:8000/proxy?uri=http://example.com/resource.n3'
curl -H 'Accept: application/ld+json' 'http://localhost:8000/proxy?uri=http://example.com/resource.n3'
curl -H 'Accept: application/n-triples' 'http://localhost:8000/proxy?uri=http://example.com/resource.n3'
```

Proxy works both directions. You can POST data, convert them through proxy and it POSTs  further.
Response is then converted to Accepted format. To specify format for proxy request, use `produce` query parameter.
```bash
curl --data "<http://example.com/subject> <http://example.com/predicate> <http://example.com/object> ." \
  -H 'Content-type: n-triples' \
  -H 'Accept: text/n3' \
  'http://localhost:8000/proxy?uri=http://example.com/consumer/&produce=application/ld+json'
```

Proxy can act as a converter. To use converter mode don't use any query parameter and just POST your data. Proxy will return the data converted according to the request's Accept header.
```bash
curl --data "<http://example.com/subject> <http://example.com/predicate> <http://example.com/object> ." \
  -H 'Content-type: n-triples' \
  -H 'Accept: application/ld+json' \
  'http://localhost:8000/proxy'
```

## tested cases
```mocha
    ✓ no uri specified -> 400 Bad Request
    ✓ convert json+ld to n3
    ✓ convert rdf+xml to n3
    ✓ convert to unsupported/type should pass data through(?)
    ✓ pass non-rdf page data through(?)
    ✓ fetching a not existing page should pass 404 through
    ✓ fetching from a not existing server should yield 502(?)
    ✓ forward convert json+ld to n3 and response convert n-triples to n3
    ✓ convert mode - json+ld to n-triples
```

