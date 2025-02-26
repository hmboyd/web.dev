---
layout: post
title: PRPLパターンを使用して即時読み込みを適用する
authors:
  - houssein
description: PRPLは、Webページを読み込ませ、インタラクティブにさせ、高速化させるために使用されるパターンを説明する頭字語です。このガイドでは、これらの各手法が連動する仕組みと、個別に使用してパフォーマンス向上を実現する方法について説明します。
date: 2018-11-05
tags:
  - performance
---

PRPLは、Webページを読み込ませ、インタラクティブにさせ、高速化させるために使用されるパターンを説明する頭字語です。

- 最も重要なリソースを**プッシュ** (または**事前読み込み**) します。
- できるだけ早く最初のルートを**レンダリング**します。
- 残りのアセットを**事前にキャッシュに保存**します。
- 他のルートや重要でないアセットを**遅延読み込み**します。

このガイドでは、これらの各手法が連動する仕組みと、個別に使用してパフォーマンス向上を実現する方法について説明します。

## Lighthouseでページを監査する

Lighthouseを実行して、PRPL手法に沿った改善の機会を特定します。

{% Instruction 'devtools-lighthouse', 'ol' %}

1. **[パフォーマンス]** および **[プログレッシブWebアプリ]** チェックボックスを選択します。
2. **[監査の実行]** をクリックして、レポートを生成します。

詳細については、[Lighthouseを使用してパフォーマンス改善の機会を特定する](/discover-performance-opportunities-with-lighthouse)を参照してください。

## 重要なリソースを事前読み込み

Lighthouseは、特定のリソースが解析されて遅れて取得された場合、次の失敗した監査を示します。

{% Img src="image/admin/tgcMfl3HJLmdoERFn7Ji.png", alt="Lighthouse: 主要な要求の監査を事前読み込み", width="745", height="97" %}

