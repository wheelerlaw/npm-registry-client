# npm-registry-client

The code that npm uses to talk to the registry.

It handles all the caching and HTTP calls.

## Usage

```javascript
var RegClient = require('npm-registry-client')
var client = new RegClient(config)
var uri = "npm://registry.npmjs.org/npm"
var params = {timeout: 1000}

client.get(uri, params, function (error, data, raw, res) {
  // error is an error if there was a problem.
  // data is the parsed data object
  // raw is the json string
  // res is the response from couch
})
```

# Registry URLs

The registry calls take either a full URL pointing to a resource in the
registry, or a base URL for the registry as a whole (for the base URL, any path
will be ignored). In addition to `http` and `https`, `npm` URLs are allowed.
`npm` URLs are `https` URLs with the additional restrictions that they will
always include authorization credentials, and the response is always registry
metadata (and not tarballs or other attachments).

# Configuration

This program is designed to work with
[npmconf](https://npmjs.org/package/npmconf), but you can also pass in
a plain-jane object with the appropriate configs, and it'll shim it
for you.  Any configuration thingie that has get/set/del methods will
also be accepted.

* `tag` {String} The default tag to use when publishing new packages.
  Default = `"latest"`
* `ca` {String} Cerficate signing authority certificates to trust.
* `cert` {String} Client certificate (PEM encoded). Enable access
  to servers that require client certificates
* `key` {String} Private key (PEM encoded) for client certificate 'cert'
* `strict-ssl` {Boolean} Whether or not to be strict with SSL
  certificates.  Default = `true`
* `user-agent` {String} User agent header to send.  Default =
  `"node/{process.version} {process.platform} {process.arch}"`
* `log` {Object} The logger to use.  Defaults to `require("npmlog")` if
  that works, otherwise logs are disabled.
* `fetch-retries` {Number} Number of times to retry on GET failures.
  Default=2
* `fetch-retry-factor` {Number} `factor` setting for `node-retry`. Default=10
* `fetch-retry-mintimeout` {Number} `minTimeout` setting for `node-retry`.
  Default=10000 (10 seconds)
* `fetch-retry-maxtimeout` {Number} `maxTimeout` setting for `node-retry`.
  Default=60000 (60 seconds)
* `proxy` {URL} The url to proxy requests through.
* `https-proxy` {URL} The url to proxy https requests through.
  Defaults to be the same as `proxy` if unset.
* `_token` {Object} A token for use with
  [couch-login](https://npmjs.org/package/couch-login)

## Using the client

Every call to the client follows the same pattern:

* `uri` {String} The *fully-qualified* URI of the registry API method being
  invoked.
* `params` {Object} Per-request parameters.
* `callback` {Function} Callback to be invoked when the call is complete.

### Credentials

Many requests to the registry can by authenticated, and require credentials
for authorization. These credentials always look the same:

* `username` {String}
* `password` {String}
* `email` {String}
* `alwaysAuth` {Boolean} Whether calls to the target registry are always
  authed.

**or**

* `token` {String}
* `alwaysAuth` {Boolean} Whether calls to the target registry are always
  authed.

## API

### client.adduser(uri, params, cb)

* `uri` {String} Base registry URL.
* `params` {Object} Object containing per-request properties.
  * `auth` {Credentials}
* `cb` {Function}
  * `error` {Error | null}
  * `data` {Object} the parsed data object
  * `raw` {String} the json
  * `res` {Response Object} response from couch

Add a user account to the registry, or verify the credentials.

### client.bugs(uri, params, cb)

* `uri` {String} Full registry URI for the package
* `params` {Object} Object containing per-request properties.
  * `auth` {Credentials} Optional.
* `cb` {Function}

Get the URL for bugs from a package's metadata.

### client.deprecate(uri, params, cb)

* `uri` {String} Full registry URI for the deprecated package.
* `params` {Object} Object containing per-request properties.
  * `version` {String} Semver version range.
  * `message` {String} The message to use as a deprecation warning.
  * `auth` {Credentials}
* `cb` {Function}

Deprecate a version of a package in the registry.

### client.get(uri, params, cb)

* `uri` {String} The complete registry URI to fetch
* `params` {Object} Object containing per-request properties.
  * `timeout` {Number} Duration before the request times out. Optional
    (default: never).
  * `follow` {Boolean} Follow 302/301 responses. Optional (default: true).
  * `staleOk` {Boolean} If there's cached data available, then return that to
    the callback quickly, and update the cache the background. Optional
    (default: false).
  * `auth` {Credentials} Optional.
* `cb` {Function}

Fetches data from the registry via a GET request, saving it in the cache folder
with the ETag.

### client.publish(uri, params, cb)

* `uri` {String} The registry URI for the package to publish.
* `params` {Object} Object containing per-request properties.
  * `metadata` {Object} Package metadata.
  * `body` {Stream} Stream of the package body / tarball.
  * `auth` {Credentials}
* `cb` {Function}

Publish a package to the registry.

Note that this does not create the tarball from a folder.

### client.star(uri, params, cb)

* `uri` {String} The complete registry URI for the package to star.
* `params` {Object} Object containing per-request properties.
  * `starred` {Boolean} True to star the package, false to unstar it. Optional
    (default: false).
  * `auth` {Credentials}
* `cb` {Function}

Star or unstar a package.

Note that the user does not have to be the package owner to star or unstar a
package, though other writes do require that the user be the package owner.

### client.stars(uri, params, cb)

* `uri` {String} The base URL for the registry.
* `params` {Object} Object containing per-request properties.
  * `username` {String} Name of user to fetch starred packages for. Optional
    (default: user in `auth`).
  * `auth` {Credentials} Optional (required if `username` is omitted).
* `cb` {Function}

View your own or another user's starred packages.

### client.tag(uri, params, cb)

* `uri` {String} The complete registry URI to tag
* `params` {Object} Object containing per-request properties.
  * `version` {String} Version to tag.
  * `tag` {String} Tag name to apply.
  * `auth` {Credentials}
* `cb` {Function}

Mark a version in the `dist-tags` hash, so that `pkg@tag` will fetch the
specified version.

### client.unpublish(uri, params, cb)

* `uri` {String} The complete registry URI of the package to unpublish.
* `params` {Object} Object containing per-request properties.
  * `version` {String} version to unpublish. Optional – omit to unpublish all
    versions.
  * `auth` {Credentials}
* `cb` {Function}

Remove a version of a package (or all versions) from the registry.  When the
last version us unpublished, the entire document is removed from the database.

### client.whoami(uri, params, cb)

* `uri` {String} The base registry for the URI.
* `params` {Object} Object containing per-request properties.
  * `auth` {Credentials}
* `cb` {Function}

Simple call to see who the registry thinks you are. Especially useful with
token-based auth.


## PLUMBING

The below are primarily intended for use by the rest of the API, or by the npm
caching logic directly.

### client.request(uri, params, cb)

* `uri` {String} URI pointing to the resource to request.
* `params` {Object} Object containing per-request properties.
  * `method` {String} HTTP method. Optional (default: "GET").
  * `body` {Stream | Buffer | String | Object} The request body.  Objects
    that are not Buffers or Streams are encoded as JSON. Optional – body
    only used for write operations.
  * `etag` {String} The cached ETag. Optional.
  * `follow` {Boolean} Follow 302/301 responses. Optional (default: true).
  * `auth` {Credentials} Optional.
* `cb` {Function}
  * `error` {Error | null}
  * `data` {Object} the parsed data object
  * `raw` {String} the json
  * `res` {Response Object} response from couch

Make a generic request to the registry. All the other methods are wrappers
around `client.request`.

### client.fetch(uri, params, cb)

* `uri` {String} The complete registry URI to upload to
* `params` {Object} Object containing per-request properties.
  * `headers` {Stream} HTTP headers to be included with the request. Optional.
  * `auth` {Credentials} Optional.
* `cb` {Function}

Fetch a package from a URL, with auth set appropriately if included. Used to
cache remote tarballs as well as request package tarballs from the registry.

### client.upload(uri, params, cb)

* `uri` {String} The complete registry URI to upload to
* `params` {Object} Object containing per-request properties.
  * `body` {Stream} A readable stream containing the package body.
  * `etag` {String} Cache ETag. Optional.
  * `follow` {Boolean} Whether to follow 301/302 responses. Optional (default: true).
  * `auth` {Credentials}
* `cb` {Function}

Upload an attachment.  Mostly used by `client.publish()`.
