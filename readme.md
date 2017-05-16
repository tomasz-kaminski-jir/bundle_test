[![Build Status](https://travis-ci.org/springmeyer/bundle-dedupe-testcase.svg?branch=master)](https://travis-ci.org/springmeyer/bundle-dedupe-testcase)

This testcase replicates an error of:

```
npm ERR! missing: minimist@0.0.8, required by mkdirp@0.5.1
```

That happens with node v6.10.3 and npm v3.10.10 when `npm ls` is run after install of the module. What appears to be happening is that:

 - The module depends on `minimist@~1.2.0`
 - The module depends on `node-pre-gyp` which depends on `mkdirp@^0.5.1`
 - `mkdirp@0.5.1` is pinned exactly to `minimist@0.0.8`: https://github.com/substack/node-mkdirp/blob/f2003bbcffa80f8c9744579fabab1212fc84545a/package.json#L19
 - During npm install the `mkdirp` is installed in the node-pre-gyp modules folder
 - With node v0.10, v4, and v7 the `minimist@1.2.0` is installed at the top level and `minimist@0.0.8` is installed within the `mkdirp` dependended upon my `node-pre-gyp`.
 - With node v6 the `minimist@1.2.0` is installed at the top level and `minimist@0.0.8` is NOT installed within the `mkdirp` dependended upon my `node-pre-gyp`.
 - During npm install `node-pre-gyp` is run and the `mkdirp` appears to work at runtime with all versions of node despite the deduping problem with node v6
 - But with node v6 the `npm ls` that is run during `prepublish` (which is automatically run by npm at the end of the install) errors since the pinnned `minimist@0.0.8` is missing.

The tree that `npm ls` is outputs (when it returns an error with node v6) is:

 ```
 └─┬ node-pre-gyp@0.6.34
  ├���┬ mkdirp@0.5.1
  │ └── UNMET DEPENDENCY minimist@0.0.8
 ```

This does not happen with node v7 and npm v4.2.0
