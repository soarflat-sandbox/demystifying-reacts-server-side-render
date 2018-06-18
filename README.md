# Demystifying server-side rendering in React

## サーバーサイドレンダリング（SSR）

フロントエンドフレームワークがバックエンドシステムを実行中にマークアップをレンダリングする機能のこと。

サーバーとクライアントの両方でレンダリングする機能を持つアプリケーションは、 **ユニバーサルアプリケーション（Universal App）** と呼ばれる。

**SSR + SPA = Universal App** という認識でも問題ない。

## なぜ SSR を利用するのか（必要なのか）？

SPA の問題点を解消するために利用する。

SPA は最初に大量のリクエスト（CSS、JS、空の HTML や外部ファイルなど）を送るため、ユーザーが最初のレンダリングに長く待たなければならない。また、クローラがページを空であると解釈する可能性もある。

SSR を利用すれば、完全にレンダリングされた HTML が返ってくるため、ユーザーがレンダリングを待つ必要がなくなる。

また、クローラーはサーバー上でレンダリングしたコンテンツのインデックスを作成するため、SEO にも有効である。

そのため、SSR から得られる 主な利点は以下の 2 つ。

- 最初のページレンダリング時間の短縮
- SEO 対策の手段となる

## SSR を試してみる

フロントエンドフレームワークである React を利用して、SSR を試してみる。

### Basic Setup

SSR を利用するにはサーバーが必要なため、簡単な Express アプリケーションを実装する。

**`server.js`**

```js
import express from 'express';
import path from 'path';

import React from 'react';
import { renderToString } from 'react-dom/server';
import Layout from './components/Layout';

const app = express();

// distにあるファイルを静的アセットファイルとして提供する
app.use(express.static(path.resolve(__dirname, '../dist')));

// 全てのGETリクエストに対して、レンダリングされたレスポンスを返す
app.get('/*', (req, res) => {
  const jsx = <Layout />;
  // jsxをHTMLテンプレートに挿入する文字列に変換する
  const reactDom = renderToString(jsx);

  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end(htmlTemplate(reactDom));
});

app.listen(2048);

function htmlTemplate(reactDom) {
  return `
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="utf-8">
            <title>React SSR</title>
        </head>
        
        <body>
            <div id="app">${reactDom}</div>
            <script src="./app.bundle.js"></script>
        </body>
        </html>
    `;
}
```

クライアントには、サーバーレンダリングされた React アプリケーションを使用し、イベントハンドラをアタッチする`ReactDOM.hydrate`メソッドを定義する。

**`client.js`**

```js
import React from 'react';
import ReactDOM from 'react-dom';

import Layout from './components/Layout';

const app = document.getElementById('app');
ReactDOM.hydrate(<Layout />, app);
```

サンプルのフルは[GitHub リポジトリ](https://github.com/alexnm/react-ssr/tree/basic)で確認できる。
