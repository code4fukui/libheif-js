# libheif-js

> [`libheif`](https://github.com/strukturag/libheif) の Emscripten ビルドを、Node.js およびブラウザ向けの npm モジュールとして配布するものです。

[
![github actions test][github-actions-test.svg]][github-actions-test.link]
[![jsdelivr][jsdelivr.svg]][jsdelivr.link]
[![npm-downloads][npm-downloads.svg]][npm.link]
[![npm-version][npm-version.svg]][npm.link]

このモジュールは、同梱されている `libheif` のメジャーバージョンおよびマイナーバージョンに準拠しています。パッチバージョンは、本モジュール独自の変更を表します。実際に使用されている `libheif` の正確なバージョンについては、[インストールスクリプト](scripts/install.js)を参照してください。

## インストール

```bash
npm install libheif-js
```

## 使い方

このライブラリは、Node.js、Deno、およびモダンブラウザで使用できます。環境やパフォーマンスのニーズに合わせて、いくつかのバリアントが用意されています。

### バリアントの選択

バージョン1.17以降、複数のバリアントが利用可能になりました：

*   **`libheif-js`** (デフォルト): ピュアJavaScript実装。後方互換性や、追加の `.wasm` ファイルを必要としないシンプルなセットアップに最適です。
*   **`libheif-js/wasm`**: Node.js向けのWebAssemblyバージョン。実行時に `.wasm` バイナリを動的にロードすることで、最高のパフォーマンスを提供します。
*   **`libheif-js/wasm-bundle`**: バンドル済みのWebAssemblyバージョン。`.wasm` バイナリがJavaScriptファイル内に埋め込まれているため、Webpackなどのバンドラーやブラウザ環境で簡単に使用できます。

### Node.js (CommonJS)

```js
// ピュアJavaScript実装
const libheif = require('libheif-js');

// WASMバージョン（.wasmバイナリを動的にロード）
const libheif = require('libheif-js/wasm');

// バンドル済みのWASMバージョン（バンドラー向け）
const libheif = require('libheif-js/wasm-bundle');
```

### ブラウザ & Deno (ES Modules)

バンドル済みのWASMバリアントはES Moduleとして利用可能であり、CDNから直接インポートできます。

```js
import libheif_init from 'https://cdn.jsdelivr.net/npm/libheif-js@1.17.1/libheif-wasm/libheif-bundle.mjs';

const libheif = libheif_init();

// Denoの場合:
// const buffer = await Deno.readFile('./image.heic');

// ブラウザの場合（例: ファイル入力から）:
// const buffer = await file.arrayBuffer();

const decoder = new libheif.HeifDecoder();
const data = decoder.decode(buffer);
```
*注: `@1.17.1` の部分は最新バージョンに置き換えてください。予期せぬ破壊的変更を防ぐため、特定のバージョンに固定することをお勧めします。*

### ブラウザ (`<script>` タグ)

HTMLページに直接ライブラリを読み込むこともできます。これにより、グローバルな `libheif` オブジェクトが利用可能になります。

```html
<!-- ピュアJavaScriptバージョン -->
<script src="https://cdn.jsdelivr.net/npm/libheif-js@1.17.1/libheif/libheif.js"></script>

<!-- バンドル済みのWASMバージョン -->
<script src="https://cdn.jsdelivr.net/npm/libheif-js@1.17.1/libheif-wasm/libheif-bundle.js"></script>

<script>
  // グローバルな `libheif` オブジェクトが利用可能になります。
</script>
```

## 使用例

### 画像のデコード

コア機能は、HEIC/HEIFファイルバッファをRAW画像データにデコードすることです。

```js
const fs = require('fs');
const libheif = require('libheif-js/wasm-bundle');

const fileBuffer = fs.readFileSync('./temp/0002.heic');

const decoder = new libheif.HeifDecoder();
const data = decoder.decode(fileBuffer);
// dataはheicファイル内のすべての画像を保持する配列です

const image = data[0];
const width = image.get_width();
const height = image.get_height();

console.log(`画像の解像度: ${width}x${height}`);
```

### Node.js: PNGへの変換

デコード後、`pngjs` などのライブラリを使用してRAWピクセルデータをPNGファイルに変換できます。

```js
const { PNG } = require('pngjs');

const imageData = await new Promise((resolve, reject) => {
  // RAWピクセルデータはRGBAバッファです
  const outputBuffer = new Uint8ClampedArray(width * height * 4);
  image.display({ data: outputBuffer, width, height }, (displayData) => {
    if (!displayData) {
      return reject(new Error('HEIF処理エラー'));
    }
    resolve(displayData);
  });
});

const png = new PNG({ width: imageData.width, height: imageData.height });
png.data = imageData.data;

const pngBuffer = PNG.sync.write(png);
fs.writeFileSync('output.png', pngBuffer);
```

### ブラウザ: Canvasへの表示

ブラウザでは、デコードされた画像をHTMLの `<canvas>` 要素にレンダリングできます。

```js
const canvas = document.createElement('canvas');
canvas.width = width;
canvas.height = height;
const context = canvas.getContext('2d');
const imageData = context.createImageData(width, height);

await new Promise((resolve, reject) => {
  image.display(imageData, (displayData) => {
    if (!displayData) {
      return reject(new Error('HEIF処理エラー'));
    }
    // imageDataオブジェクトにデコードされたピクセルが格納されます。
    resolve();
  });
});

context.putImageData(imageData, 0, 0);
document.body.appendChild(canvas);
```

## 関連プロジェクト

このモジュールは、低レベルな `libheif` の実装を提供します。より使いやすい機能については、以下のプロジェクトを参照してください：

*   [heic-cli](https://github.com/catdad-experiments/heic-cli) - コマンドラインからheic/heif画像をjpegまたはpngに変換
*   [heic-convert](https://github.com/catdad-experiments/heic-convert) - heic/heif画像をjpegおよびpngに変換
*   [heic-decode](https://github.com/catdad-experiments/heic-decode) - heic画像をRAW画像データにデコード

## ライセンス

このライブラリは、[GNU Lesser General Public License, version 3](LICENSE) の下で配布されています。
