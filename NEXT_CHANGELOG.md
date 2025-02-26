# Changelog for the next release

All notable changes to Router will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!-- <KEEP> THIS IS AN SET OF TEMPLATES TO USE WHEN ADDING TO THE CHANGELOG.

## ❗ BREAKING ❗
## 🚀 Features
## 🐛 Fixes
## 📃 Configuration
Configuration changes will be [automatically migrated on load](https://www.apollographql.com/docs/router/configuration/overview#upgrading-your-router-configuration). However, you should update your source configuration files as these will become breaking changes in a future major release.
## 🛠 Maintenance
## 📚 Documentation
## 🥼 Experimental

## Example section entry format

### Headline ([Issue #ISSUE_NUMBER](https://github.com/apollographql/router/issues/ISSUE_NUMBER))

Description! And a link to a [reference](http://url)

By [@USERNAME](https://github.com/USERNAME) in https://github.com/apollographql/router/pull/PULL_NUMBER
</KEEP> -->

## 🚀 Features

### Always deduplicate variables ([Issue #2387](https://github.com/apollographql/router/issues/2387))

Variable deduplication allows the router to reduce the number of entities that are requested from subgraphs if some of them are redundant, and as such reduce the size of subgraph responses. It has been available for a while but was not active by default. This is now always on.

By [@Geal](https://github.com/geal) in https://github.com/apollographql/router/pull/2445

### Add optional `Access-Control-Max-Age` header to CORS plugin ([Issue #2212](https://github.com/apollographql/router/issues/2212))

Adds new option called `max_age` that is used like this:
```
cors:
  max_age: 86400secs
```

By [@osamra-rbi](https://github.com/osamra-rbi) in https://github.com/apollographql/router/pull/2331

## 🐛 Fixes

### Listen on root URL when `/*` is set in `supergraph.path` configuration ([Issue #2471](https://github.com/apollographql/router/issues/2471))

If you provided this configuration:

```yaml
supergraph:
  path: /*
```

Since release `1.8` and due to [Axum upgrade](https://github.com/tokio-rs/axum/releases/tag/axum-v0.6.0) it wasn't listening on `localhost` without a path. It now has a special case for `/*` to also listen to the URL without a path so you're able to call on `http://localhost` for example.

By [@bnjjj](https://github.com/bnjjj) in https://github.com/apollographql/router/pull/2472

### Better support for wildcard in `supergraph.path` configuration ([Issue #2406](https://github.com/apollographql/router/issues/2406))

You can now use wildcard in supergraph endpoint path like this:

```yaml
supergraph:
  listen: 0.0.0.0:4000
  path: /g*
```

if you want the supergraph to answer on `/graphql` or `/gateway` for example.

By [@bnjjj](https://github.com/bnjjj) in https://github.com/apollographql/router/pull/2410

### Fix panic in schema parse error reporting ([Issue #2269](https://github.com/apollographql/router/issues/2269))

In order to support introspection,
some definitions like `type __Field { … }` are implicitly added to schemas.
This addition was done by string concatenation at the source level.
In some cases like unclosed braces, a parse error could be reported at a position
beyond the size of the original source.
This would cause a panic because only the unconcatenated string
is given the the error reporting library `miette`.

Instead, the Router now parses introspection types separately
and “concatenates” definitions at the AST level.

By [@SimonSapin](https://github.com/SimonSapin) in https://github.com/apollographql/router/pull/2448

### Always accept compressed subgraph responses  ([Issue #2415](https://github.com/apollographql/router/issues/2415))

Subgraph response decompression was only supported when subgraph request compression was configured. This is now always active.

By [@Geal](https://github.com/geal) in https://github.com/apollographql/router/pull/2450

### Fix handling of root query operation not named `Query`

With such a schema, some parsing code in the Router would incorrectly
return an error because it was assuming the default name.
Similarly with a root mutation operation not named `Mutation`.

By [@SimonSapin](https://github.com/SimonSapin) in https://github.com/apollographql/router/pull/2459

### Remove the `locations` field from subgraph errors ([Issue #2297](https://github.com/apollographql/router/issues/2297))

Subgraph errors can come with a `locations` field indicating which part of the query was causing issues, but it refers to the subgraph query generated by the query planner, and we have no way of translating it to locations in the client query, so this field is removed for now.

By [@Geal](https://github.com/geal) in https://github.com/apollographql/router/pull/2442

### Emit metrics showing number of client connections ([issue #2384](https://github.com/apollographql/router/issues/2384))

New metrics are available to track the client connections:
- `apollo_router_session_count_total` indicates the number of currently connected clients
- `apollo_router_session_count_active` indicates the number of in flight GraphQL requests from connected clients.

This also fixes the behaviour when we reach the maximum number of file descriptors: instead of going into a busy loop, the router will wait a bit before accepting a new connection.

By [@Geal](https://github.com/geal) in https://github.com/apollographql/router/pull/2395

## 🛠 Maintenance

### Improve #[serde(default)] attribute on structs ([Issue #2424](https://github.com/apollographql/router/issues/2424))

If all the fields of your struct have their default value then use the `#[serde(default)]` on the struct instead of all fields. If you have specific default values for field, you have to create your own `Default` impl.

+ GOOD
```rust
#[serde(deny_unknown_fields, default)]
struct Export {
    url: Url,
    enabled: bool
}

impl Default for Export {
  fn default() -> Self {
    Self {
      url: default_url_fn(),
      enabled: false
    }
  }
}
```

+ BAD
```rust
#[serde(deny_unknown_fields)]
struct Export {
    #[serde(default="default_url_fn")
    url: Url,
    #[serde(default)]
    enabled: bool
}
```

By [@bnjjj](https://github.com/bnjjj) in https://github.com/apollographql/router/pull/2424

## 📃 Configuration

Configuration changes will be [automatically migrated on load](https://www.apollographql.com/docs/router/configuration/overview#upgrading-your-router-configuration). However, you should update your source configuration files as these will become breaking changes in a future major release.

### `health-check` renamed to `health_check` ([Issue #2161](https://github.com/apollographql/router/issues/2161))

The health_check key in router.yaml has been renamed to snake case for consistency. 

Before:
```yaml
health-check:
  enabled: true
```

After:
```yaml
health_check:
  enabled: true
```

By [@bryncooke](https://github.com/bryncooke) in https://github.com/apollographql/router/pull/2451 and https://github.com/apollographql/router/pull/2463

### Return a proper timeout response ([Issue #2360](https://github.com/apollographql/router/issues/2360) [Issue #2400](https://github.com/apollographql/router/issues/240))

There was a regression where timeouts generated a HTTP response with status `500 Internal Server Error`. This is now fixed with a test to guarantee it, the status code is now `504 Gateway Timeout` (Instead of previously `408 Request Timeout` which blamed the client). There is also a new metric `apollo_router_timeout` to track when timeouts are triggered.

By [@Geal](https://github.com/geal) in https://github.com/apollographql/router/pull/2419

## 📚 Documentation

### Fix the documentation to disable the Apollo telemetry ([Issue #2478](https://github.com/apollographql/router/issues/2478))

To disable the Apollo telemetry you have to use `APOLLO_TELEMETRY_DISABLED=true`.

By [@bnjjj](https://github.com/bnjjj) in https://github.com/apollographql/router/pull/2479

### `send_headers` and `send_variable_values` in `telemetry.apollo` ([Issue #2149](https://github.com/apollographql/router/issues/2149))

+ `send_headers`

  Provide this field to configure which request header names and values are included in trace data that's sent to Apollo Studio. Valid options are: `only` with an array, `except` with an array, `none`, `all`.

  The default value is `none``, which means no header names or values are sent to Studio. This is a security measure to prevent sensitive data from potentially reaching the Router.

+ `send_variable_values`

  Provide this field to configure which variable values are included in trace data that's sent to Apollo Studio. Valid options are: `only` with an array, `except` with an array, `none`, `all`.

  The default value is `none`, which means no variable values are sent to Studio. This is a security measure to prevent sensitive data from potentially reaching the Router.


By [@bnjjj](https://github.com/bnjjj) in https://github.com/apollographql/router/pull/2435

### Documentation on how to propagate headers between subgraph services ([Issue #2128](https://github.com/apollographql/router/issues/2128))

Migrating headers between subgraph services is possible via Rhai script. An example has been added to the header propagation page.

By [@bryncooke](https://github.com/bryncooke) in https://github.com/apollographql/router/pull/2446

### Added documentation for listening on IPv6 ([Issue #1835](https://github.com/apollographql/router/issues/1835))

Added documentation for listening on IPv6
```yaml
supergraph:
  # The socket address and port to listen on. 
  # Note that this must be quoted to avoid interpretation as a yaml array.
  listen: '[::1]:4000'
```

By [@bryncooke](https://github.com/bryncooke) in https://github.com/apollographql/router/pull/2440

## 🛠 Maintenance

### Parse schemas and queries with `apollo-compiler`

The Router now uses the higher-level representation from `apollo-compiler`
instead of using the AST from `apollo-parser` directly.
This is a first step towards replacing a bunch of code that grew organically
during the Router’s early days, with a general-purpose library with intentional design.
Internal data structures are unchanged for now.
Parsing behavior has been tested to be identical on a large corpus
of production schemas and queries.

By [@SimonSapin](https://github.com/SimonSapin) in https://github.com/apollographql/router/pull/2466