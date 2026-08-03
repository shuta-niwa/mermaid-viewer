# mermaid-viewer

単一HTMLファイル（index.html）で完結するMermaid図のビューア。GitHub Pagesで配信しており、
mainブランチにpushすると即座に公開反映される。

## バージョン管理

`index.html` 内の `const APP_VERSION = 'x.xx'`（TEMPLATES定義の直前）が唯一のバージョン表記源。
title・ヘッダーロゴの表示はどちらもここから生成される（直書き禁止）。

**実装に手を入れたら、コミットのたびにこの値をインクリメントすること。**

## 仕様メモ

### キーボードショートカット（修飾キー対応）
`window` の keydown ハンドラで `+`/`-`/`f`/`w`/`1`/`0` を単独キーのショートカットとして
割り当てている。Ctrl/Cmd/Alt併用時（Ctrl+Fでのブラウザ検索など）はハンドラ内で早期returnし、
ブラウザ標準の動作を優先する。

### コード選択→プレビューのパン追従
コードエディタ（`#editor`）でテキストを選択すると、選択文字列と一致するプレビューSVG内の
テキスト要素（`text`/`tspan`/`foreignObject`内の葉要素）を探し、ズーム倍率は変えずに
その要素が画面中央に来るようパン位置（tx, ty）を更新する（`focusPreviewOnSelection`）。
完全一致を優先し、なければ部分一致にフォールバック。ヒットしない場合は何もしない。

選択範囲が前回と同じ場合は何もしない（`lastFocusedSelection`でガード）。
中ボタンドラッグでプレビューを手動パンした後、同一選択のまま`mouseup`/`select`が
再発火しても手動パンの結果を上書きしないようにするため。

### PNG変換前のvoid要素補正
ノードラベルに`<br>`（改行）を含めると、mermaidが出力するSVGのforeignObject内HTMLに
自己終了していない`<br>`が残る。これを`image/svg+xml`として厳格なXMLパースにかけると
タグ不整合でパースエラーになり、PNGコピー/保存が「SVGをImageに読み込めませんでした」で
失敗する。`svgToPngBlob`に渡す前に`closeVoidTags()`でHTMLのvoid要素を自己終了形式
（`<br/>`等）に補正してから`DOMParser`に渡す。

### 日本語ラベルの折り返し（themeCSS）
mermaidのラベルは`white-space: nowrap`かつ幅上限`wrappingWidth`（既定200px）を持ち、
mermaid自身の折り返しは空白区切り前提のため、空白のない日本語の長文は改行できず
foreignObjectの右端で切れる。`LABEL_WRAP_CSS`（`white-space: normal` /
`word-break: break-word`）を`mermaid.initialize`の**`themeCSS`**として渡して解決している。

`themeCSS`である点が重要で、mermaidがこのCSSをSVG内の`<style>`に埋め込むため
①ノードサイズ計測時（描画時）②PNG化時（ページCSSが効かない単体SVGコンテキスト）
の両方で折り返しが有効になる。ページ側の`<style>`に書くと①だけ効いてPNGで切れる。
