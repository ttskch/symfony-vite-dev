# Performances

## Preloading your scripts

When your files contain common dependencies (React, Vue, etc.), Vite will split your files in order to reduce the overall size of your scripts.

The bundle allows you to choose how you will preload your scripts with the `preload` option.

```yaml
# config/packages/pentatrion_vite.yaml
pentatrion_vite:
    preload: link-header
```


with a configuration like this for example:

```js
// vite.config.js
import symfonyPlugin from 'vite-plugin-symfony';
import vuePlugin from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [
    vuePlugin(),
    symfonyPlugin(),
  ],

  build: {
    rollupOptions: {
      input: {
        "app": "./assets/app.js",
      },
      output: {
        manualChunks: {
          vue: ['vue']
        }
      }
    },

  },
});
```

```js
// assets/app.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```
```twig
{{ vite_entry_link_tags('app') }}
{{ vite_entry_script_tags('app') }}
```

The preload option is only effective when you have launched a `vite build`.

This is how your application will behave depending on the `preload` option you choose.


With the `none` option:

```html
<link rel="stylesheet" href="/build/assets/app-05a88f8a.css">
<script type="module" src="/build/assets/app-1b490458.js"></script>
```

With the `link-tag` option (default behavior):

Your JS dependencies are preloaded when the html page is processed.

```html
<link rel="stylesheet" href="/build/assets/app-05a88f8a.css">
<link rel="modulepreload" href="/build/assets/vue-1efeee8e.js">
<script type="module" src="/build/assets/app-1b490458.js"></script>
```

With the `link-header` option:

All your files are preloaded before processing the html page. Requires installing the Symfony component [symfony/web-link](https://github.com/symfony/web-link).

```css
/* HTTP header added by the Symfony Web-Link component */
Link: \
  </build/assets/vue-1efeee8e.js>; rel="preload"; as="script", \
  </build/assets/app-1b490458.js>; rel="preload"; as="script", \
  </build/assets/app-05a88f8a.css>; rel="preload"; as="style"
```
```html
<link rel="stylesheet" href="/build/assets/app-05a88f8a.css">
<script type="module" src="/build/assets/app-1b490458.js"></script>
```


## Caching configuration files

When you call the twig functions `vite_entry_link_tags('app')` or `vite_entry_script_tags('app')` or `asset('assets/image.jpg')`, the bundle will look for `public/build/entrypoints files. json` and `manifest.json` to fill your html tags with the correct `src` or `href` attributes.

If you have specified multiple configurations, even more files will be opened and processed.

When your application is in production you can choose to group these `json` files into a single `php` cache file.

```yaml
# config/packages/pentatrion_vite.yaml
when@prod:
    pentatrion_vite:
        cache: true
```

```bash
# build your js files
# and also generates `public/build/entrypoints.json` and `public/build/manifest.json`
npm run build

# clear the cache and perform a warm-up
# important this step must take place after the `npm run build`
symfony console cache:clear
```

The warm-up step allows the creation of a single PHP file which will be used in place of your `*.json` files.

```php
// var/cache/prod/pentatrion_vite.cache.php
// This file has been auto-generated by the Symfony Cache Component.
return [
  [
    '_default.entrypoints' => 0,
    '_default.manifest' => 1,
  ],
  [
    0 => [
        'base' => '/build/',
        'entryPoints' => [
            'app' => ...,
        ],
        'legacy' => false,
        'metadatas' => [],
        'version' => '5.0.0',
        'viteServer' => null,
    ],
    1 => [
        'assets/app.js' => ...,
    ],
  ]
];
```