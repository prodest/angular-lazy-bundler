# Angular Lazy Bundler

## Table of contents
- [What does it do?](#what-does-it-do)
- [How is it different from tools like Browserify?](#how-is-it-different-from-tools-like-browserify)
- [Requirements](#requirements)
- [API](#api)
- [Usage example](#usage-example)
- [Troubleshooting](#troubleshooting)

## What does it do?

Lazy loading components at runtime gives us the advantage to ony load code that is going to be used by the client. At the same time it introduces one big disadvantage; it increases the number of network calls we need to make to load a component / library. This leads to slower page load times and affects the user experience negatively. This is what Angular Lazy Bundler tries to solve.

Lets take the following component as an example:

```text
+src
|  +components
|  |  +home-state
|  |  |  home-route.js
|  |  |  home-state.html
|  |  |  home-state-controller.js
|  |  |  index.js
```

To load the home state component the browser would need to make at least four requests. Angular Lazy Bundler solves this issue by creating a bundle of our source code per component. Given the above example, the bundler would create a combined file containing all four of the home state's files.

```text
+src
|  +build
|  |  +bundles
|  |  |  home-state.js
```

The same is applied for JSPM packages. As of now, if you load JSPM packages in the browser you at least make two server calls. One for the package file, e.g. `lodash@3.x.x.js`. The only purpose of that file is to reference the main package file defined in package.json, e.g. `index.js` for lodash. This is a necessary evil to make NPM packages easy loadable by name in the browser. The Bundler optimizes this load process by combining the package and the main file into one.

After creating the bundles you can tell Angular Lazy Bundler to also update your SystemJS configuration, so that the loader knows about the bundles and loads the combined resources instead of the individual files. See the [bundle config API](https://github.com/systemjs/systemjs/blob/master/docs/config-api.md#bundle) in the SystemJS documentation.

## How is it different from tools like Browserify?

Browserify and similar tools, by default, create one big bundle which contains all the application's source code. This works good for smaller applications. But when you have a few megabytes of code this becomes somewhat of a burden. Let's say our application has 20 different states / screens. The only thing the user wants to do is check his new messages which is one state. But even in that scenario the browser would actually download the remaining code of the application before the user can look at his inbox. This unnecessary wait time is what we want to omit by loading only the parts of the application the user is actually going to access.

## Requirements

For the bundler to work it's required to have a project structure as generated by [Angular Lazy Generator](github.com/matoilic/generator-angular-lazy). Each index.js file in the src folder indicates a root of a component. Only resources which are referenced / imported by the corresponding index.js file, and their sub-dependencies, will land in the resulting bundle. Files which are not imported statically from anywhere will not be bundled. But they will still get loaded by SystemJS individually.

## API

[new Bundler(options)](#new_Bundler_new)
- [.bundle(content, saveAs)](#Bundler+bundle)
- [.bundleComponent(index)](#Bundler+bundleComponent)
- [.bundleComponents(componentNames, saveAs)](#Bundler+bundleComponents)
- [.bundlePackage(packageName)](#Bundler+bundlePackage)
- [.bundlePackages(packageNames, saveAs)](#Bundler+bundlePackages)
- [.bundleRemainingComponents()](#Bundler+bundleRemainingComponents)
- [.bundleRemainingPackages()](#Bundler+bundleRemainingPackages)
- [.saveConfig()](#Bundler+saveConfig)

<a name="new_Bundler_new"></a>
### new Bundler(options)

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| options | <code>Object</code> |  | Bundler options. |
| [options.source] | <code>String</code> | <code>src</code> | Where to search for components / index.js files. |
| [options.baseUrl] | <code>String</code> | <code>.</code> | Base URL on the file system to use for bundling. |
| [options.dest] | <code>String</code> | <code>build/bundles</code> | Destination folder where the bundled resources will be written to. |
| [options.bundlesBaseUrl] | <code>String</code> | <code>bundles</code> | Path relative to the baseURL of SystemJS in the browser of the destination folder. |
| [options.systemJsConfig] | <code>String</code> | <code>config/system.js</code> | Path to the SystemJS configuration file. |
| [options.sourceMaps] | <code>Boolean</code> | <code>true</code> | Enable / disable sourcemap generation. |
| [options.minify] | <code>Boolean</code> | <code>true</code> | Enable / disable minification of bundled resources. |
| [options.cssOptimize] | <code>Boolean</code> | <code>false</code> | Enable / disable CSS optimization through SystemJS' CSS plugin. The plugin uses `clean-css` in the background. |
| [options.tab] | <code>String</code> | <code>4 spaces</code> | What to use as tab when formatting the updated SystemJS configuration. |

<a name="Bundler+bundle"></a>
### bundler.bundle(content, saveAs) ⇒ <code>Promise</code>
Bundles components and 3rd-party packages.

| Param | Type | Description |
| --- | --- | --- |
| content | <code>Object</code> | Bundle content. |
| [content.components] | <code>Array</code> | Which components to bundle (without "components/" prefix and without "/index.js" sufix). |
| [content.packages] | <code>Array</code> | Which packages to bundle. |
| saveAs | <code>String</code> | Name of the resulting bundle (without .js extension). |

<a name="Bundler+bundleComponent"></a>
### bundler.bundleComponent(index) ⇒ <code>Promise</code>
Bundle a specific component.

| Param | Type | Description |
| --- | --- | --- |
| index | <code>String</code> | Path to the index.js file of the component. |

<a name="Bundler+bundleComponents"></a>
### bundler.bundleComponents(componentNames, saveAs) ⇒ <code>Promise</code>
Combine multiple components into one bundle.

| Param | Type | Description |
| --- | --- | --- |
| componentNames | <code>Array</code> | Which components to bundle (without "components/" prefix and without "/index.js" sufix). |
| saveAs | <code>String</code> | Name of the resulting bundle (without .js extension). |

<a name="Bundler+bundlePackage"></a>
### bundler.bundlePackage(packageName) ⇒ <code>Promise</code>
Bundle a certain vendor package.

| Param | Type | Description |
| --- | --- | --- |
| packageName | <code>String</code> | Package name, same as in the SystemJS configuration. |

<a name="Bundler+bundlePackages"></a>
### bundler.bundlePackages(packageNames, saveAs) ⇒ <code>Promise</code>
Combine multiple vendor packages into one bundle.

| Param | Type | Description |
| --- | --- | --- |
| packageNames | <code>Array</code> | Which packages to bundle. |
| saveAs | <code>String</code> | Name of the resulting bundle (without .js extension). |

<a name="Bundler+bundleRemainingComponents"></a>
### bundler.bundleRemainingComponents() ⇒ <code>Promise</code>
Bundles all components which are not yet part of an existing bundle.

<a name="Bundler+bundleRemainingPackages"></a>
### bundler.bundleRemainingPackages() ⇒ <code>Promise</code>
Bundles all vendor packages which are not yet part of an existing bundle.

<a name="Bundler+saveConfig"></a>
### bundler.saveConfig() ⇒ <code>Promise</code>
Saves bundle information to the SystemJS configuration.

## Usage example

```javascript
const Bundler = require('angular-lazy-bundler').Bundler;

const bundler = new Bundler({
    systemJsConfig: 'config/system.js'
});

bundler
    .bundle(
        {
            components: [
                'application',
                'home-state'
            ],
            packages: [
                'angular',
                'angular-resource',
                'angular-sanitize',
                'angular-ui-router',
                'ui-router-extras'
            ]
        },
        'main'
    )
    //bundles the sources of our application per component
    .then(() => bundler.bundleRemainingComponents())
    //creates a custom bundle with all packages required for boostrapping the application
    .then(() => {
        return bundler.bundlePackages(
            [
                'date-picker',
                'moment'
            ],
            'date-picker'
        );
    })
    //bundles the remaining packages individually
    .then(() => bundler.bundleRemainingPackages())
    //updates our SystemJS configuration
    .then(() => bundler.saveConfig())
    //here we can handle errors
    .catch((err) => { });
```

## Troubleshooting

A lot of packages don't declare their dependencies properly. For example, UI Router doesn't declare a dependency to Angular in it's package.json. Same with Bootstrap, which doesn't have a dependency to jQuery. This leads to errors when bundling such libraries as their dependencies don't get included properly. If you encounter such issues search for special distribution build of the package on it's GitHub page, e.g. [github.com/angular/bower-angular-animate](https://github.com/angular/bower-angular-animate) for angular-animate. In that case try installing the package from there.

Another possibility is to amend the missing information in `config/system.js` as already done by JSPM when it finds dependency declarations in package.json or in the JSPM registry itself.
