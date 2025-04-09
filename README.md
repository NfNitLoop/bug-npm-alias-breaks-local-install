README
======

This repository demonstrates a bug with `pnpm install`.

To reproduce:

 * Clone this repo
 * `pnpm install`
 * `pnpm test`

The Bug
-------

This is a PNPM workspace with two example library packages:
  * `foo` - has no dependencies
  * `bar` - has a workspace dependency on `foo`. 

We `pnpm pack` both of these libraries to create .tgz files for them.

Then we use `pnpm install file:...` to install them into a new `installDir/`.

### Expected:

When we install `bar`, `foo` is already installed, so `pnpm` doesn't need to download it.

### Actual:

`pnpm` tries to download `foo` from npm, even though it's already installed locally.

```
> cd installDir && pnpm install file:../pkgDist/my_private_scope_example-bar-1.0.0.tgz

 ERR_PNPM_FETCH_404  GET https://registry.npmjs.org/@my_private_scope_example%2Ffoo: Not Found - 404

This error happened while installing the dependencies of @my_private_scope_example/bar@1.0.0

@my_private_scope_example/foo is not in the npm registry, or you have no permission to fetch it.

No authorization header was set for the request.
..                                       | Progress: resolved 2, reused 2, downloaded 0, added 0
 ELIFECYCLE  Command failed with exit code 1.
 ELIFECYCLE  Command failed with exit code 1.
 ELIFECYCLE  Test failed. See above for more details.
```

`--prefer-offline` does not change this behavior.

`--offline` changes the error to:

```
> cd installDir && pnpm install file:../pkgDist/my_private_scope_example-bar-1.0.0.tgz --offline

 ERR_PNPM_NO_OFFLINE_META  Failed to resolve @my_private_scope_example/foo@latest in package mirror /Users/codycasterline/Library/Caches/pnpm/metadata-v1.3/registry.npmjs.org/@my_private_scope_example/foo.json

This error happened while installing the dependencies of @my_private_scope_example/bar@1.0.0
..                                       | Progress: resolved 0, reused 1, downloaded 0, added 0
 ELIFECYCLE  Command failed with exit code 1.
```

I've reproduced this with PNPM 10.2.0 and 10.8.0.

Works in `npm`
--------------

`npm install` does not have this problem.  I may end up using it instead of `pnpm` for this reason.

To reproduce: 

 * update the `installFoo` and `installBar` tasks in the root package.json to use `npm` isntead of `pnpm`.
 * `pnpm test`


