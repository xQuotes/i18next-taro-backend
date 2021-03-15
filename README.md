# Introduction

This is a i18next cache layer to be used in the browser. It will load and cache resources from localStorage and can be used in combination with the [chained backend](https://github.com/i18next/i18next-chained-backend).

# Getting started

Source can be loaded via [npm](https://www.npmjs.com/package/i18next-localstorage-cache) or [downloaded](https://github.com/i18next/i18next-localStorage-cache/blob/master/i18nextLocalStorageCache.min.js) from this repo.

- If you don't use a module loader it will be added to window.i18nextLocalStorageBackend

```
# npm package
$ npm install i18next-localstorage-backend
```

Create storage.js for Taro

```js
// ./storage.js
import Taro from '@tarojs/taro';
class Storage {
  constructor(props) {}
  setItem(key, value) {
    try {
        return Taro.setStorageSync(key, value);
    } catch (e) {
    // f.log('failed to get value for key "' + key + '" from localStorage.');
    }
    return undefined;
  }
  getItem(key) {
    try {
        return Taro.getStorageSync(key);
    } catch (e) {
    // f.log('failed to get value for key "' + key + '" from localStorage.');
    }
    return undefined;
  }
  removeItem(key) {
    try {
        return Taro.removeStorageSync(key);
    } catch (e) {
    // f.log('failed to get value for key "' + key + '" from localStorage.');
    }
    return undefined;
  }
  clear() {
    return Taro.clearStorageSync(key, value);
  }
}
export default new Storage();
```

Wiring up with the chained backend:

```js
import i18next from 'i18next';
import Backend from 'i18next-chained-backend';
import LocalStorageBackend from 'i18next-localstorage-backend'; // primary use cache
import HttpApi from 'i18next-http-backend'; // fallback http load
import Taro from '@tarojs/taro';
import Storage from './storage'

console.log('i18n');
i18next
  .use(Backend)
  .init({
    fallbackLng: ['cn'],
    preload: ['cn'],
    ns: 'memo',
    defaultNS: 'memo',
    keySeparator: false, // Allow usage of dots in keys
    nsSeparator: false,
    initImmediate: false,
    backend: {
      backends: [
        LocalStorageBackend,  // primary
        HttpApi               // fallback
      ],
      backendOptions: [{
        // prefix for stored languages
        prefix: 'i18next_',

        // expiration
        expirationTime: 7*24*60*60*1000,

        // Version applied to all languages, can be overriden using the option `versions`
        defaultVersion: '0.0.1',

        // language versions
        versions: { en: 'v1.2', cn: 'v1.1' },

        // can be either window.localStorage or window.sessionStorage. Default: window.localStorage
        store: Storage //window.localStorage
      }, {
        // path where resources get loaded from, or a function
        // returning a path:
        // function(lngs, namespaces) { return customPath; }
        // the returned path will interpolate lng, ns if provided like giving a static path
        loadPath: 'http://localhost:3333/locales/{{lng}}/{{ns}}.json', // xhr load path for my own fallback
        // loadPath: '/locales/{{lng}}/{{ns}}.json',

        // parse data after it has been fetched
        // in example use https://www.npmjs.com/package/json5
        // here it removes the letter a from the json (bad idea)
        parse: function(data) { 
          console.log('parse parse', data);
          return data.replace(/a/g, '');
        },

        // path to post missing resources
        addPath: 'locales/add/{{lng}}/{{ns}}',

        // define how to stringify the data when adding missing resources
        stringify: JSON.stringify,

        // your backend server supports multiloading
        // /locales/resources.json?lng=de+en&ns=ns1+ns2
        allowMultiLoading: false, // set loadPath: '/locales/resources.json?lng={{lng}}&ns={{ns}}' to adapt to multiLoading

        multiSeparator: '+',

        // init option for fetch, for example
        requestOptions: {
          mode: 'cors',
          credentials: 'same-origin',
          cache: 'default',
        },

        // define a custom fetch function
        request: async function (options, url, payload, callback) {
          return await Taro.request({
            url,
            ...options,
            fail: (err, res) => {
              return callback(new Error('fetch json fail;', true), {
                code: 400,
                data: null
              })
            },
            success: (res, err) => {
              return callback(null, {
                ...res,
                status: res.statusCode,
              });
              return res
            }
          })
        },
      }]
    }
  }, (err, t) => {
    if (err) return console.error(err)
    // i18next.reloadResources();
  });
```

## Cache Backend Options


```js
{
  // prefix for stored languages
  prefix: 'i18next_res_',

  // expiration
  expirationTime: 7*24*60*60*1000,

  // Version applied to all languages, can be overriden using the option `versions`
  defaultVersion: '',

  // language versions
  versions: {},

  // can be either window.localStorage or window.sessionStorage. Default: window.localStorage
  store: window.localStorage
};
```

- Contrary to cookies behavior, the cache will respect updates to `expirationTime`. If you set 7 days and later update to 10 days, the cache will persist for 10 days

- Passing in a `versions` object (ex.: `versions: { en: 'v1.2', fr: 'v1.1' }`) will give you control over the cache based on translations version. This setting works along `expirationTime`, so a cached translation will still expire even though the version did not change. You can still set `expirationTime` far into the future to avoid this

- Passing in a `defaultVersion` string (ex.: `version: 'v1.2'`) will act as if you applied a version to all languages using `versions` option.


## IMPORTANT ADVICE for the usage in combination with saveMissing/updateMissing

We suggest not to use a caching layer in combination with saveMissing or updateMissing, because it may happen, that the trigger for this is based on stale data.


--------------

<h3 align="center">Gold Sponsors</h3>

<p align="center">
  <a href="https://locize.com/" target="_blank">
    <img src="https://raw.githubusercontent.com/i18next/i18next/master/assets/locize_sponsor_240.gif" width="240px">
  </a>
</p>

---

**localization as a service - locize.com**

Needing a translation management? Want to edit your translations with an InContext Editor? Use the orginal provided to you by the maintainers of i18next!

![locize](https://locize.com/img/ads/github_locize.png)

With using [locize](http://locize.com/?utm_source=react_i18next_readme&utm_medium=github) you directly support the future of i18next and react-i18next.

---
