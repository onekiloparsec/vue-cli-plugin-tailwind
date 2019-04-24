# vue-cli-plugin-tailwind

[Tailwind CSS](https://tailwindcss.com/docs/what-is-tailwind)'s utility classes are minned by [Purgecss](https://www.purgecss.com), saving hundreds of kBs in production builds. [postcss-preset-env](https://preset-env.cssdb.org/features) polyfills modern CSS standards based on your `browserslist` configuration.


## Install

### Tailwind 1.0

```console
vue add @ky-is/tailwind@next
```

When the plugin is updated, you can upgrade your configuration with:
```console
vue invoke @ky-is/tailwind
```

### Tailwind 0.x

See the [`tailwind-0.x` branch](https://github.com/ky-is/vue-cli-plugin-tailwind/tree/tailwind-0.x).


## Usage

Use inline classes, or `@apply`. For example, in `src/components/HelloWorld.vue` of the auto-generated cli app:
```html
<style lang="postcss" scoped>
h1 {
  @apply text-purple-500 flex;
}
</style>
```

Applies scoped, browser-prefixed CSS, while PurgeCSS strips all other unused classes, including the thousands generated by Tailwind.


## Configuration

### `postcss.config.js` Plugins

- `postcss-preset-env`: Defaults to stage 2, as these draft proposals are considered reasonably stable. If you want to enable handy experimental features like nested classes (`a { &:hover: {...} }`), change to `stage: 0`. You can safely delete this plugin from the list if you only write old CSS or use another preprocessor.

- `@fullhuman/postcss-purgecss`: Purgecss removes all CSS classes that it can't find reference to. By default, all Vue and style files in the `src` folder are included. Adjust `content` array if you have CSS classes in other files. Add class names to the `whitelist` array you want to manually prevent PurgeCSS from removing if it thinks they're unused.

### Whitelisting

Any CSS class that isn't used inside your `.html` files in `public/`, or by your `.vue` components (outside of the `<style>` block where they're defined) in `src/` will be stripped by PurgeCSS by default, because with it doesn't have a way to know that those classes are being used out-of-the-box.

### 3rd-party CSS

If you're importing CSS from a 3rd-party library, you'll need to add its source files to PurgeCSS's search paths in `postcss.config.js` via the `content` array so it knows they're in use, or `whitelist` the imported classnames so they're never purged.

As an example, `vue-good-table` requires importing its bundled CSS classes. Since PurgeCSS doesn't see those classes being used anywhere, by default they'll be removed from your production build.

There's two approaches to fix this:
- Add `'./node_modules/vue-good-table/src/**/*.vue'` to the `content` array (so PurgeCSS sees these classes being used)

Or:
- Add `/^vgt-/` to the `whitelistPatterns` array (as this library prefixes its class names with `.vgt-*`)


## Check your production build

The first time you build for production after major changes, it's always a good idea to check the output CSS to ensure PurgeCSS is configured correctly for your project. Look over the CSS files in `dist/`, or spin up your production build on `localhost` with a tool like [serve](https://github.com/zeit/serve):
```console
npm install --global serve
cd yourproject
serve -s dist
```


## Caveats

- Don't reference class names by string concatination, as PurgeCSS cannot find them. **Don't:** `text-${error ? 'red' : 'green'}-600`. **Do:** `error ? 'text-red-600' : 'text-green-600'`.
- By default, any class you declare that matches `.*-move` will be whitelisted and always included in your output CSS. This is required to support `<transition-group>`'s [generated classnames](https://vuejs.org/v2/guide/transitions.html#List-Move-Transitions). You can change `whitelistPatterns` in `postcss.config.js` if you don't want this behavior.
- Any time you're using TailwindCSS and Vue, be careful not to define a `<transition-group>` with `name="cursor"`, as this will generate `.cursor-move` which will inherit [TailwindCSS's cursor class](https://tailwindcss.com/docs/cursor/).
- If you use any custom characters in your css classes beyond `/` and `:` (which are required for TailwindCSS), you need to add them to the default regex pattern `/[A-Za-z0-9-_/:]*[A-Za-z0-9-_/]+/g`.
