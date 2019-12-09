# Bundler Comparison

For the comparison we use a super small project consisting of some dependencies:

- TypeScript
- SASS
- React with React DOM

By default all project folders have thus been set up using the following command:

```sh
npm i react react-dom @types/react @types/react-dom typescript sass @types/node
```

We installed all the typings for these dependencies, too. The requirements for the app bundling are:

- We load one component lazy (bundle splitting)
- We have 1 HTML file
- We have 1 CSS file
- We have 1 image (JPG) file

We should obtain 5 files (1 html, 2 js, 1 css, 1 jpg).

Our key metrics are:

- The number of packages have been installed for bundling
- The ease of setup (how many packages had to be installed manually, how large is the configuration)
- The time of the 1st build and subsequent builds
- How hard is a dev mode to establish
- What is the bundle size (minimum for the side-bundle, full for the main bundle)
- How many changes to our original source code (available in `src` have been necessary)

## Browserify

### Installation

### Setup

### Modifications

### Running

### Results

## Brunch

### Installation

### Setup

### Modifications

### Running

### Results

## FuseBox

### Installation

### Setup

### Modifications

### Running

### Results

## Parcel

### Installation

The installation only requires a single package.

```sh
npm i parcel-bundler --save-dev
```

This results in about 743 new packages.

```plain
+ parcel-bundler@1.12.4
added 743 packages from 533 contributors and audited 8421 packages in 91.127s
found 0 vulnerabilities
```

### Setup

Nothing had to be changed. Everything could run immediately without any configuration.

### Modifications

The original source code was left unchanged.

### Running

Running is done via one command:

```sh
parcel build src/index.html
```

### Results

The initial build took about 7s and resulted in a 132 kB bundle.

```plain
✨  Built in 5.84s.

dist/app.9baf3188.js.map        271.5 KB    125ms
dist/app.9baf3188.js           132.24 KB    5.81s
dist/smiley.b199c00b.jpg        99.38 KB    666ms
dist/Page.cb038bbe.js            2.04 KB    1.29s
dist/Page.cb038bbe.js.map        1.11 KB      5ms
dist/index.html                    374 B    358ms
dist/style.cb6d7d54.css.map        304 B      5ms
dist/style.cb6d7d54.css            130 B    4.95s

real    0m7.293s
user    0m11.828s
sys     0m4.563s
```

The stylesheet was minified to 130 bytes, the additional bundle came at 2 kB.

Subsequent builds took about 2s.

```plain
✨  Built in 734ms.

dist/app.9baf3188.js.map        271.5 KB    152ms
dist/app.9baf3188.js           132.24 KB    478ms
dist/smiley.b199c00b.jpg        99.38 KB     48ms
dist/Page.cb038bbe.js            2.04 KB     25ms
dist/Page.cb038bbe.js.map        1.11 KB     14ms
dist/index.html                    374 B     16ms
dist/style.cb6d7d54.css.map        304 B     14ms
dist/style.cb6d7d54.css            130 B     21ms

real    0m2.201s
user    0m1.359s
sys     0m1.766s
```

## rollup.js

### Installation

The installation requires 8 packages.

```sh
npm i rollup rollup-plugin-typescript2 rollup-plugin-scss @rollup/plugin-html @rollup/plugin-image @rollup/plugin-node-resolve rollup-plugin-commonjs rollup-plugin-terser --save-dev
```

This results in about 229 new packages.

```plain
+ rollup-plugin-commonjs@10.1.0
+ rollup-plugin-terser@5.1.2
+ rollup-plugin-scss@1.0.2
+ rollup@1.27.9
+ rollup-plugin-typescript2@0.25.3
+ @rollup/plugin-node-resolve@6.0.0
+ @rollup/plugin-image@2.0.0
+ @rollup/plugin-html@0.1.0
added 229 packages from 161 contributors and audited 657 packages in 13.473s
found 0 vulnerabilities
```

### Setup

