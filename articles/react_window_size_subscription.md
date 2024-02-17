---
title: "【React】ブラウザのウィンドウサイズを自動検知する"
emoji: "📐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, typescript, frontend, hooks, useSyncExternalStore]
published: true
publication_name: ncdc
---

# はじめに

以降にカスタムフックのコード例を記載しますが、全て以下の`create-react-app`で作成された初期の`App.tsx`ベースのコードで動きます。
どのカスタムフックのパターンでも呼び出し側のコードは同じで、同じ動作となります。

```tsx: App.tsx
import "./App.css";
import { useWindowSize } from "./hooks/useWindowSize";

function App() {
  const { windowWidth, windowHeight } = useWindowSize();
  // const [windowWidth, windowHeight] = useWindowSize(); // 配列で返すパターンの場合


  return (
    <div className="App">
      <header className="App-header">
        <p>windowWidth: {windowWidth}</p>
        <p>windowHeight: {windowHeight}</p>
      </header>
    </div>
  );
}
export default App;
```

![](https://storage.googleapis.com/zenn-user-upload/24c3093a3da7-20240217.gif)

<br>

# React18 以前

実装するには`useState`と、`useEffect`（`useLayoutEffect`）を組み合わせて実装する方法がメジャーかなと思います。

以下にサンプルで 3 つのパターンを記載します。
やってることはほぼ一緒なので、好みやプロジェクトの方針に沿ったものを使用してやってください。

```ts: useWindowSize.ts（パターン1）
type WindowSize = {
  windowWidth: number;
  windowHeight: number;
};

export const useWindowSize = (): WindowSize => {
  const [windowSize, setWindowSize] = useState({
    windowWidth: window.innerWidth,
    windowHeight: window.innerHeight,
  });

  useEffect(() => {
    const resize = () => {
      setWindowSize({
        windowWidth: window.innerWidth,
        windowHeight: window.innerHeight,
      });
    };

    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  }, []);

  return windowSize;
};
```

```ts: useWindowSize.ts（パターン2）
export const useWindowSize2 = (): number[] => {
  const [windowSize, setWindowSize] = useState([0, 0]);

  useEffect(() => {
    const resize = (): void => {
      setWindowSize([window.innerWidth, window.innerHeight]);
    };

    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  }, []);

  return windowSize;
};
```

```ts: useWindowSize.ts（パターン3）
export const useWindowSize3 = () => {
  const getWindowSize = () => {
    const { innerWidth: windowWidth, innerHeight: windowHeight } = window;

    return {
      windowWidth,
      windowHeight,
    };
  };

  const [windowSize, setWindowSize] = useState(getWindowSize());

  useEffect(() => {
    const resize = () => {
      setWindowSize(getWindowSize());
    };

    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  }, []);

  return windowSize;
};
```

<br>

`useEffect` と `useLayoutEffect` の違いの詳細はここでは割愛しますので、以下の記事などを参照ください。
（とりあえず`useEffect`で実装しておいて、レンダリング時のちらつきやパフォーマンスに問題がある場合は`useLayoutEffect`を検討するぐらいでいいのでは、と個人的には思ってます。）

https://zenn.dev/syu/articles/6b96e34535b33e#useeffect%E3%81%A8uselayouteffect%E3%81%AE%E4%BD%BF%E3%81%84%E5%88%86%E3%81%91

<br>

# React18 以降

React18 から新たに追加された`useSyncExternalStore`を使うことで、より簡単にウィンドウサイズを検知できるようになりました。
今まで `useState`と`useEffect`などを組みわせて実装していたものを、`useSyncExternalStore`が内部的によしなにやってくれるようになります。

`useSyncExternalStore`のことを簡単に説明すると、**React の状態管理の管轄外（外部）にある状態を React の状態と同期させるためのフック**です。
例えば、ウィンドウサイズ、ネットワーク状態、FireStore との同期、などなど。

詳細は公式ドキュメントを参照してください。

https://ja.react.dev/reference/react/useSyncExternalStore

<br>

```ts: useWindowSize.ts（実装例）
import { useSyncExternalStore } from "react";

type WindowSize = {
  windowWidth: number;
  windowHeight: number;
};

const getWindowWidth = () => {
  return window.innerWidth;
};
const getWindowHeight = () => {
  return window.innerHeight;
};

const subscribeWindowSizeChange = (callback: () => void) => {
  window.addEventListener("resize", callback);
  return () => window.removeEventListener("resize", callback);
};

export const useWindowSizeSyncExternalStore = (): WindowSize => {
  const width = useSyncExternalStore(subscribeWindowSizeChange, getWindowWidth);
  const height = useSyncExternalStore(
    subscribeWindowSizeChange,
    getWindowHeight
  );
  return { windowWidth: width, windowHeight: height };
};
```

<br>

# まとめ

- React18 採用しているプロジェクトなら、`useSyncExternalStore`を使うのが簡単で良さそう。
- React18 未満のプロジェクトなら、`useState`と`useEffect`の組み合わせて実装すれば対応可能。
  `useEffect` か `useLayoutEffect` かは、実装方針や方針に合わせて選択してください。
