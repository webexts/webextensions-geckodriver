### WebExtensions GeckoDriver

When testing [WebExtensions](https://developer.mozilla.org/Add-ons/WebExtensions) you might want to automatically load them into Firefox and do functional testing with [geckodriver](https://github.com/mozilla/geckodriver).

### Installation

```
npm install --save-dev webextensions-geckodriver
```

### Usage

```js
const webExtensionsGeckoDriver = require('webextensions-geckodriver');
const webExtension = await webExtensionsGeckoDriver('/absolute/path/to/manifest.json');
```

Loads the WebExtension as Temporary Add-on into a new Firefox instance. See [API docs](#api) for more details.


### Example

manifest.json includes
```
  "browser_action": {
    "default_title": "Visit Example.com"
  },
  "applications": {
    "gecko": {
      "id": "@examplewebextension",
      "strict_min_version": "57.0"
    }
  }
```

Test could look like this (using `mocha`):
```js
const path = require('path');
const assert = require('assert');

const webExtensionsGeckoDriver = require('webextensions-geckodriver');
const {webdriver} = webExtensionsGeckoDriver;

const manifestPath = path.resolve(path.join(__dirname, './path/to/manifest.json'));

describe('Example', () => {
  let geckodriver;

  before(async () => {
    const webExtension = await webExtensionsGeckoDriver(manifestPath);
    geckodriver = webExtension.geckodriver;
  });

  it('should have a Toolbar Button', async () => {
    const button = await geckodriver.wait(webdriver.until.elementLocated(
      // browser_actions automatically have applications.gecko.id as prefix
      // special chars in the id are replaced with _
      webdriver.By.id('_examplewebextension-browser-action')
    ), 1000);
    assert.equal(await button.getAttribute('tooltiptext'), 'Visit Example.com');
  });

  after(() => {
    geckodriver.quit();
  });
});
```

Full executable example is in the [example directory](example/).


### API

#### Exported default function(path[, options])

* *path* `<string>`, required, absolute path to the `manifest.json` file
* *options* `<object>`, optional
  * *binary* `<string>`, optional, lets you set the `binary` that is passed to [`fx-runner`](https://github.com/mozilla-jetpack/node-fx-runner). Possible values: `firefox`, `beta`, `aurora`, `nightly`, `firefoxdeveloperedition`. Defaults to: `firefox`.
  * *autoInstall*, `<boolean>`, optional, if set to `false` the extension will not be installed, you can manually do so later by calling `install`. Defaults to `true`.
  * *webExt* `<object>`, optional, lets you overwrite the parameters that get passed into [`web-ext.cmd.build`](https://github.com/mozilla/web-ext#using-web-ext-in-nodejs-code)
  * *fxOptions* `firefox.Options`, optional, a [`firefox.Options`](https://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/firefox_exports_Options.html) that will be passed to the webdriver


Returns a Promise that resolves with an initialized `WebExtensionsGeckodriver` instance in case of success, notably with the following properties:

* *geckodriver*, `<object>`, a new [`selenium-webdriver/firefox`](https://www.npmjs.com/package/selenium-webdriver) instance with previously loaded [`geckodriver`](https://www.npmjs.com/package/geckodriver)
* *install*, `<function>`, returns a Promise that resolves when installing is finished, accepts an options `<object>`:
  * *extensionPath*, `<string>`, optional, path to something that [`installAddon`](https://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/firefox_exports_Driver.html) can handle. Defaults to the `web-ext` build extensionPath.
  * *temporary*, `<boolean>`, optional, whether the WebExt should be installed temporary. Defaults to `true`.
* *internalUUID*, `<function>`, returns a Promise that resolves to the `Internal UUID` of the installed extension
* *uninstall*, `<function>`, returns a Promise that resolves when uninstalling is finished, accepts an optional extensions id as `<string>`


#### Exported property: `webdriver`

Return value of [`require('selenium-webdriver')`](https://www.npmjs.com/package/selenium-webdriver)

#### Exported property: `firefox`

Return value of [`require('selenium-webdriver/firefox')`](https://www.npmjs.com/package/selenium-webdriver)


### Travis Configuration

```
dist: xenial
services:
  - xvfb

language: node_js
addons:
  firefox: latest

node_js:
  - 'lts/*'
```

### Headless Example

```js
const webExtensionsGeckoDriver = require('webextensions-geckodriver');

const {firefox} = webExtensionsGeckoDriver;
// or equivalently:
//   const firefox = require('selenium-webdriver/firefox')

const fxOptions = new firefox.Options()
  .headless()
  .windowSize({height: 1080, width: 1920}) // If you rely on viewport size

webExtensionsGeckoDriver(manifestPath, {fxOptions})
```

### JSDOM

If you're looking for a way to test WebExtensions with JSDOM then [`webextensions-jsdom`](https://github.com/webexts/webextensions-jsdom) might be for you.

### Credits

Thanks to [Standard8](https://github.com/Standard8) for the original work in [example-webextension](https://github.com/Standard8/example-webextension).