We had to write about a 50 LOC configuration file.

```js
import typescript from 'rollup-plugin-typescript2';
import scss from 'rollup-plugin-scss';
import image from '@rollup/plugin-image';
import html from '@rollup/plugin-html';
import nodeResolve from '@rollup/plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';
import { terser } from 'rollup-plugin-terser';
import { readFileSync } from 'fs';
import { resolve } from 'path';

const content = readFileSync(resolve(__dirname, 'src/index.html'), 'utf8');
const cssName = 'main.css';
const dist = 'dist';

export default {
  input: [resolve(__dirname, 'src/app.tsx'), resolve(__dirname, 'src/style.scss')],
  output: {
    format: 'amd',
    dir: dist,
  },
  plugins: [
    nodeResolve(),
    typescript({
      objectHashIgnoreUnknownHack: true,
    }),
    image({
      dom: true,
    }),
    scss({
      output: `${dist}/${cssName}`,
    }),
    commonjs({
      sourceMap: false,
      namedExports: { react: ['createElement', 'Component', 'lazy', 'Fragment', 'useState'], 'react-dom': ['render'] },
    }),
    terser(),
    html({
      fileName: 'index.html',
      template: ({ files, publicPath }) => {
        const { js = [{ fileName: 'missing.js' }] } = files;
        return content
          .replace(
            '<link rel="stylesheet" href="style.scss">',
            `<link rel="stylesheet" href="${publicPath}${cssName}">`,
          )
          .replace('<script src="app.tsx"></script>', `<script src="${publicPath}${js[0].fileName}"></script>`);
      },
    }),
  ],
};
```

### Modifications

We had to add an additional TypeScript declaration file to import standard images:

```ts
declare module '*.jpg';
```

Furthermore, we are not allowed the mix the classic `require` with `import`. So we had to change the *app.tsx* file.

```diff
import * as React from 'react';
import { render } from 'react-dom';
+import smiley from './smiley.jpg';

const Page = React.lazy(() => import('./Page'));

const App = () => {
  const [showPage, setShowPage] = React.useState(false);

  return (
    <>
      <div className="main-content">
        <h2>Let's talk about smileys</h2>
        <p>More about smileys can be found here ...</p>
        <p>
+          <img src={smiley} alt="A classic smiley" />
-          <img src={require('./smiley.jpg')} alt="A classic smiley" />
        </p>
        <p>
          <button onClick={() => setShowPage(!showPage)}>Toggle page</button>
        </p>
      </div>
      {showPage && <Page />}
    </>
  );
};

render(<App />, document.querySelector('#app'));
```

### Running

Running is done via one command:

```sh
rollup -c rollup.config.js --environment BUILD:production
```

### Results

The initial build took about 7s and resulted in a 132 kB bundle.

```plain

```

The stylesheet was minified to 130 bytes, the additional bundle came at 2 kB.

Subsequent builds took about 2s.

```plain

```

## Webpack

### Installation

The installation requires 9 packages.

```sh
npm install webpack webpack-cli ts-loader sass-loader css-loader html-loader file-loader extract-loader mini-css-extract-plugin --save-dev
```

This results in about 457 new packages.

```plain
+ html-loader@0.5.5
+ extract-loader@3.1.0
+ css-loader@3.2.1
+ mini-css-extract-plugin@0.8.0
+ file-loader@5.0.2
+ sass-loader@8.0.0
+ ts-loader@6.2.1
+ webpack-cli@3.3.10
+ webpack@4.41.2
added 456 packages from 240 contributors and audited 5598 packages in 37.111s
found 0 vulnerabilities
```

### Setup

A new `webpack.config.js` with 40 LOC was required.

```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: ['./src/app.tsx', './src/index.html', './src/style.scss'],
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/i,
        loader: 'file-loader',
      },
      {
        test: /\.html$/,
        use: ['file-loader?name=[name].[ext]', 'extract-loader', 'html-loader'],
      },
      {
        test: /\.s[ac]ss$/i,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader'],
      },
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css',
    }),
  ],
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'index.js',
    path: path.resolve(__dirname, 'dist'),
  },
};

```

