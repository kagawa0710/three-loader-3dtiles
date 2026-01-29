# three-loader-3dtiles
![license](https://img.shields.io/badge/License-Apache%202.0-yellow.svg) [![npm version](https://badge.fury.io/js/three-loader-3dtiles.svg)](https://badge.fury.io/js/three-loader-3dtiles)

[デモ](#デモ) &mdash;
[基本的な使い方](#基本的な使い方) &mdash;
[ロードマップ](#ロードマップ) &mdash;
[コントリビュート](#コントリビュート) &mdash;
[ドキュメント](#ドキュメント) &mdash;
[代替ライブラリ](#代替ライブラリ) &mdash;
[English](README.md)

[Cesium](https://github.com/CesiumGS/3d-tiles) が策定した [OGC 3D Tiles](https://www.ogc.org/standards/3DTiles) 形式を扱うための [Three.js](https://threejs.org/) ローダーモジュールです。現在、以下の2つの主要フォーマットをサポートしています：

1. Batched 3D Model (b3dm) - glTF ベース
2. Point cloud（点群）

内部では [loaders.gl ライブラリ](https://github.com/visgl/loaders.gl) を使用しています。loaders.gl は [Urban Computing Foundation](https://uc.foundation/) が管理する [vis.gl プラットフォーム](https://vis.gl/) の一部です。

このフォークは **gawatech** がメンテナンスしており、最新の Three.js および loaders.gl への対応、Google Maps 3D Tiles サポート、実験的な GeoJSON ドレーピング機能を追加しています。

> オリジナルは [The New York Times R&D](https://rd.nytimes.com) チームが開発しました。

---

## デモ
* [RealityCapture で作成したフォトグラメトリ](https://kagawa0710.github.io/three-loader-3dtiles/examples/demos/realitycapture)
* [Cesium ION でホストされた LiDAR 点群](https://kagawa0710.github.io/three-loader-3dtiles/examples/demos/cesium)
* [OpenStreetMap とのマップオーバーレイ](https://kagawa0710.github.io/three-loader-3dtiles/examples/demos/map-overlay)
* [Google Maps Photorealistic 3D Tiles](https://kagawa0710.github.io/three-loader-3dtiles/examples/demos/google-3dtiles)
* [Google 3D Tiles + GeoJSON ドレーピング（実験的）](https://kagawa0710.github.io/three-loader-3dtiles/examples/demos/google-geojson)

---

## 基本的な使い方
`Loader3DTiles` モジュールを使用して `tileset.json` を読み込む簡単な例です。

```javascript
import {
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  Clock
} from 'three'
import { Loader3DTiles } from 'three-loader-3dtiles';

const scene = new Scene()
const camera = new PerspectiveCamera()
const renderer = new WebGLRenderer()
const clock = new Clock()

renderer.setSize(window.innerWidth, window.innerHeight)
document.body.appendChild(renderer.domElement)

let tilesRuntime = null;

async function loadTileset() {
  const result = await Loader3DTiles.load(
      url: 'https://<TILESET URL>/tileset.json',
      viewport: {
        width: window.innerWidth,
        height: window.innerHeight,
        devicePixelRatio: window.devicePixelRatio
      }
      options: {
        dracoDecoderPath: 'https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/libs/draco',
        basisTranscoderPath: 'https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/libs/basis',
      }
  )
  const {model, runtime} = result
  tilesRuntime = runtime
  scene.add(model)
}

function render() {
  const dt = clock.getDelta()
  if (tilesRuntime) {
    tilesRuntime.update(dt, window.innerHeight, camera)
  }
  renderer.render(scene, camera)
  window.requestAnimationFrame(render)
}

loadTileset()
render()
```

---

## インストール

このライブラリは [three.js](https://threejs.org/) r160 をサポートし、GLTF、Draco、KTX2/Basis ローダーを使用します。
対象ブラウザは [package.json](./package.json) の `browserslist` フィールドを参照してください。

### 1. ES Module
`importmap` を使用して npm から依存関係をインポートします。完全な例は[こちら](examples/installation/es-module)を参照してください。

#### **`index.html`**
```html
<script type="importmap">
  {
    "imports": {
      "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
      "three/examples/jsm/": "https://unpkg.com/three@0.160.0/examples/jsm/",
      "three-loader-3dtiles" : "https://unpkg.com/three-loader-3dtiles/dist/lib/three-loader-3dtiles.js"
    }
  }
</script>
<script src='index.js' type='module'>
```

#### **`index.js`**
```javascript
import { Scene, PerspectiveCamera } from 'three';
import { Loader3DTiles } from 'three-loader-3dtiles';
```

### 2. NPM
Webpack / Vite / Rollup などのビルドシステムを使用する場合は、npm からインストールしてください：
```
npm install -s three three-loader-3dtiles
```

完全な webpack の例は[こちら](examples/installation/webpack)を参照してください。

### 3. A-Frame
A-Frame コンポーネント: [aframe-loader-3dtiles-component](https://github.com/nytimes/aframe-loader-3dtiles-component)（オリジナル NYTimes リポジトリ）

### 4. React-Three-Fiber
[examples/r3f](examples/r3f) を参照してください。

---

## ロードマップ

### 3D Tiles Next のサポート
[3D Tiles Next 仕様](https://cesium.com/blog/2021/11/10/introducing-3d-tiles-next/)の一部機能は既に loaders.gl でサポートされています。新しい拡張機能のサポートにより、新たなアプリケーションの可能性が広がります。

### Skip-traversal
[Skip traversal メカニズム](https://cesium.com/blog/2017/05/05/skipping-levels-of-detail/)の実装により、b3dm（メッシュ）タイルのパフォーマンスが大幅に向上する可能性があります。シェーダー/ステンシルバッファベースの実装が必要で、貢献を歓迎します。

---

## コントリビュート

一般的なコントリビューション手順は [CONTRIBUTING.MD](./CONTRIBUTING.md) を参照してください。

### 開発
ライブラリは Vite でビルドされています。シンプルな開発サーバーを起動するには：
```
npm run dev
```

### ビルド
ライブラリをビルドするには：
```
npm run build
```

本番用の minified バージョンをビルドするには：
```
npm run build:production
```

API ドキュメントをビルドするには：
```
npm run docs
```

### テスト
[./test](./test) にテストスペックがあります。実行するには：
```
npm run test
```

---

## ドキュメント
* API ドキュメントは[こちら](docs/three-loader-3dtiles.md)
* デモのコードは [examples/demos](https://github.com/kagawa0710/three-loader-3dtiles/tree/dev/examples/demos) にあります

---

## 代替ライブラリ
Three.js 用の 3D Tiles 実装は他にもあります：

- [NASA-AMMOS / 3DTilesRendererJS](https://github.com/NASA-AMMOS/3DTilesRendererJS)
- [ebeaufay / 3DTilesViewer](https://github.com/ebeaufay/3DTilesViewer)
- [iTowns](https://github.com/iTowns/itowns)

---

## クレジット

これは [nytimes/three-loader-3dtiles](https://github.com/nytimes/three-loader-3dtiles) のフォークです。オリジナルは The New York Times の Research & Development チームが開発しました。詳細は [rd.nytimes.com](https://rd.nytimes.com) をご覧ください。

このフォークは **gawatech** がメンテナンスしており、そのままの状態でご利用いただけます。
