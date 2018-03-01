![Logo](https://cdn.rawgit.com/jaydenseric/graphql-react/b2e60e80/graphql-react-logo.svg)

# graphql-react

[![npm version](https://img.shields.io/npm/v/graphql-react.svg)](https://npm.im/graphql-react) ![Licence](https://img.shields.io/npm/l/graphql-react.svg) [![Github issues](https://img.shields.io/github/issues/jaydenseric/graphql-react.svg)](https://github.com/jaydenseric/graphql-react/issues) [![Github stars](https://img.shields.io/github/stars/jaydenseric/graphql-react.svg)](https://github.com/jaydenseric/graphql-react/stargazers) [![Travis status](https://img.shields.io/travis/jaydenseric/graphql-react.svg)](https://travis-ci.org/jaydenseric/graphql-react)

A lightweight GraphQL client for React.

> ⚠️ SSR API coming soon.

### Easy 🍺

* Add 1 dependency to get started with GraphQL in a React project.
* No Webpack or Babel setup.
* Simple components, no decorators.
* Query components fetch on mount and when props change. While loading, cache from the last identical request is available to display.
* Automatically fresh cache, even after mutations.
* Use file input values as mutation arguments to upload files; compatible with [a variety of servers](https://github.com/jaydenseric/graphql-multipart-request-spec#server).
* [Template literal](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Template_literals) queries; no need for [`gql`](https://github.com/apollographql/graphql-tag#gql).

### Smart 💡

* Adds ~ 4 KB to a typical min+gzip bundle.
* [Native ESM in Node.js](https://nodejs.org/api/esm.html) via `.mjs`.
* [Package `module` entry](https://github.com/rollup/rollup/wiki/pkg.module) for [tree shaking](https://developer.mozilla.org/docs/Glossary/Tree_shaking) bundlers.
* Components use the [React v16.3 context API](https://github.com/facebook/react/pull/11818).
* All fetch options overridable per request.
* Request hash based cache:
  * No data denormalization and no need to query `id` fields.
  * No tampering with queries or `__typename` insertion.
  * Query multiple GraphQL services without stitching data.
  * Errors also cache, enabling error SSR.

## Setup

Install with [npm](https://npmjs.com):

```shell
npm install graphql-react
```

Create and provide a `GraphQL` client:

```jsx
import { GraphQL, Provider } from 'graphql-react'

const graphql = new GraphQL()

const Page = () => (
  <Provider value={graphql}>Use Consumer or Query components…</Provider>
)
```

## Example

See the [example Next.js app and GraphQL API](example/readme.md).

## Support

* Node.js v7.6+.
* Browsers [>1% usage](http://browserl.ist/?q=%3E1%25).

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

* [Provider](#provider)
* [Consumer](#consumer)
* [Query](#query)
* [QueryRender](#queryrender)
* [ConsumerRender](#consumerrender)
* [HTTPError](#httperror)
* [Operation](#operation)
* [RequestOptions](#requestoptions)
* [RequestOptionsOverride](#requestoptionsoverride)
* [GraphQL](#graphql)
  * [cache](#cache)
  * [reset](#reset)
  * [query](#query-1)
* [RequestCache](#requestcache)
* [CacheUpdateCallback](#cacheupdatecallback)
* [RequestCachePromise](#requestcachepromise)
* [ActiveQuery](#activequery)

### Provider

[src/components.mjs:23-23](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/components.mjs#L23-L23 'Source code on GitHub')

A React component that puts a [GraphQL](#graphql) instance in context for nested [Consumer](#consumer) components to use.

**Parameters**

* `value` **[GraphQL](#graphql)** A [GraphQL](#graphql) instance.
* `children` **ReactNode** A React node.

**Examples**

```javascript
import { GraphQL, Provider } from 'graphql-react'

const graphql = new GraphQL()

const Page = () => (
  <Provider value={graphql}>Use Consumer or Query components…</Provider>
)
```

Returns **ReactElement** React virtual DOM element.

### Consumer

[src/components.mjs:39-39](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/components.mjs#L39-L39 'Source code on GitHub')

A React component that gets the [GraphQL](#graphql) instance from context.

**Parameters**

* `children` **[ConsumerRender](#consumerrender)** Render function that receives a [GraphQL](#graphql) instance.

**Examples**

_A button component that resets the [GraphQL](#graphql) cache._

```javascript
import { Consumer } from 'graphql-react'

const ResetCacheButton = () => (
  <Consumer>
    {graphql => <button onClick={graphql.reset}>Reset cache</button>}
  </Consumer>
)
```

Returns **ReactElement** React virtual DOM element.

### Query

[src/components.mjs:241-245](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/components.mjs#L241-L245 'Source code on GitHub')

A React component to manage a GraphQL query or mutation.

**Parameters**

* `props` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Component props.
  * `props.variables` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL query variables.
  * `props.query` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** GraphQL query.
  * `props.loadOnMount` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the query load when the component mounts. (optional, default `false`)
  * `props.loadOnReset` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the query load when the [GraphQL](#graphql) cache is reset. (optional, default `false`)
  * `props.resetOnLoad` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the [GraphQL](#graphql) cache reset when the query loads. (optional, default `false`)
* `children` **[QueryRender](#queryrender)** Renders the query status.

**Examples**

_A query to display a user profile._

```javascript
import { Query } from 'graphql-react'

const Profile = ({ userId }) => (
  <Query
    loadOnMount
    loadOnReset
    variables={{ userId }}
    query={`
      query user($userId: ID!) {
        user(userId: $id) {
          name
        }
      }
    `}
  >
    {({ load, loading, httpError, parseError, graphQLErrors, data }) => (
      <article>
        <button onClick={load}>Reload</button>
        {loading && <span>Loading…</span>}
        {(httpError || parseError || graphQLErrors) && <strong>Error!</strong>}
        {data && <h1>{data.user.name}</h1>}
      </article>
    )}
  </Query>
)
```

_A mutation to clap an article._

```javascript
import { Query } from 'graphql-react'

const ClapArticleButton = ({ articleId }) => (
  <Query
    resetOnLoad
    variables={{ articleId }}
    query={`
      mutation clapArticle($articleId: ID!) {
        clapArticle(articleId: $id) {
          clapCount
        }
      }
    `}
  >
    {({ load, loading, httpError, parseError, graphQLErrors, data }) => (
      <aside>
        <button onClick={load} disabled={loading}>
          Clap
        </button>
        {(httpError || parseError || graphQLErrors) && <strong>Error!</strong>}
        {data && <p>Clapped {data.clapArticle.clapCount} times.</p>}
      </aside>
    )}
  </Query>
)
```

Returns **ReactElement** React virtual DOM element.

### QueryRender

[src/components.mjs:247-254](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/components.mjs#L247-L254 'Source code on GitHub')

Renders the status of a query or mutation.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `load` **[Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)** Loads the query on demand, updating cache.
* `loading` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Is the query loading.
* `httpError` **[HTTPError](#httperror)?** Fetch HTTP error.
* `parseError` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** Parse error message.
* `graphQLErrors` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response errors.
* `data` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response data.

**Examples**

```javascript
;({ load, loading, httpError, parseError, graphQLErrors, data }) => (
  <aside>
    <button onClick={load}>Reload</button>
    {loading && <span>Loading…</span>}
    {(httpError || parseError || graphQLErrors) && <strong>Error!</strong>}
    {data && <h1>{data.user.name}</h1>}
  </aside>
)
```

Returns **ReactElement** React virtual DOM element.

### ConsumerRender

[src/components.mjs:247-254](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/components.mjs#L247-L254 'Source code on GitHub')

Renders a [GraphQL](#graphql) consumer.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `graphql` **[GraphQL](#graphql)** [GraphQL](#graphql) instance.

**Examples**

_A button that resets the [GraphQL](#graphql) cache._

```javascript
graphql => <button onClick={graphql.reset}>Reset cache</button>
```

Returns **ReactElement** React virtual DOM element.

### HTTPError

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

Fetch HTTP error.

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `status` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** HTTP status code.
* `statusText` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** HTTP status text.

### Operation

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

A GraphQL operation object. Additional properties may be used; all are sent to the GraphQL server.

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `query` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** GraphQL queries or mutations.
* `variables` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Variables used by the query.

### RequestOptions

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

Options for a GraphQL fetch request. See [polyfillable fetch options](https://github.github.io/fetch/#options).

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `url` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** A GraphQL API URL.
* `body` **([String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [FormData](https://developer.mozilla.org/docs/Web/API/FormData))** HTTP request body.
* `headers` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** HTTP request headers.
* `credentials` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** Authentication credentials mode.

### RequestOptionsOverride

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

A way to override request options generated for a fetch. Modify the provided options object directly; no return.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `requestOptions` **[RequestOptions](#requestoptions)**

**Examples**

```javascript
options => {
  options.url = 'https://api.example.com/graphql'
  options.credentials = 'include'
}
```

### GraphQL

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

A lightweight GraphQL client that caches requests.

**Parameters**

* `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Options. (optional, default `{}`)
  * `options.cache` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Cache to import; useful once a SSR API is available. (optional, default `{}`)
  * `options.requestOptions` **[RequestOptionsOverride](#requestoptionsoverride)?** A function that accepts and modifies generated options for every request.

**Examples**

```javascript
import { GraphQL } from 'graphql-react'

const graphql = new GraphQL({
  requestOptions: options => {
    options.url = 'https://api.example.com/graphql'
    options.credentials = 'include'
  }
})
```

#### cache

[src/graphql.mjs:25-25](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L25-L25 'Source code on GitHub')

GraphQL request cache.

#### reset

[src/graphql.mjs:75-79](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L75-L79 'Source code on GitHub')

Resets the cache. Useful when a user logs out.

**Examples**

```javascript
graphql.reset()
```

#### query

[src/graphql.mjs:194-206](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L194-L206 'Source code on GitHub')

Queries a GraphQL server.

**Parameters**

* `operation` **[Operation](#operation)** GraphQL operation object.

Returns **[ActiveQuery](#activequery)** In-flight query details.

### RequestCache

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

JSON serializable result of a request (including all errors and data) for caching purposes.

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `httpError` **[HTTPError](#httperror)?** Fetch HTTP error.
* `parseError` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** Parse error message.
* `graphQLErrors` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response errors.
* `data` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response data.

### CacheUpdateCallback

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

A cache update listener callback.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `requestCache` **[RequestCache](#requestcache)** Request cache.

### RequestCachePromise

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

A promise for an in-flight query that resolves the request cache.

Type: [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)&lt;[RequestCache](#requestcache)>

### ActiveQuery

[src/graphql.mjs:19-207](https://github.com/jaydenseric/graphql-react/blob/79fe2b0ccbc29aa48d5bc47415c853fa282384c0/src/graphql.mjs#L19-L207 'Source code on GitHub')

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `pastRequestCache` **[RequestCache](#requestcache)?** Results from the last identical request.
* `requestHash` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** Request options hash.
* `request` **[RequestCachePromise](#requestcachepromise)** Promise that resolves fresh request cache.