### Modifications

In order to have the script and sheet correctly referenced we have to change the `index.html` to the *final* names instead of their entry points.

```diff
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Bundler Comparison</title>
-<link rel="stylesheet" href="style.scss">
+<link rel="stylesheet" href="main.css">
</head>
<body>
<h1>Test Page</h1>
<div id="app"></div>
-<script src="app.tsx"></script>
+<script src="index.js"></script>
</body>
</html>
```

### Running

Running is done via one command:

```sh
webpack --mode production
```

### Results

The initial build took about 5s and resulted in a 130 kB bundle.

```plain
Hash: bdd8a76b5baaf9b51608
Version: webpack 4.41.2
Time: 4178ms
Built at: 12/09/2019 4:51:36 AM
                               Asset       Size  Chunks             Chunk Names
                          1.index.js  386 bytes       1  [emitted]
e69134e93d4d91d02d0b6864a4b9f3a3.jpg   99.4 KiB          [emitted]
                          index.html  366 bytes          [emitted]
                            index.js    130 KiB       0  [emitted]  main
                            main.css   83 bytes       0  [emitted]  main
Entrypoint main = main.css index.js
 [3] multi ./src/app.tsx ./src/index.html ./src/style.scss 52 bytes {0} [built]
 [4] ./src/app.tsx 904 bytes {0} [built]
 [9] ./src/smiley.jpg 80 bytes {0} [built]
[10] ./src/index.html 54 bytes {0} [built]
[11] ./src/style.scss 39 bytes {0} [built]
[12] ./src/Page.tsx 358 bytes {1} [built]
    + 8 hidden modules
Child mini-css-extract-plugin node_modules/css-loader/dist/cjs.js!node_modules/sass-loader/dist/cjs.js!src/style.scss:
    Entrypoint mini-css-extract-plugin = *
    [0] ./node_modules/css-loader/dist/cjs.js!./node_modules/sass-loader/dist/cjs.js!./src/style.scss 220 bytes {0} [built]
        + 1 hidden module

real    0m5.825s
user    0m6.000s
sys     0m1.688s
```

The stylesheet was minified to 83 bytes, the additional bundle came at 386 bytes.

Subsequent builds seem to be slightly faster with 4s.

```plain
Hash: bdd8a76b5baaf9b51608
Version: webpack 4.41.2
Time: 2635ms
Built at: 12/09/2019 4:57:20 AM
                               Asset       Size  Chunks             Chunk Names
                          1.index.js  386 bytes       1  [emitted]
e69134e93d4d91d02d0b6864a4b9f3a3.jpg   99.4 KiB          [emitted]
                          index.html  366 bytes          [emitted]
                            index.js    130 KiB       0  [emitted]  main
                            main.css   83 bytes       0  [emitted]  main
Entrypoint main = main.css index.js
 [3] multi ./src/app.tsx ./src/index.html ./src/style.scss 52 bytes {0} [built]
 [4] ./src/app.tsx 904 bytes {0} [built]
 [9] ./src/smiley.jpg 80 bytes {0} [built]
[10] ./src/index.html 54 bytes {0} [built]
[11] ./src/style.scss 39 bytes {0} [built]
[12] ./src/Page.tsx 358 bytes {1} [built]
    + 8 hidden modules
Child mini-css-extract-plugin node_modules/css-loader/dist/cjs.js!node_modules/sass-loader/dist/cjs.js!src/style.scss:
    Entrypoint mini-css-extract-plugin = *
    [0] ./node_modules/css-loader/dist/cjs.js!./node_modules/sass-loader/dist/cjs.js!./src/style.scss 220 bytes {0} [built]
        + 1 hidden module

real    0m4.310s
user    0m4.141s
sys     0m1.594s
```