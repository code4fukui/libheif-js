# libheif-js

> 日本語のREADMEはこちらです: [README.ja.md](README.ja.md)

> An Emscripten build of [`libheif`](https://github.com/strukturag/libheif) distributed as an npm module for Node.js and the browser.

[
![github actions test][github-actions-test.svg]][github-actions-test.link]
[![jsdelivr][jsdelivr.svg]][jsdelivr.link]
[![npm-downloads][npm-downloads.svg]][npm.link]
[![npm-version][npm-version.svg]][npm.link]

This module respects the major and minor versions of the included `libheif`. The patch version represents changes in this module itself. For the exact version of `libheif` used, please see the [install script](scripts/install.js)
.

## Installation

```bash
npm install libheif-js
```

## Usage

This library can be used in Node.js, Deno, and modern browsers. It comes in several variants to suit different environments and performance needs.

### Choosing a Variant

Starting with version 1.17, there are multiple variants available:

*   **`libheif-js`** (default): A pure JavaScript implementation. Best for backwards compatibility and simple setups where an additional `.wasm` file is undesirable.
*   **`libheif-js/wasm`**: A WebAssembly version for Node.js. It offers the best performance by dynamically loading the `.wasm` binary at runtime.
*   **`libheif-js/wasm-bundle`**: A pre-bundled WebAssembly version. The `.wasm` binary is embedded within the JavaScript file, making it easier to use with bundlers like Webpack or in browser environments.

### Node.js (CommonJS)

```js
// Pure JavaScript implementation
const libheif = require('libheif-js');

// WASM version (dynamically loads .wasm binary)
const libheif = require('libheif-js/wasm');

// Pre-bundled WASM version (easier for bundlers)
const libheif = require('libheif-js/wasm-bundle');
```

### Browser & Deno (ES Modules)

The pre-bundled WASM variant is available as an ES Module, which can be imported directly from a CDN.

```js
import libheif_init from 'https://cdn.jsdelivr.net/npm/libheif-js@1.17.1/libheif-wasm/libheif-bundle.mjs';

const libheif = libheif_init();

// In Deno:
// const buffer = await Deno.readFile('./image.heic');

// In the browser (e.g., from a file input):
// const buffer = await file.arrayBuffer();

const decoder = new libheif.HeifDecoder();
const data = decoder.decode(buffer);
```
*Note: Remember to replace `@1.17.1` with the latest version. Pinning to a specific version is recommended to prevent unexpected breaking changes.*

### Browser (`<script>` tag)

You can include the library directly in an HTML page. This will expose a global `libheif` object.

```html
<!-- Pure JavaScript version -->
<script src="https://cdn.jsdelivr.net/npm/libheif-js@1.17.1/libheif/libheif.js"></script>

<!-- Pre-bundled WASM version -->
<script src="https://cdn.jsdelivr.net/npm/libheif-js@1.17.1/libheif-wasm/libheif-bundle.js"></script>

<script>
  // The `libheif` global is now available.
</script>
```

## Examples

### Decoding an Image

The core functionality is to decode a HEIC/HEIF file buffer into raw image data.

```js
const fs = require('fs');
const libheif = require('libheif-js/wasm-bundle');

const fileBuffer = fs.readFileSync('./temp/0002.heic');

const decoder = new libheif.HeifDecoder();
const data = decoder.decode(fileBuffer);
// data is an array holding all images inside the heic file

const image = data[0];
const width = image.get_width();
const height = image.get_height();

console.log(`Image dimensions: ${width}x${height}`);
```

### Node.js: Convert to PNG

After decoding, you can use a library like `pngjs` to convert the raw pixel data to a PNG file.

```js
const { PNG } = require('pngjs');

const imageData = await new Promise((resolve, reject) => {
  // The raw pixel data is an RGBA buffer
  const outputBuffer = new Uint8ClampedArray(width * height * 4);
  image.display({ data: outputBuffer, width, height }, (displayData) => {
    if (!displayData) {
      return reject(new Error('HEIF processing error'));
    }
    resolve(displayData);
  });
});

const png = new PNG({ width: imageData.width, height: imageData.height });
png.data = imageData.data;

const pngBuffer = PNG.sync.write(png);
fs.writeFileSync('output.png', pngBuffer);
```

### Browser: Display on a Canvas

In a browser, you can render the decoded image onto an HTML `<canvas>` element.

```js
const canvas = document.createElement('canvas');
canvas.width = width;
canvas.height = height;
const context = canvas.getContext('2d');
const imageData = context.createImageData(width, height);

await new Promise((resolve, reject) => {
  image.display(imageData, (displayData) => {
    if (!displayData) {
      return reject(new Error('HEIF processing error'));
    }
    // The imageData object is now populated with the decoded pixels.
    resolve();
  });
});

context.putImageData(imageData, 0, 0);
document.body.appendChild(canvas);
```

## Related

This module provides the low-level `libheif` implementation. For more user-friendly functionality, check out these projects:

*   [heic-cli](https://github.com/catdad-experiments/heic-cli) - convert heic/heif images to jpeg or png from the command line
*   [heic-convert](https://github.com/catdad-experiments/heic-convert) - convert heic/heif images to jpeg and png
*   [heic-decode](https://github.com/catdad-experiments/heic-decode) - decode heic images to raw image data

## License

This library is distributed under the [GNU Lesser General Public License, version 3](LICENSE).