[**事前読み込み**](https://developer.mozilla.org/docs/Web/HTML/Preloading_content)は宣言型の取得要求であり、ブラウザにできるだけ早くリソースを要求するように命令します。 HTMLドキュメントの先頭に`<link>`タグと`rel="preload"`を追加して、重要なリソースを事前に読み込みます。

```html
<link rel="preload" as="style" href="css/style.css">
```

`window.onload`イベントを遅らせずに、リソースを早くダウンロードするために、ブラウザはリソースに対してより適切な優先度レベルを設定します。

重要なリソースの事前読み込みの詳細については、[重要なアセットの事前読み込み](/preload-critical-assets)ガイドを参照してください。

## できるだけ早く最初のルートをレンダリングする

サイトが画面にピクセルをレンダリングする瞬間に、[**First Paint**](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics#first_paint_and_first_contentful_paint)を遅らせるリソースがある場合、Lighthouseによって警告が表示されます。

{% Img src="image/admin/gvj0jlCYbMdpLNtHu0Ji.png", alt="レンダリングをブロックするリソースの監査を排除する", width="800", height="111" %}

First Paintを改善するために、Lighthouseは、重要なJavaScriptをインライン化し、[`async`](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/adding-interactivity-with-javascript)を使用してその他の項目を遅延すること、および最も重要なCSSをインライン化することをお勧めします。これにより、レンダリングをブロックするアセットを取得するためのサーバーとの間の往復処理が排除され、パフォーマンスが向上します。ただし、開発の観点からは、インラインコードはメンテナンスが難しく、ブラウザで個別にキャッシュに保存できません。

First Paintを改善する別のアプローチは、ページの初期HTML**をサーバー側でレンダリング**することです。これにより、スクリプトの取得、解析、および実行中に、コンテンツがすぐにユーザーに表示されます。ただし、HTMLファイルのペイロードが大幅に増加する可能性があります。このため、[**Time to Interactive**](/tti/) (アプリケーションがインタラクティブになってユーザー入力に応答までにかかる時間) が長くなる可能性があります。

アプリケーションのFirst Paintを減らすための唯一の正解はありません。利点がアプリケーションのトレードオフを上回る場合にのみ、スタイルのインライン化とサーバーサイドレンダリングを検討してください。これらの概念の詳細については、次のリソースを参照してください。

- [CSS配信を最適化する](https://developers.google.com/speed/docs/insights/OptimizeCSSDelivery)
- [サーバーサイドレンダリングとは何か](https://www.youtube.com/watch?v=GQzn7XRdzxY)

<figure data-float="right">{% Img src="image/admin/xv1f7ZLKeBZD83Wcw6pd.png", alt="サービスワーカーとの間の要求/応答", width="800", height="1224" %}</figure>

## アセットを事前にキャッシュする

**サービスワーカー**はプロキシとして機能し、繰り返しアクセスが発生したときに、サーバーではなく、キャッシュから直接アセットを取得できます。これにより、ユーザーがオフライン時にアプリケーションを使用できるだけでなく、繰り返しアクセスしたときのページの読み込み時間も短縮されます。

複雑なキャッシュ要件がなく、ライブラリで対応できる場合は、サードパーティのライブラリを使用して、サービスワーカーを生成するプロセスを簡素化します。たとえば、[Workbox](/workbox)にはツール群が用意されており、サービスワーカーを作成および管理し、アセットをキャッシュに保存できます。サービスワーカーとオフラインの信頼性の詳細については、信頼性学習パスの[サービスワーカーガイド](/service-workers-cache-storage)を参照してください。

## 遅延読み込み

ネットワーク経由で送信するデータが多すぎると、Lighthouseは失敗した監査を表示します。

{% Img src="image/admin/Ml4hOCqfD4kGWfuKYVTN.png", alt="Lighthouse: 膨大なネットワークペイロード監査があります", width="800", height="99" %}

これにはすべてのアセットタイプが含まれますが、大きなJavaScriptペイロードは、ブラウザで解析およびコンパイルするのに時間がかかるため、特にコストがかかります。Lighthouseは、必要に応じてこれに対する警告も提供します。

{% Img src="image/admin/aKDCV8qv3nuTVFt0Txyj.png", alt="Lighthouse: JavaScript起動時間監査", width="797", height="100" %}

ユーザーが最初にアプリケーションを読み込むときに必要なコードのみを含む小さなJavaScriptペイロードを送信するには、バンドル全体を分割し、オンデマンドでチャンクの[遅延読み込み](/reduce-javascript-payloads-with-code-splitting)を実行します。

バンドルを分割できたら、より重要なチャンクを事前読み込みします ([重要なアセットの事前読み込み](/preload-critical-assets)ガイドを参照)。事前読み込みにより、重要性の高いリソースがブラウザによって先に取得およびダウンロードされることが保証されます。

Lighthouseは、オンデマンドでさまざまなJavaScriptチャンクを分割して読み込むだけでなく、重要ではない画像を遅延読み込みするための監査も提供します。

{% Img src="image/admin/sEgLhoYadRCtKFCYVM1d.png", alt="Lighthouse: オフスクリーン画像の遅延監査", width="800", height="90" %}

Webページで多数の画像を読み込む場合は、ページが読み込まれるときに、スクロールしなければ見えない位置、またはデバイスのビューポートの外にあるすべての画像を遅延します ([lazysizesを使用した画像の遅延読み込み](/use-lazysizes-to-lazyload-images)を参照)。

## 次のステップ

PRPLパターンの背景にあるいくつかの基本的な概念を理解したので、このセクションの次のガイドに進んで詳細を確認してください。すべての手法をまとめて適用する必要はないことを覚えておくことが重要です。次のいずれかを使用すると、パフォーマンスが大幅に向上します。

- 重要なリソースを**プッシュ** (または**事前読み込み**) する。
- できるだけ早く最初のルートを**レンダリング**する。
- 残りのアセットを**事前にキャッシュに保存**する。
- 他のルートや重要でないアセットを**遅延読み込み**する。
