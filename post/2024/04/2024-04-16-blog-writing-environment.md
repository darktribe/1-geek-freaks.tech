---
title: "Typoraを入れる事でブログライティング環境が整いました。"
date: "2024-04-16T21:45:00"
slug: "new-environment-is-complete-within-typora"
categories: 
  - "Apple"
  - "Mac"
  - "PC系"
  - "Windows"
  - "Linux"
  - "ソフト・サービス"
  - "アプリ紹介"
  - "ガジェット紹介"
status: publish
---

　こんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。
　今日はクロスOSのアプリであるTyporaを導入する事で、今まで探し求めてきた理想のブログライティング環境が完成したと感じたので、今僕が使っているブログ執筆環境についてお話したいと思います。
　ちなみに僕はクロスOS使いなので、ブログ執筆環境についてもWindows/Mac/Linuxがあるんですが、基本的には全部クロスOSのアプリを使っているのでどのOSを使っている人も僕の環境を真似る事ができるので、セットアップは面倒ですが環境としては本気で最高だと思っていますので、ぜひ参考にしていただければと思います。

## まず使用パソコンから

　まずブログを書くのにあたって使っているパソコンからご紹介します。ぶっちゃけブログを書くだけならオーバースペックな代物ばかりなのですが、僕は開発もゲームもしたいですしガジェットは手を抜かずにいいのを買う派閥なのでこういう選択肢になりました。
- Mac：M3 MacBookAir15インチ
  　MacはM3MacBookAir15インチのメモリ16GB・SSD512GBのをこの前購入して徹底的にカスタマイズして使っています。見た目も使い勝手も最高で、本当に完璧です。
- Windows：HP OMEN16
  　Windowsは去年末にブラックフライデーセールでHPのゲーミングパソコンであるOMEN16を購入しました。本来のスペックはCore i7-13700H・メモリ16GB・SSD1TB・RTX4060搭載なんですが、購入して早々にメモリを32GBに換装して使っています。ぶっちゃけゲーム目的と重めの3D目的、そして開発目的があり、まあ値段相応というか、僕の価値基準から見ると「安い」買い物でした。
- Linux：Acer Aspire3 A315-24P-N56Y　Ubuntu22.04LTSを入れています　
  　LinuxについてはもともとAcerのAspire3がJoshinの阪神優勝セールでやすかったので、RyzenのCPUの乗ったパソコンを持っていないな、と思って購入し、その後OMEN16を購入したのでWindowsである必要がなくなったので粛々とUbuntu22.04LTSをインストールして使っています。今月末24.04LTSが出て日本語Remixが出たらOSを入れ替え予定です。

　こんな感じで結構豪華な使用パソコンを使っています。

## 次に入力環境です

　ブログライティングには「とにかく量を書く」ために良いキーボードが必須です。

　今まではMacBookのペチペチしたキーボードで頑張っていたんですが、流石にパソコンを使い分けるにおいて3種類のキーボードに慣れるのは不可能と判断し、外付けのキーボードであるKeychronのK8を使いまわしていたんですが、去年末ちょっと思い立って、「世界最高峰」の名高いHHKBに挑戦しようと思い立ったのです。

　まあこのブログでも書いていますが入手で色々考えたり手はずを整えたりして最終的にはHHKB Studioの日本語配列/墨を入手し、更に「ノートパソコンの上にキーボードを乗せて使う【尊師スタイル】」をするためにFAR EASTさんのタイプスティックスというキーボードブリッジを使ってノートパソコンの上にHHKB Studioをおいて、更にバード電子さんのHHKB Studio専用のパームレストを導入して今タッチタイピングでブログを書くようにしているのです。疲労が段違いです。

　そして、HHKBを使う事でノートPCにあるメディアキーが使えなくなるので、左手デバイスであるXPPenさんのACK05を導入してメディアキーの操作と、あとブラウザのタブ移動やですくとっぷいどうなんかを左手デバイスで操作しています。

　HH Studioは本気で入力フィールが最高で、入力していてテンションがあがりますし、本当に入力速度も出力量も増えているので、かなり高かったんですがパームレスト込みで購入した甲斐があると本気で思っているのです。

## マウスも整えました

　以前KeychronのK8を導入する時に尊師スタイルにするとMacBookのトラックパッドが使えなくなるので、外付け用にApple純正のトラックパッドを購入していたのですが、今回はそれを使わずに、マウスをいいのに変えました。

　そもそも使っていたマウスもLogicoolのMX Anywhere2という決して悪くないマウスだったんですが、HHKB Studioを使う事で「良い道具は生産性を上げる」のを痛感したので、マウスについても業界最高峰の名高いLogicoolのMX Master 3s（8000dpi）を購入して使っているのです。これはカスタマイズ性が高くて本当に便利です。

## では実際のブログを書く環境ですが

　では実際のブログを書く環境ですが、ちょっと複雑なので説明していきます。

### エディタ本体：Typora（タイポラ）

　まずブログ本体はTypora（タイポラ）というクロスOSのマークダウンエディタで書いています。これはマークダウンで書くと装飾された文章が表示されるいわゆるWYSIWYG（いまどきこんな言葉使うか？「見たままを得られる」という意味です）のエディタで、もちろんモードを切り替えてマークダウンでどういうコードを書いているかを見る事もでき、写真の貼り付けなんかも便利でとても使いやすいです。

### ブログのデータ管理：VS Code

　次にブログのデータ管理自体ですが、VS Codeという基本的には開発者さんがプログラムを書くために使うツールを使っています。
　これにプラグインをいれる事で、マークダウンで書いた文章をWordPressに投稿する事ができるようになり、それが非常に便利なのです。
　WordPress自体は結構良いブログツールなんですが、いかんせんブラウザ上で動くものなので操作ミスでデータが消えたりする恐れがあるのと、結局WordPressはプレビューを見ないとどんな見栄えになるかわからないので、TyporaとVS Codeを使って管理した方が僕としては楽なんですよね。

### データの同期：GitHub DeskTop

　そしてこれが肝なんですが、ブログのデータはサーバが壊れると消える恐れがあるのでサーバデータはこまめに残しておくんですが、それとは別にローカルとクラウドに記事を保存できると非常に安心です。
　それにはGitHubというサービスを使うと安心で、GitHubに非公開のリポジトリ（入れ物）を作ってそこに記事データを保存し、それをGitHub DeskTopを利用してMacとWindowsとLinuxでデータを同期する事ができるのです。
　これにより、常に最新の情報を全てのパソコンに置いておくことができ、そしていつでもどのパソコンからでも過去記事を確認して必要があれば修正ができるのです。

## というわけでこういう環境でブログを書いているのですが

　というわけで、好きなパソコンを使ってGitHub Desktopを使ってデータを同期しつつ、VS Codeでファイル管理をしながらVS Codeでヘッダだけ作ったマークダウンをTyporaで開いて編集し、満足行くまでブログを書いた後保存してTyporaを終了、VS Codeに戻ってきてコマンドパレットからWordPressに投稿する、という形で今ブログを書いているのですが、「コレだ」感が非常に強く、なんというか、この環境を作るために今までやってきたんじゃないかと思うくらいです。

　今回紹介したものはVS CodeとGitHub DeskTop以外は全部有料ですし、決して安いものではないので万人におすすめするわけではないんですが、Typoraは3台にインストールできて15ドルですし、僕はそれだけの価値があると思って購入して使っていますので、もしよければなにか一つでも参考になれば嬉しいな、と思います。