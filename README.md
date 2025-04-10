NPM Install Bug
===============

Reproduction
------------

With pnpm:
 * clone this repo
 * `pnpm install`
 * `pnpm test`

(pnpm is just how I'm creating the example packages. But we install them with `npm`)

With npm:
 * clone this repo
 * `npm install`
 * `npm run localInstall`

The Bug
-------

When installing local .tgz packages, using this dependency format:

```json
"dependencies": {
    "@my_private_scope_example/foo": "1.0.0"
}
```

... then, if the dependency is already present in the node_modules, `npm` will install the package correctly.


However, if the dependency uses an alias, this breaks. (See: [example commit][1])

[1]: https://github.com/NfNitLoop/bug-npm-alias-breaks-local-install/commit/8d03d5309a7c6bb5687b2c193aba6996a198cb85

```json
"dependencies": {
    "@my_private_scope_example/myFooAlias": "npm:@my_private_scope_example/foo@1.0.0"
  },
```

output:
```
> cd installDir && npm install --verbose file:../pkgDist/my_private_scope_example-bar-1.0.0.tgz

npm verbose cli /opt/homebrew/Cellar/node/23.7.0/bin/node /opt/homebrew/bin/npm
npm info using npm@11.3.0
npm info using node@v23.7.0
npm warn Unknown env config "verify-deps-before-run". This will stop working in the next major version of npm.
npm warn Unknown env config "resolution-mode". This will stop working in the next major version of npm.
npm warn Unknown env config "package-manager". This will stop working in the next major version of npm.
npm verbose title npm install file:../pkgDist/my_private_scope_example-bar-1.0.0.tgz
npm verbose argv "install" "--loglevel" "verbose" "file:../pkgDist/my_private_scope_example-bar-1.0.0.tgz"
npm verbose logfile logs-max:10 dir:/Users/codycasterline/.npm/_logs/2025-04-10T18_47_54_792Z-
npm verbose logfile /Users/codycasterline/.npm/_logs/2025-04-10T18_47_54_792Z-debug-0.log
npm http fetch GET 404 https://registry.npmjs.org/@my_private_scope_example%2ffoo 225ms (cache skip)
npm http fetch GET 404 https://registry.npmjs.org/@my_private_scope_example%2ffoo 66ms (cache skip)
npm verbose stack HttpErrorGeneral: 404 Not Found - GET https://registry.npmjs.org/@my_private_scope_example%2ffoo - Not found
npm verbose stack     at /opt/homebrew/lib/node_modules/npm/node_modules/npm-registry-fetch/lib/check-response.js:103:15
npm verbose stack     at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
npm verbose stack     at async RegistryFetcher.packument (/opt/homebrew/lib/node_modules/npm/node_modules/pacote/lib/registry.js:90:19)
npm verbose stack     at async RegistryFetcher.manifest (/opt/homebrew/lib/node_modules/npm/node_modules/pacote/lib/registry.js:128:23)
npm verbose stack     at async #fetchManifest (/opt/homebrew/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/build-ideal-tree.js:1213:20)
npm verbose stack     at async #nodeFromEdge (/opt/homebrew/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/build-ideal-tree.js:1051:19)
npm verbose stack     at async #buildDepStep (/opt/homebrew/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/build-ideal-tree.js:915:11)
npm verbose stack     at async Arborist.buildIdealTree (/opt/homebrew/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/build-ideal-tree.js:182:7)
npm verbose stack     at async Promise.all (index 1)
npm verbose stack     at async Arborist.reify (/opt/homebrew/lib/node_modules/npm/node_modules/@npmcli/arborist/lib/arborist/reify.js:130:5)
npm verbose statusCode 404
npm verbose pkgid @my_private_scope_example/foo@1.0.0
npm error code E404
npm error 404 Not Found - GET https://registry.npmjs.org/@my_private_scope_example%2ffoo - Not found
npm error 404
npm error 404  '@my_private_scope_example/foo@1.0.0' is not in this registry.
npm error 404
npm error 404 Note that you can also install from a
npm error 404 tarball, folder, http url, or git url.
npm verbose cwd /Users/codycasterline/code/bug-npm-alias-breaks-local-install/installDir
npm verbose os Darwin 24.3.0
npm verbose node v23.7.0
npm verbose npm  v11.3.0
npm verbose exit 1
npm verbose code 1
```

`npm` appears to try to fetch the dependency from npmjs.org in this case. But these packages are not (and will not be) published there.
