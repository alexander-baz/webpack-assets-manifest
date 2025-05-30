# Webpack Assets Manifest

[![Node.js CI](https://github.com/webdeveric/webpack-assets-manifest/actions/workflows/node.js.yml/badge.svg)](https://github.com/webdeveric/webpack-assets-manifest/actions/workflows/node.js.yml) [![codecov](https://codecov.io/gh/webdeveric/webpack-assets-manifest/branch/master/graph/badge.svg)](https://codecov.io/gh/webdeveric/webpack-assets-manifest)

This webpack plugin will generate a JSON file that matches the original filename with the hashed version.

## Installation

```shell
pnpm add -D webpack-assets-manifest
```

```shell
npm install webpack-assets-manifest -D
```

```shell
yarn add webpack-assets-manifest -D
```

## New in version 6

- This plugin is now written with TypeScript.
- TypeScript types are provided by this package.
- The package exports `esm` and `cjs`.
- Added `manifest.utils` for use in the `customize` hook.
  - See [examples/customized.js](https://github.com/webdeveric/webpack-assets-manifest/blob/master/examples/customized.js)
  - https://github.com/webdeveric/webpack-assets-manifest/blob/b6ed8c09394bf33c8fadd5adff8bad7694f295ba/src/plugin.ts#L185-L195

### Breaking changes

- Bumped minimum webpack version to `5.61`
- Bumped minimum Node version to `20.10`
- The plugin is exported using a named export instead of default.
  - Use `import { WebpackAssetsManifest } from 'webpack-assets-manifest';`

## New in version 5

- Compatible with webpack 5 only (5.1+ required).
- Supports finding [asset modules](https://webpack.js.org/guides/asset-modules/).
- Updated options schema to prevent additional properties. This helps with catching typos in option names.
- :warning: Updated default value of the `output` option to be `assets-manifest.json`. This is to prevent confusion when working with [Web app manifests](https://developer.mozilla.org/en-US/docs/Web/Manifest) or [WebExtension manifests](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json).

## New in version 4

- Requires Node 10+.
- Compatible with webpack 4 only (4.40+ required).
- Added options: [`enabled`](#enabled), [`entrypointsUseAssets`](#entrypointsUseAssets), [`contextRelativeKeys`](#contextRelativeKeys).
- Updated [`writeToDisk`](#writeToDisk) option to default to `auto`.
- Use lock files for various operations.
- `done` hook is now an `AsyncSeriesHook`.
- :warning: The structure of the `entrypoints` data has been updated to include `preload` and `prefetch` assets. Assets for an entrypoint are now included in an `assets` property under the entrypoint.

  Example:

  ```json
  {
    "entrypoints": {
      "main": {
        "assets": {
          "css": ["main.css"],
          "js": ["main.js"]
        },
        "prefetch": {
          "js": ["prefetch.js"]
        },
        "preload": {
          "js": ["preload.js"]
        }
      }
    }
  }
  ```

## Usage

In your webpack config, require the plugin then add an instance to the `plugins` array.

```js
import { WebpackAssetsManifest } from 'webpack-assets-manifest';

export default {
  entry: {
    // Your entry points
  },
  output: {
    // Your output details
  },
  module: {
    // Your loader rules go here.
  },
  plugins: [
    new WebpackAssetsManifest({
      // Options go here
    }),
  ],
};
```

## Sample output

```json
{
  "main.js": "main-9c68d5e8de1b810a80e4.js",
  "main.css": "main-9c68d5e8de1b810a80e4.css",
  "images/logo.svg": "images/logo-b111da4f34cefce092b965ebc1078ee3.svg"
}
```

---

## Options ([read the schema](src/options-schema.ts))

### `enabled`

Type: `boolean`

Default: `true`

Is the plugin enabled?

### `output`

Type: `string`

Default: `assets-manifest.json`

This is where to save the manifest file relative to your webpack `output.path`.

### `assets`

Type: `object`

Default: `{}`

Data is stored in this object.

#### Sharing data

You can share data between instances by passing in your own object in the `assets` option.

This is useful in [multi-compiler mode](https://github.com/webpack/webpack/tree/master/examples/multi-compiler).

```js
const data = Object.create(null);

const manifest1 = new WebpackAssetsManifest({
  assets: data,
});

const manifest2 = new WebpackAssetsManifest({
  assets: data,
});
```

### `contextRelativeKeys`

Type: `boolean`

Default: `false`

Keys are relative to the compiler context.

### `space`

Type: `int`

Default: `2`

Number of spaces to use for pretty printing.

### `replacer`

Type: `null`, `function`, or `array`

Default: `null`

[Replacer reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#The_replacer_parameter)

You'll probably want to use the `transform` hook instead.

### `fileExtRegex`

Type: `regex`

Default: `/\.\w{2,4}\.(?:map|gz)$|\.\w+$/i`

This is the regular expression used to find file extensions. You'll probably never need to change this.

### `writeToDisk`

Type: `boolean`, `string`

Default: `'auto'`

Write the manifest to disk using `fs`.

:warning: If you're using another language for your site and you're using `webpack-dev-server` to process your assets during development, you should set `writeToDisk: true` and provide an absolute path in `output` so the manifest file is actually written to disk and not kept only in memory.

### `sortManifest`

Type: `boolean`, `function`

Default: `true`

The manifest is sorted alphabetically by default. You can turn off sorting by setting `sortManifest: false`.

If you want more control over how the manifest is sorted, you can provide your own [comparison function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort#Parameters). See the [sorted](examples/sorted.js) example.

```js
new WebpackAssetsManifest({
  sortManifest(left, right) {
    // Return -1, 0, or 1
  },
});
```

### `merge`

Type: `boolean`, `string`

Default: `false`

If the `output` file already exists and you'd like to add to it, use `merge: true`. The default behavior is to use the existing keys/values without modification.

```js
new WebpackAssetsManifest({
  output: '/path/to/manifest.json',
  merge: true,
});
```

If you need to customize during merge, use `merge: 'customize'`.

If you want to know if `customize` was called when merging with an existing manifest, you can check `manifest.isMerging`.

```js
new WebpackAssetsManifest({
  merge: 'customize',
  customize(entry, original, manifest, asset) {
    if ( manifest.isMerging ) {
      // Do something
    }
  },
}),
```

### `publicPath`

Type: `string`, `function`, `boolean`,

Default: `null`

When using `publicPath: true`, your webpack config `output.publicPath` will be used as the value prefix.

```js
const manifest = new WebpackAssetsManifest({
  publicPath: true,
});
```

When using a string, it will be the value prefix. One common use is to prefix your CDN URL.

```js
const manifest = new WebpackAssetsManifest({
  publicPath: 'https://cdn.example.com',
});
```

If you'd like to have more control, use a function. See the [custom CDN](examples/custom-cdn.js) example.

```js
const manifest = new WebpackAssetsManifest({
  publicPath(filename, manifest) {
    // customize filename here
    return filename;
  },
});
```

### `entrypoints`

Type: `boolean`

Default: `false`

Include `compilation.entrypoints` in the manifest file.

### `entrypointsKey`

Type: `string`, `boolean`

Default: `entrypoints`

If this is set to `false`, the `entrypoints` will be added to the root of the manifest.

### `entrypointsUseAssets`

Type: `boolean`

Default: `false`

Entrypoint data should use the value from `assets`, which means the values could be customized and not just a `string` file path. This new option defaults to `false` so the new behavior is opt-in.

### `integrity`

Type: `boolean`

Default: `false`

Include the [subresource integrity hash](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity).

### `integrityHashes`

Type: `array`

Default: `[ 'sha256', 'sha384', 'sha512' ]`

Hash algorithms to use when generating SRI. For browsers, the currently the allowed integrity hashes are `sha256`, `sha384`, and `sha512`.

Other hash algorithms can be used if your target environment is not a browser. If you were to create a tool to audit your S3 buckets for [data integrity](https://aws.amazon.com/premiumsupport/knowledge-center/data-integrity-s3/), you could use something like this [example](examples/aws-s3-data-integrity.js) to record the `md5` hashes.

### `integrityPropertyName`

Type: `string`

Default: `integrity`

This is the property that will be set on each entry in `compilation.assets`, which will then be available during `customize`. It is customizable so that you can have multiple instances of this plugin and not have them overwrite the `currentAsset.integrity` property.

You'll probably only need to change this if you're using multiple instances of this plugin to create different manifests.

### `apply`

Type: `function`

Default: `null`

Callback to run after setup is complete.

### `customize`

Type: `function`

Default: `null`

Callback to customize each entry in the manifest.

You can use this to customize entry names for example. In the sample below, we adjust `img` keys so that it's easier to use them with a template engine:

```javascript
new WebpackAssetsManifest({
  customize(entry, original, manifest, asset) {
    if (manifest.utils.isKeyValuePair(entry) && entry.key.startsWith('img/')) {
      return { key: entry.key.split('img/')[1], value: entry.value };
    }

    return entry;
  }
}
```

The function is called per each entry and provides you a way to intercept and rewrite each object. The result is then merged into a whole manifest.

[View the example](examples/customized.js) to see what else you can do with this function.

### `transform`

Type: `function`

Default: `null`

Callback to transform the entire manifest.

### `done`

Type: `function`

Default: `null`

Callback to run after the compilation is done and the manifest has been written.

---

### Hooks

This plugin is using hooks from [Tapable](https://github.com/webpack/tapable/).

The `apply`, `customize`, `transform`, and `done` options are automatically tapped into the appropriate hook.

| Name           | Type                | Callback signature                             |
| -------------- | ------------------- | ---------------------------------------------- |
| `apply`        | `SyncHook`          | `function(manifest){}`                         |
| `customize`    | `SyncWaterfallHook` | `function(entry, original, manifest, asset){}` |
| `transform`    | `SyncWaterfallHook` | `function(assets, manifest){}`                 |
| `done`         | `AsyncSeriesHook`   | `async function(manifest, stats){}`            |
| `options`      | `SyncWaterfallHook` | `function(options){}`                          |
| `afterOptions` | `SyncHook`          | `function(options){}`                          |

#### Tapping into hooks

Tap into a hook by calling the `tap` method on the hook as shown below.

If you want more control over exactly what gets added to your manifest, then use the `customize` and `transform` hooks. See the [customized](examples/customized.js) and [transformed](examples/transformed.js) examples.

```js
const manifest = new WebpackAssetsManifest();

manifest.hooks.apply.tap('YourPluginName', function (manifest) {
  // Do something here
  manifest.set('some-key', 'some-value');
});

manifest.hooks.customize.tap('YourPluginName', function (entry, original, manifest, asset) {
  // customize entry here
  return entry;
});

manifest.hooks.transform.tap('YourPluginName', function (assets, manifest) {
  // customize assets here
  return assets;
});

manifest.hooks.options.tap('YourPluginName', function (options) {
  // customize options here
  return options;
});

manifest.hooks.done.tap('YourPluginName', function (manifest, stats) {
  console.log(`The manifest has been written to ${manifest.getOutputPath()}`);
  console.log(`${manifest}`);
});

manifest.hooks.done.tapPromise('YourPluginName', async (manifest, stats) => {
  await yourAsyncOperation();
});
```

These hooks can also be set by passing them in the constructor options.

```js
new WebpackAssetsManifest({
  done(manifest, stats) {
    console.log(`The manifest has been written to ${manifest.getOutputPath()}`);
    console.log(`${manifest}`);
  },
});
```

## Manifest methods

If the manifest instance is passed to a hook, you can use the following methods to manage what goes into the manifest.

- `has(key)`
- `get(key)`
- `set(key, value)`
- `setRaw(key, value)`
- `delete(key)`

If you want to write the manifest to another location, you can use `writeTo(destination)`.

```js
new WebpackAssetsManifest({
  async done(manifest) {
    await manifest.writeTo('/some/other/path/assets-manifest.json');
  },
});
```
