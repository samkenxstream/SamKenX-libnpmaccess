# npm-registry-fetch [![npm version](https://img.shields.io/npm/v/npm-registry-fetch.svg)](https://npm.im/npm-registry-fetch) [![license](https://img.shields.io/npm/l/npm-registry-fetch.svg)](https://npm.im/npm-registry-fetch) [![Travis](https://img.shields.io/travis/npm/npm-registry-fetch/latest.svg)](https://travis-ci.org/npm/npm-registry-fetch) [![AppVeyor](https://img.shields.io/appveyor/ci/zkat/npm-registry-fetch/latest.svg)](https://ci.appveyor.com/project/npm/npm-registry-fetch) [![Coverage Status](https://coveralls.io/repos/github/npm/npm-registry-fetch/badge.svg?branch=latest)](https://coveralls.io/github/npm/npm-registry-fetch?branch=latest)

[`npm-registry-fetch`](https://github.com/npm/npm-registry-fetch) is a Node.js
library that provides programmatic access to the guts of the npm CLI's `npm
access` command and its various subcommands. This includes managing account 2FA,
listing packages and permissions, looking at package collaborators, and defining
package permissions for users, orgs, and teams.

## Example

```javascript
const access = require('libnpmaccess')

// List all packages @zkat has access to on the npm registry.
console.log(Object.keys(await access.lsPackages('zkat')))
```

## Table of Contents

* [Installing](#install)
* [Example](#example)
* [Contributing](#contributing)
* [API](#api)
  * [access opts](#opts)
  * [`public()`](#public)
  * [`restricted()`](#restricted)
  * [`grant()`](#grant)
  * [`revoke()`](#revoke)
  * [`tfaRequired()`](#tfa-required)
  * [`tfaNotRequired()`](#tfa-not-required)
  * [`lsPackages()`](#ls-packages)
  * [`lsCollaborators()`](#ls-collaborators)

### Install

`$ npm install libnpmaccess`

### Contributing

The npm team enthusiastically welcomes contributions and project participation!
There's a bunch of things you can do if you want to contribute! The [Contributor
Guide](CONTRIBUTING.md) has all the information you need for everything from
reporting bugs to contributing entire new features. Please don't hesitate to
jump in if you'd like to, or even ask us questions if something isn't clear.

All participants and maintainers in this project are expected to follow [Code of
Conduct](CODE_OF_CONDUCT.md), and just generally be excellent to each other.

Please refer to the [Changelog](CHANGELOG.md) for project history details, too.

Happy hacking!

### API

#### <a name="opts"></a> `opts` for `libnpmaccess` commands

`libnpmaccess` uses [`npm-registry-fetch`](https://npm.im/npm-registry-fetch).
All options are passed through directly to that library, so please refer to [its
own `opts`
documentation](https://www.npmjs.com/package/npm-registry-fetch#fetch-options)
for options that can be passed in.

A couple of options of note for those in a hurry:

* `opts.token` - can be passed in and will be used as the authentication token for the registry. For other ways to pass in auth details, see the n-r-f docs.
* `opts.otp` - certain operations will require an OTP token to be passed in. If a `libnpmaccess` command fails with `err.code === EOTP`, please retry the request with `{otp: <2fa token>}`
* `opts.Promise` - If you pass this in, the Promises returned by `libnpmaccess` commands will use this Promise class instead. For example: `{Promise: require('bluebird')}`

#### <a name="public"></a> `> access.public(spec, [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec.

Makes package described by `spec` public.

##### Example

```javascript
await access.public('@foo/bar', {token: 'myregistrytoken'})
// `@foo/bar` is now public
```

#### <a name="restricted"></a> `> access.restricted(spec, [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec.

Makes package described by `spec` private/restricted.

##### Example

```javascript
await access.restricted('@foo/bar', {token: 'myregistrytoken'})
// `@foo/bar` is now private
```

#### <a name="grant"></a> `> access.grant(spec, scope, team, permissions, [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec. `scope` must be a valid scope, with or without the `@` prefix,
and `team` must be a valid team within that scope. `permissions` must be one of
`'read-only'` or `'read-write'`.

Grants `read-only` or `read-write` permissions for a certain package to a team.

##### Example

```javascript
await access.grant('@foo/bar', '@foo', 'myteam', 'read-write', {
  token: 'myregistrytoken'
})
// `@foo/bar` is now read/write enabled for the @foo:myteam team.
```

#### <a name="revoke"></a> `> access.revoke(spec, scope, team, [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec. `scope` must be a valid scope, with or without the `@` prefix,
and `team` must be a valid team within that scope. `permissions` must be one of
`'read-only'` or `'read-write'`.

Removes access to a package from a certain team.

##### Example

```javascript
await access.revoke('@foo/bar', '@foo', 'myteam', {
  token: 'myregistrytoken'
})
// @foo:myteam can no longer access `@foo/bar`
```

#### <a name="tfa-required"></a> `> access.tfaRequired(spec, [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec.

Makes it so publishing or managing a package requires using 2FA tokens to
complete operations.

##### Example

```javascript
await access.tfaRequires('lodash', {token: 'myregistrytoken'})
// Publishing or changing dist-tags on `lodash` now require OTP to be enabled.
```

#### <a name="tfa-not-required"></a> `> access.tfaNotRequired(spec, [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec.

Disabled the package-level 2FA requirement for `spec`. Note that you will need
to pass in an `otp` token in `opts` in order to complete this operation.

##### Example

```javascript
await access.tfaNotRequired('lodash', {otp: '123654', token: 'myregistrytoken'})
// Publishing or editing dist-tags on `lodash` no longer requires OTP to be
// enabled.
```

#### <a name="ls-packages"></a> `> access.lsPackages(scope, [team], [opts]) -> Promise`

`scope` must be a valid org or user name, with or without the `@` prefix. `team`
is optional and, if provided, must be a valid team within that scope. `team`
must be `null` in order to pass in `opts`.

Lists out packages a user, org, or team has access to, with corresponding
permissions. Packages that the access token does not have access to won't be
listed.

##### Example

```javascript
await access.lsPackages('zkat', null, {
  token: 'myregistrytoken'
})
// Lists all packages `@zkat` has access to on the registry, and the
// corresponding permissions.
```

#### <a name="ls-collaborators"></a> `> access.lsCollaborators(spec, [user], [opts]) -> Promise`

`spec` must be an [`npm-package-arg`](https://npm.im/npm-package-arg)-compatible
registry spec. `scope` must be a valid org or user name, with or without the `@`
prefix. `team` is optional and, if provided, must be a valid team within that
scope. `team` must be `null` in order to pass in `opts`.

Lists out access privileges for a certain package. Will only show permissions
for packages to which you have at least read access. If `user` is passed in, the
list is filtered only to teams _that_ user happens to belong to.

##### Example

```javascript
await access.lsCollaborators('@npm/foo', 'zkat', {
  token: 'myregistrytoken'
})
// Lists all teams with access to @npm/foo that @zkat belongs to.
```