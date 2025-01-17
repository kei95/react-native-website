---
id: performance
title: Performance Overview
---

WebView ベースのツールの代わりに React Native を使う理由の 1 つは、毎秒 60 フレームのネイティブアプリケーションのような見た目と体感を実現することです。
可能であれば React Native が上手く動いてくれて、開発者がパフォーマンス最適化ではなくアプリケーション開発に集中できるようしたいのですが、まだそこには至っておらず React Native 自身では(ネイティブコードを直接記述する場合も同様に)どう最適化するのがよいかを決められないため、あなた自身での作業が必要になります。我々はバッテリー消費の少ない UI を提供するために最善を尽くしていますが、それが難しいこともあります。

このガイドは、パフォーマンスの問題のトラブルシューティングに役立ついくつかの基本事項を説明し、問題の一般的な原因とおすすめの解決策について説明することを目的としています。

## フレームについて知っておくべきこと

あなたの祖父母の世代では、映画は[活動写真](https://www.youtube.com/watch?v=F1i40rnpOsA)と呼んでいました。なぜなら、ビデオのリアルな動きは、静止画を一定の速度ですばやく切り替えるよる錯覚によって生成されるからです。これらの各画像をフレームと呼びます。
毎秒表示されるフレーム数は、ビデオ（またはユーザーインターフェイス）がどれだけスムーズで、究極的には実世界のように見えるかに大きな影響を与えます。
iOS デバイスは毎秒 60 フレームを表示できます。そのため、UI のシステムは 16.67 ミリ秒の間にユーザーへ表示するための静止画(フレーム)を生成する必要があります。
もし割り当てられた 16.67 ミリ秒以内にそのフレームを生成できない場合は、「フレーム落ち」が発生し UI が応答しなくなります。

少し混乱するかもしれませんが、アプリケーションで開発者メニューを開き、`Show Pref Monitor`を選択してください。 2 つの異なるフレームレートがあることに気付くでしょう。

![](/docs/assets/PerfUtil.png)

### JS フレームレート (JavaScript スレッド)

多くの React Native アプリケーションでは、ビジネスロジックを JavaScript スレッドで実行します。
ここに React アプリケーション自体が存在し、API 呼び出し、タッチイベントの処理などが行われます。
ネイティブへのビューの更新は、バッチ処理されてイベントループの終了時、フレームの期限前にネイティブ側へ送信されます（すべてがうまくいく場合）。
JavaScript スレッドがフレームに対して応答しない場合、フレーム落ちと見なされます。
たとえば複雑なアプリケーションのルートコンポーネントで this.setState が呼び出され、計算コストの高いコンポーネントサブツリーが再レンダリングされ、これに 200 ミリ秒かかったとすると 12 フレームがドロップされるでしょう。
JavaScript で制御されているアニメーションは、その間フリーズしているように見えます。何かの表示に 100ms より長くかかる場合、ユーザーはそれに気付くでしょう。

これは `Navigator`の遷移の際によく起こります。
新しいルートをプッシュすると、JavaScript スレッドは適切なコマンドをネイティブ側へ送信し画面に表示するため、シーンに必要なすべてのコンポーネントをレンダリングする必要があります。
遷移は JavaScript スレッドによって制御されるため、ここで行われている作業には数フレームかかり、ジャンクが起きてしまうことがよくあります。
コンポーネントが componentDidMount 内でさらに処理を行う場合もあり、これによって遷移の際にさらにジャンクが発生する可能性もあります。

もう 1 つの例はタッチの反応です。JavaScript スレッドで複数のフレームにわたって処理を行っている場合、たとえば、TouchableOpacity の応答が遅れることに気付くでしょう。
これは、JavaScript スレッドがビジーであり、メインスレッドから送信された生のタッチイベントを処理できないためです。その結果、TouchableOpacity はタッチイベントに反応できず、ネイティブビューに不透明度を調整するように命令できません。

### UI フレームレート (メインスレッド)

多くの人が、 `NavigatorIOS`のパフォーマンスが ` Navigator` よりもだいぶ優れていることに気付いています。これは、画面遷移のアニメーションが完全にメインスレッドで実行されるため、JavaScript スレッドでのフレームドロップに邪魔されないからです。

同様に、JavaScript スレッドがロックされている場合でも、 `ScrollView`はメインスレッド上にあるため、` ScrollView`自体は上下にスクロールできます。スクロールイベントは JS スレッドにディスパッチされますが、スクロールを表示するためにはそれらの受信は必要ありません。

## パフォーマンス問題のよくある原因

### 開発モードで実行している (`dev=true`)

開発モードで実行すると、JavaScript スレッドのパフォーマンスが大幅に低下します。これは避けられません。propTypes やその他のさまざまなアサーションの検証など、適切な警告やエラーメッセージを提供するには、実行時に多くの処理が必要です。パフォーマンスをテストする際は常に[リリースビルド](running-on-device.md#building-your-app-for-production)を使うようにしてください。

### `console.log` を使っている

バンドルされたアプリケーションを実行する場合、これらのステートメントは JavaScript スレッドに大きなボトルネックを引き起こす可能性があります。これは[redux-logger](https://github.com/evgenyrodionov/redux-logger)のようなバンドルライブラリを含みます。なのでバンドルする前にそれらを除くことを忘れないでください。[babel plugin](https://babeljs.io/docs/plugins/transform-remove-console/) を使ってすべての `console.*` 呼び出しを取り除くこともできます。まず `npm i babel-plugin-transform-remove-console --save-dev` をインストールし、プロジェクトディレクトリ配下の `.babelrc` を以下のように変更してください。

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

これでリリース(本番)バージョンのプロジェクトで `console.*` 呼び出しを自動で取り除くことができます。

### リストが長いときに `ListView` の初回レンダリングがおそすぎる, もしくはスクロールのパフォーマンスが悪い

代わりに[`FlatList`](flatlist.md) か [`SectionList`](sectionlist.md) を使ってください。 API がシンプルになるだけでなく、これらのリストコンポーネントではパフォーマンスが大きく改善されています。主な改善は、行数に関わらずメモリ使用量が一定になることです。

もし [`FlatList`](flatlist.md) のレンダリングが遅い場合は、 [`getItemLayout`](flatlist.md#getitemlayout) を実装して、アイテムの測定処理をスキップしてレンダリング速度を最適化したか確認してください。

### ビューがほとんど変わっていないのに JS の FPS が急に落ちる

もし ListView を使っているなら、`rowHasChanged`関数を提供すべきです。これは行を再レンダリングするかしないかを判断するための処理を減らしてくれます。もしイミュータブルな構造を使っているならば、これは参照が等しいことをチェックするだけで済みます。

同様に、 `shouldComponentUpdate`を実装すると、コンポーネントを再レンダリングする条件を詳細に指定できます。純粋なコンポーネントを作成する場合（レンダリング関数の戻り値は完全に props と state に依存します）、PureComponent を利用できます。繰り返しになりますが、これらを高速に行うためイミュータブルなデータ構造が役立ちます。オブジェクトの大きなリストを詳細に比較する必要がある場合は、コンポーネント全体の再レンダリングが速くなり、必要なコードが確実に少なくなる可能性があります。

### JavaScript のスレッドで同時に多くの処理をするせいで JS スレッドの FPS が落ちる

「ナビゲーターの遷移が遅い」はこの最もよくある兆候ですが、他のケースもあります。 InteractionManager を使用することは良いアプローチですが、ユーザーエクスペリエンスのコストが高すぎてアニメーション中の作業を遅らせることをできない場合は、LayoutAnimation を検討することをお勧めします。

Animated API は、[set `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)をしない限り、現状では各キーフレームを JavaScript スレッドで都度計算しています。一方、LayoutAnimation は CoreAnimation を利用しており、JS スレッドとメインスレッドのフレームドロップの影響を受けません。

私がこれを使用した事例の 1 つは、モーダルをアニメーション（下から上にスライドして半透明のオーバーレイでフェードイン）です。初期化して複数のネットワークからのレスポンスを受け取り、コンテンツをレンダリングし、モーダルのビューを更新してモーダルを開く場合です。 LayoutAnimation の使用方法の詳細については、アニメーションガイドを参照してください。

(警告)

- LayoutAnimation は fire-and-forget(静的なアニメーション)でのみ機能します -- 割り込み可能なアニメーションを実装したい場合は `Animated`を使う必要があります。

### スクリーン上でビューを動かすと (scrolling, translating, rotating) UI スレッドの FPS が落ちる

これは、画像の上の透明な背景上にテキストを配置した場合、または各フレームにビューを再描画する度にアルファ合成が必要になる状況で特に当てはまります。 `shouldRasterizeIOS`か ` renderToHardwareTextureAndroid` を有効にすると、これが大幅に改善されます。

これを使いすぎないように注意してください。使いすぎると、メモリ使用量が溢れるでしょう。これらの props を使用するときはパフォーマンスとメモリ使用量を計測してください。ビューを移動させない場合は、このプロパティをオフにしてください。

### 画像サイズをアニメーションさせると UI スレッドの FPS が落ちる

iOS では、画像コンポーネントの幅または高さを調整するたびに、元の画像から再トリミングおよび拡大縮小されます。これは、特に大きな画像の場合、非常にコストがかかるでしょう。代わりに、 `transform：[{scale}]`というスタイルプロパティを使用してサイズをアニメーションしましょう。これを使う一例は、画像をタップして全画面に拡大する場合です。

### TouchableX の反応があまりよくない

タッチに反応してコンポーネントの不透明度またはハイライトを調整させようとするとき、 `onPress`関数が戻るまで、そのエフェクトが画面に表示されないことがあります。 `onPress`が ` setState` を実行したときに多くの処理が発生し、数フレームがドロップされた場合にも発生する可能性があります。これに対する解決策は、 `onPress`ハンドラー内のアクションを `requestAnimationFrame` でラップすることです。

```jsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 画面遷移が遅い

前述のように、 `Navigator`アニメーションは JavaScript スレッドによって制御されます。 右から差し込まれる画面遷移を想像してみてください。画面外（たとえば 320 の x オフセット）で開始し、各フレームで新しい画面が右から左に移動し、画面が最終的に x オフセット 0 に位置すると固定されます。 この移動中の各フレームで、JavaScript スレッドは新しい x オフセットをメインスレッドに送信する必要があります。 JavaScript スレッドがロックされている場合、これを行うことができないため、そのフレームで更新が行われず、アニメーションが途切れます。

これに対する 1 つの解決策は、JavaScript ベースのアニメーションをメインスレッドにオフロードすることです。このアプローチで上記の例と同じことを行う場合、遷移を開始するときに新しい画面のすべての x オフセットのリストを計算し、それらをメインスレッドに送信することで、最適化された方法で実行できます。 JavaScript スレッドがこの責任から解放されるので、画面のレンダリング中に数フレームドロップしても大したことではありません。遷移に気を取られるため、おそらく気付かないでしょう。

これを解決することは、新しい[React Navigation](navigation.md) ライブラリのメインゴールの 1 つです。 React Navigation のビューはネイティブコンポーネントと [`Animated`](animated.md) ライブラリを使い、ネイティブスレッドで 60FPS のアニメーションを実行します。
