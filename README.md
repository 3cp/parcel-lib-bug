# Parcel bug on building lib bundle

This is a simple demo. The `src/index.js` is in commonjs format, using one npm package "mri" through `require('mri')`.

We use parcel to bundle it into `dist/index.js` in CommonJS format, including package "mri" throught `"includeNodeModules": true`.

Compare the output of source file and dist bundle file.
The source runs as expected, but target failed to run.

```
~/p/p> node src/index.js hello
{ _: [ 'hello' ] }
~/p/p> node dist/index.js hello
/Users/huocp/playground/p/dist/index.js:103
console.log($f7eee4adaf510650$exports($ade4ae6ad7d0284b$var$argv));
						^

TypeError: $f7eee4adaf510650$exports is not a function
		at Object.<anonymous> (/Users/huocp/playground/p/dist/index.js:103:13)
		at Module._compile (internal/modules/cjs/loader.js:1085:14)
		at Object.Module._extensions..js (internal/modules/cjs/loader.js:1114:10)
		at Module.load (internal/modules/cjs/loader.js:950:32)
		at Function.Module._load (internal/modules/cjs/loader.js:790:12)
		at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:76:12)
		at internal/main/run_main_module.js:17:47
```

The reason of the failure in bundled file, is that "mri" package is actually a CommonJS format npm package. But it has a "module" field in package.json pointing to an esm format.
```
// mri package.json
"main": "lib/index.js",
"module": "lib/index.mjs",
```

The "module" field (and also "browser" field) in package.json is NOT npm standard. In Nodejs environment, it's never utilised.

The "module" and "browser" fields are traditions developed by various web app bundler, starting with browserify I believe.

It's understandable for parcel to utilise "module" field when bundling for browser target. However when targeting Nodejs lib, parcel should follow Nodejs practice and avoid peeking "module" and "browser" fields.

In the bundle source map `dist/index.js.map`, you can clearly see parcel uses `"module": "lib/index.mjs"` from "mri"'s package.json.

```
sources":["src/index.js","node_modules/mri/lib/index.mjs"]
```

