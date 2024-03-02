---
title: "【React】react-konvaでSVG画像を描画する"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, konva, svg, reactKonva]
published: true
publication_name: ncdc
---

https://konvajs.org/docs/react/Images.html

konva ドキュメントの `react/images` 項目には SVG 画像の使用方法 については特に記載されていない。

<br>

https://konvajs.org/docs/sandbox/SVG_On_Canvas.html

上記のリンク先で SVG について色々と記載されているが、サンプルコードは HTML なので少々見にくい。

<br>

なので、 React で SVG 画像を描画する方法について、上記リンク先の内容を参考に色々と試していく。

<br>

## はじめに

SVG 描画の検証にあたり以降の記載に出てくる`SvgImage`コンポーネントは、全て以下のような画面に表示されるものとします。

```tsx: SvgKonvaPage.tsx
import { Box } from "@mui/material";
import { Layer, Stage } from "react-konva";
import { SvgImage } from "../components/SvgImage";
import { FC, ReactElement } from "react";

export const SvgKonvaPage: FC = (): ReactElement => {
  const padding = 50;
  const stageWidth = window.innerWidth - padding * 2;
  const stageHeight = window.innerHeight - padding * 2;

  return (
    <Box sx={outerBoxStyles}>
      <Box sx={innerBoxStyles(stageWidth, stageHeight)}>
        <Stage width={stageWidth} height={stageHeight}>
          <Layer>
            <SvgImage /> // ← これ
          </Layer>
        </Stage>
      </Box>
    </Box>
  );
};

const outerBoxStyles = {
  bgcolor: "grey",
  width: "100vw",
  height: "100vh",
  display: "flex",
  justifyContent: "center",
  alignItems: "center",
};

const innerBoxStyles = (width: number, height: number) => ({
  width: width,
  height: height,
  bgcolor: "white",
});
```

また、表示する SVG 画像は、 Figma で適当に作った以下のシンプルな画像とします。

```svg: demo.svg
<svg width="100" height="100" viewBox="0 0 100 100" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect x="0.5" y="0.5" width="99" height="99" fill="white" stroke="black"/>
<rect x="6.5" y="7.5" width="39" height="39" fill="white" stroke="black"/>
<rect x="54.5" y="53.5" width="39" height="39" fill="white" stroke="black"/>
</svg>
```

デモの完成系は以下。

