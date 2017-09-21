# Webrouse/n-asset-macro

[![Build Status](https://travis-ci.org/webrouse/n-asset-macro.svg?branch=master)](https://travis-ci.org/webrouse/n-asset-macro)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/webrouse/n-asset-macro/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/webrouse/n-asset-macro/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/webrouse/n-asset-macro/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/webrouse/n-asset-macro/?branch=master)
[![Latest stable](https://img.shields.io/packagist/v/webrouse/n-asset-macro.svg)](https://packagist.org/packages/webrouse/n-asset-macro)

Asset macro for [Latte](https://latte.nette.org) and [Nette Framework](https://nette.org).

Useful for assets [cache busting](https://www.keycdn.com/support/what-is-cache-busting)
with [gulp](https://github.com/webrouse/n-asset-macro/tree/master/examples/gulp "Gulp example"), [webpack](https://github.com/webrouse/n-asset-macro/tree/master/examples/webpack "Webpack example"), [grunt](https://github.com/webrouse/n-asset-macro/tree/master/examples/grunt "Grunt example") and other similar tools.

## Requirements

* [PHP](https://php.net) >=5.6
* [Nette](https://github.com/nette) >=2.3
* [Latte](https://github.com/nette/latte) >=2.4

`PHP 7.1` and `Nette 3` are fully supported and tested.

## Installation

The best way to install **webrouse/n-asset-macro** is using  [Composer](http://getcomposer.org/):

```sh
$ composer require webrouse/n-asset-macro
```

Then register the extension in the config file:
```yaml
# app/config/config.neon
extensions:
    assetMacro: Webrouse\AssetMacro\DI\Extension
```
## Usage

Macro can by used in any presenter or control template:
```latte
{* app/presenters/templates/@layout.latte *}
<script src="{asset resources/vendor.js}"></script>
<script src="{asset resources/main.js}"></script>
```

It prepends path with ```$basePath``` and loads revision from the [revision manifest](#revision-manifest):

```html
<script src="/base/path/resources/vendor.d78da025b7.js"></script>
<script src="/base/path/resources/main.34edebe2a2.js"></script>
```

See the [examples](#examples) for usage with [gulp](https://github.com/webrouse/n-asset-macro/tree/master/examples/gulp "Gulp example"), [webpack](https://github.com/webrouse/n-asset-macro/tree/master/examples/webpack "Webpack example"), [grunt](https://github.com/webrouse/n-asset-macro/tree/master/examples/grunt "Grunt example").

## Revision manifest

**Revision manifest is a JSON file that contains the revision (path or version) of asset.**

It can be generated by various asset processors such as [gulp](https://github.com/webrouse/n-asset-macro/tree/master/examples/gulp), [grunt](https://github.com/webrouse/n-asset-macro/tree/master/examples/grunt) and [webpack](https://github.com/webrouse/n-asset-macro/tree/master/examples/webpack), [see examples](#examples).

Revision manifest is searched in the asset directory and in the parent directories up to `%wwwDir%`.

Expected file names: `assets.json`, `busters.json`, `versions.json`, `manifest.json`, `rev-manifest.json`.

**The path to revision manifest can be set directly (instead of autodetection):**
```yaml
# app/config/config.neon
assetMacro:
    revManifest: %wwwDir%/assets.json
```

**Or you can specify `asset => revision` pairs in config file:**

```yaml
# app/config/config.neon
assetMacro:
    revManifest:
      'js/vendor.js': 16016edc74d  # or js/vendor.16016edc74d.js
      'js/main.js':  4b82916016    # or js/main.4b82916016.js
```

Revision manifest may contains asset version or the asset path. Both ways are supported.

### Method 1: asset version in file name

With this method, the files have a **different name at each change**. 

Example revision manifest:
```json
{
	"js/app.js": "js/app.234a81ab33.js",
	"js/vendor.js": "js/vendor.d67fbce193.js",
	"js/locales/en.js": "js/locales/en.d78da025b7.js",
	"js/locales/sk.js": "js/locales/sk.34edebe2a2.js",
	"css/app.css": "css/app.04b5ff0b97.js"
}
```

With the example manifest, the expr. `{asset "js/app.js"}` generates: `/base/path/js/app.234a81ab33.js`.

### Method 2: asset version as a query string

This approach looks better at first glance. The **asset path is still the same**, and only the parameter in the query changes.

However, it can [cause problems](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/) with some cache servers, which don't take the URL parameters into account.

Example revision manifest:
```json
{
	"js/app.js": "234a81ab33",
	"js/vendor.js": "d67fbce193",
	"js/locales/en.js": "d78da025b7",
	"js/locales/sk.js": "34edebe2a2",
	"css/app.css": "04b5ff0b97"
}
```

With the example manifest, the expr. `{asset "js/app.js"}` generates: `/base/path/js/app.js?v=234a81ab33`.

Asset macro automatically detects which of these two formats of revision manifest is used.

## Macro arguments

### `format`

The format is defined by the second macro parameter or using the `format` key (default `%url%`).

`format` can be used with `needed => false` to hide whole asset expression (eg. `<link ...`) in case of an error.

**You can also use it to include asset content instead of a path.**

| Placeholder  | Example output                                                            |
| -------------|---------------------------------------------------------------------------|
| `%content%`  | `<svg>....</svg>` (file content)                                            |
| `%url%`      | `/base/path/js/main.js?v=8c48f58df` or `/base/path/js/main.8c48f58df.js`  |
| `%path%`     | `js/main.js` or `js/main.8c48f58df.js`                                    |
| `%raw%`      | `8c48f58df` or `js/main.8c48f58df.js`                                     |
| `%basePath%` | `/base/path`                                                              |

```latte
{* app/presenters/templates/@layout.latte *}
{asset 'js/vendor.js', '<script src="%url%"></script>'}
<script src="{asset 'js/livereload.js', format => '%path%?host=localhost&v=%raw%'}"></script>
```

### `needed`

Error handling is set in the [configuration](https://github.com/webrouse/n-asset-macro/blob/master/README.md#configuration) using: `missingAsset`, `missingManifest`, `missingRevision` keys.

These settings can by overrided by third macro parameter or using `needed` key (default `true`).

Argument `needed => false` will cause the missing file or the missing revision record will be ignored.

Empty string will be result for missing asset. Missing version will be replaced with `unknown` string.

**Example of `needed` parameter**
 * `absent.js` file doesn't exist.
 * `missing_rev.js` exists but doesn't have revision in manifest (or the manifest has not been found).

```latte
{asset 'js/absent.js', '<script src="%url%"></script>', FALSE}
{asset 'js/missing_rev.js', format => '<script src="%url%"></script>', needed => FALSE}
```

Generated output:
```html
<script src="/base/path/js/missing_rev.js?v=unknown"></script>
```
## Caching

In production mode is macro output cached in [global cache storage](https://doc.nette.org/en/2.4/caching). It can be changed in the [configuration](https://github.com/webrouse/n-asset-macro/blob/master/README.md#configuration) using `cache` key (`true` or `false`).

## Configuration

Default configuration, which usually doesn't need to be changed:

```yaml
# app/config/config.neon
assetMacro:
    # Cache generated output
    cache: ! %debugMode%
    # Path to revision manifest or asset => revision pairs,
    # if set, the autodetection is switched off
    revManifest: null # %wwwDir%/assets.json
    # File names for automatic detection of revision manifest
    autodetect:
        - assets.json
        - busters.json
        - versions.json
        - manifest.json
        - rev-manifest.json
    # Action if missing asset file: exception, notice, or ignore
    missingAsset: notice
    # Action if missing manifest file: exception, notice, or ignore
    missingManifest: notice
    # Action if missing asset revision in manifest: exception, notice, or ignore
    missingRevision: notice
```

## Examples

Examples based on [nette/sandbox](https://github.com/nette/sandbox):

* [Gulp + Asset macro](https://github.com/webrouse/n-asset-macro/tree/master/examples/gulp)
* [Grunt + Asset macro](https://github.com/webrouse/n-asset-macro/tree/master/examples/grunt)
* [Webpack + Asset macro](https://github.com/webrouse/n-asset-macro/tree/master/examples/webpack)

## License

N-asset-macro is under the MIT license. See the [LICENSE](LICENSE.md) file for details.