![](https://storage.googleapis.com/zenn-user-upload/0f65529de242-20240225.png)
<br>

# window.Image 　で SVG 画像を描画

> If you take a look into the image tutorial and API docs you will see that you need to use a window.Image instance as the image attribute for Konva.Image. So you need to create and download it manually.

ドキュメントの通り、`window.Image` を使用して画像を描画してみる。
useState と useEffect を使用して安全にレンダリングする。

```tsx: SvgImage.tsx
import { Image } from "react-konva";
import svgImage from "../assets/demo.svg";
import { FC, ReactElement, useEffect, useState } from "react";

export const SvgImage: FC = (): ReactElement | null => {
  // レンダリング初回はnullとして、Layer以下には何も表示しない。
  const [image, setImage] = useState<HTMLImageElement | null>(null);

  useEffect(() => {
    const img = new window.Image();
    // 画像読み込み
    img.src = svgImage;

    // 画像の読み込みが完了するとonloadが発火するので、imageに画像データが入る。→ Layer以下に画像が表示される。
    img.onload = () => setImage(img);
  }, []);

  return image ? <Image image={image} draggable x={40} y={40} /> : null;
};
```

<br>

以下の記載でもいけそうに見えるが、実際には意図しない動作となる。
読み込む画像データが小さくて読み込みが高速で完了する場合は、レンダリング時に画像データが読み込めているので、問題なく描画できているように見える。
が、読み込む画像データが大きくなると、読み込みが完了する前にレンダリングされてしまうため、描画が意図通りにならない可能性あり。（レンダリング時には img が null となるため。）

```tsx
export const SvgImage: FC = (): ReactElement => {
  const img = new window.Image();
  img.src = svgImage;

  // 画像の読み込みに時間がかかる場合、レンダリング時にはimgがnullとなるため、意図しない動作になる可能性あり。
  return <Image image={img} draggable x={40} y={40} />;
};
```

<br>

# useImage で SVG 画像を描画

> Also, you can use the brand new react hook use-image to handle loading your images or you can use the lifecycle methods of React and create your own custom component.

[use-image](https://github.com/konvajs/use-image) を使用して画像を描画してみる。
use-image イメージを使用すると、内部的に読み込み状態を管理してくれるので、スッキリと書ける。
svg 画像に限らずシンプルに画像を描画するだけならこれが一番楽そう。

```tsx: SvgImage.tsx
import { Image } from "react-konva";
import useImage from "use-image";
import svgImage from "../assets/demo.svg";
import { FC, ReactElement } from "react";

export const SvgImage: FC = (): ReactElement | null => {
  const [image] = useImage(svgImage);
  // 必要ならstatusも返せる。
  // const [image, status] = useImage(url);

  return <Image image={image} draggable x={40} y={40}></Image>;
};
```

<br>

# SVG 画像の中身を動的にカスタマイズして描画

同じ SVG 画像でも、中身を色とかを動的に変更して描画したい場合は、画像を読み込む前に少し加工入れてから読み込みを走らせるようにする。

```tsx: SvgImage.tsx
import { Image } from "react-konva";
import { FC, ReactElement, useEffect, useState } from "react";
import { createSvgString } from "../feature/create_svg_string";
import { createSvgUrl } from "../feature/create_svg_url";

export const SvgImage: FC = (): ReactElement | null => {
  const [image, setImage] = useState<HTMLImageElement | null>(null);

  // propsに渡すようにすれば、動的にSVG画像を加工してから描画できる。今回は例なので固定値。
  const svgSetting = {
    boxSize: 120,
    boxFillColor: "#9EB2B2",
    leftBoxSize: 15,
    leftBoxFillColor: "blue",
    rightBoxSize: 25,
    rightBoxFillColor: "red",
  };

  useEffect(() => {
    // SVG文字列を作成
    const svgString = createSvgString(svgSetting);

    // SVG文字列を画像URLに変換
    const imageSrc = createSvgUrl(svgString);

    const img = new window.Image();
    img.src = imageSrc;

    img.onload = () => {
      setImage(img);
    };
  }, []);

  return image ? <Image image={image} draggable x={40} y={40} /> : null;
};
```

```ts: create_svg_string.ts
type Props = {
  boxSize?: number;
  boxFillColor?: string;
  leftBoxSize?: number;
  leftBoxFillColor?: string;
  rightBoxSize?: number;
  rightBoxFillColor?: string;
};

// 描画したいsvg画像の中身をコピー。動的に変更したい部分だけpropsで受け取るように適宜加工する。
// サイズを変更しても表示が崩れないように上手いこと調整する必要はあり。
export const createSvgString = ({
  boxSize = 100,
  boxFillColor = "white",
  leftBoxSize = 40,
  leftBoxFillColor = "white",
  rightBoxSize = 40,
  rightBoxFillColor = "white",
}: Props): string => {
  return `<svg
    width="${boxSize}"
    height="${boxSize}"
    viewBox="0 0 ${boxSize} ${boxSize}"
    fill="none"
    xmlns="http://www.w3.org/2000/svg"
  >
    <rect x="0.5" y="0.5" width="${boxSize - 1}" height="${
    boxSize - 1
  }" fill="${boxFillColor}" stroke="black" />

    <rect x="6.5" y="7.5" width="${leftBoxSize}" height="${leftBoxSize}" fill="${leftBoxFillColor}" stroke="black" />

    <rect x="54.5" y="53.5" width="${rightBoxSize}" height="${rightBoxSize}" fill="${rightBoxFillColor}" stroke="black" />
  </svg>`;
};
```

```ts: create_svg_url.ts
export const createSvgUrl = (svgString: string): string => {
  // NOTE: escape関数やunescape関数は非推奨なので注意。
  const encodedSvgString = encodeURIComponent(svgString);
  const svgBase64 = window.btoa(decodeURIComponent(encodedSvgString));

  const imageSrc = `data:image/svg+xml;base64,${svgBase64}`;
  return imageSrc;
};
```

以下、動的に SVG 画像の中身を変更して描画した例。
（※ レイアウトを崩さないように拡大縮小する場合は調整は必要。今回は例なのでレイアウト崩れは無視します。。）

|                                   デフォルト                                   |                                      例 1                                      |                                      例 2                                      |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/f7f763d7e290-20240226.png) | ![](https://storage.googleapis.com/zenn-user-upload/103bdf1f01d0-20240226.png) | ![](https://storage.googleapis.com/zenn-user-upload/b9085503c0b5-20240226.png) |
|                                                                                |

### 課題

- 複雑な SVG 画像で、色やサイズだけでなく、他の要素も細かく動的に読み込みたい場合にはあまり向いていない（もっとスマートな方法がある）かもしれない。ケースバイケース。
- SVG 画像の中身をコピーして手作業で動的なプロパティ対応に加工する必要があるので、純粋に手間。簡単な SVG 文字列ならいいが。種類が増えると大変。
  - Figma 吐き出し時の設定で、変数に設定できる？
  - 他にもっと手作業が減る方法がないか検証中。。。
- 拡大等しても画像が崩れないように調整する必要あり。

<br>

# 備考

- SVG 画像のパスを直接引数に渡すと上手いこと描画されない。
  フォルダ内の画像を使用する際は、import で画像を読み込んでから useImage や window.Image に渡す。

https://github.com/konvajs/react-konva/issues/510

- Konva（react-konva）の Image コンポーネントの image プロパティに渡せる型は以下。
  上記で記載したサンプルコードは全て`HTMLImageElement`を image プロパティに渡すようにしているので、他の型で image に渡すようにすることも可能。
  ```ts
  type CanvasImageSource =
    | HTMLOrSVGImageElement
    | HTMLVideoElement
    | HTMLCanvasElement
    | ImageBitmap
    | OffscreenCanvas
    | VideoFrame;
  ```
