---
title: "ITFFF(Pro以上）を使ってTwitter(現X)へのポストをNostrにクロス投稿する方法"
categories: 
  - "アプリ・ソフト"
  - "webサービス"
  - "アプリ紹介"
slug: "Hot-to-CrossPost-from-Twitter-to-Nostr"
status: publish
---
　どうもこんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。
　今日はかなりマニアックな話なのですが、IFTTTのPro以上のプランを使っている人対象に、Twitterへのポストを自動でNostrというSNSにクロスポストする方法をガイドして行こうと思います。
　今回は対象のSNSがNostrというかなりニッチなSNSである事、そして自走でクロスポストするためにWebhookという仕組みが必要なんですが、私が探した範囲内では良いWebhook作成のプログラムがなかったので、自作のWebhook作成のプログラムを利用しています。
　なお、そのプログラム自体は[https://github.com/darktribe/PostToNostrFromAlgia](https://github.com/darktribe/PostToNostrFromAlgia)で公開してあるのですが、README-jp.mdに記載がある通り、このプログラムは[https://github.com/mattn/algia](https://github.com/mattn/algia)に存在するmattn氏のAlgiaというプログラムに依存します。
　というか私のプログラムはWenhookとしてサーバー上で待機し、呼び出しに応じてAlgiaを動かして、AlgiaでNostrに投稿するだけのプログラムです。
　mattn氏のリポジトリにWebhookとして動くプログラムが投稿されているのですが、それを使ってもGoogle Cloud Engine上のVMでは上手く動かなかったので作った次第です。
　今回はGoogle Cloud Engineを使ってクロスポストのシステムを組んでいくので、先に[https://techblog.show-ya.blue/2023/11/24/how-to-start-google-cloud-engine-instance/](https://techblog.show-ya.blue/2023/11/24/how-to-start-google-cloud-engine-instance/)の記事を読んでGoogle Cloud EngineでVMを立ち上げておく必要があります。
　今回クロスポストするだけの目的であれば独自ドメインを取る必要はなく、IPアドレスの呼び出しで使う事ができるので独自ドメインとの紐づけは必須ではありません。

## Nostrとは
　Nostrとはマイクロブログ・SNSの一種で、中央集約性を持たない珍しいSNSです。
　Nostrの本体はユーザーの投稿をサーバー間で投げ合う「リレー」という仕組みが基本で、自分がカバーしているリレーに投稿されたポストを読む事ができます。
　ポストはリレー間を渡るので最終的に制御不能な範囲まで拡散しますが、どこで止まるのかは読み切れないので、どこかのリレーに投降したからと言って世界中のNostrユーザーに届くとは限らないので要注意です。
　逆に言うと登録リレーが少ないと集まる情報も少なく、相手に届く範囲も狭く、そうなると面白くないので、いかに多くのリレーに登録しつつスパムの少ないリレーを探すのかが大事です。
　実際にNostrを使いたい場合、iOSならは「Damus」、Androidなら「Amethyst」というアプリを使うのがてっとり早いでしょう。
　ブラウザからアクセスしたい場合、日本からの場合は拡張機能のインストールが必須ですが[Rabbit](https://syusui-s.github.io/rabbit/#/)というWebから見られるNostrクライアントが便利です。
　Nostrで一番注意して欲しいのは、ユーザー認証が「公開鍵」と「秘密鍵」で一括管理される点です。「公開鍵」についてはユーザー特定のために皆が知って良い数値ですが、逆に「秘密鍵」はユーザー個人の特定に使うIDとパスワードを兼ねたもので、絶対に人に知られてはいけません。
　「秘密鍵」については暗号として物凄く強度が高いので絶対に特定の人の秘密鍵は辿れないという前提でシステムが構築されているので、逆に言うと秘密鍵が漏れると全部の情報を抜かれますし、なりすましも完璧に可能になるので手に負えません。
　なので「秘密鍵」については厳重な管理を行い、迂闊なクライアントに入力したり、秘密鍵がクリップボードにコピーされた状態でWebサイトに訪れたりしないで下さい。
　こう書くと恐いんですが、面白いbotが多いのと、わりとのどかなギークさんが多く、結構居心地が良いSNSなので、興味のある方はぜひ試してみて下さい。

## 今回の手順
今回のNostrへのクロスポスト実現のためには、mattn氏のAlgiaを使ってWebhookを実装するので、以下の手順で行います。
・Algiaをインストールするために、先にAlgiaが依存するパッケージ類をインストールする
・Algiaをインストールする
・Algiaでポストができるかを確認する
・「PostToNostrFromAlgia」を作成する
・「PostToNostrFromAlgia」をForever関数で永久起動する
・Webhookが完成する
・IFTTTでNostrにクロスポストするための設定をする
　手順が多いのでメゲますが、一つ一つの手順は簡単ですし、基本はコピペで行えるので安心して下さい。

　では、順を追って行きます。

### Algiaをインストールするために、先にAlgiaが依存するパッケージ類をインストールする
前提となる「Algia」についてはLinux上でNostrのポストを見たり、NostrにポストしたりするCLIアプリで、これはGo言語で書かれています。
　なのでAlgiaをインストールする前にGo言語をインストールする必要があります。
　[https://cloud.google.com/compute?hl=ja](https://cloud.google.com/compute?hl=ja)から自分のGoogle Cloud EngineのVMのSSHに入り、コマンドラインが表示されたら以下コマンドを順に打ちます。

<code>cd ~/</code>
　ディレクトリを規定の位置に移動します。何も表示されません。

<code>sudo apt-get update</code>
　インストールされているパッケージ類を最新に更新します。何行か表示され「Done」で終わります。

<code>sudo apt-get install git</code>
　Gitというファイル管理プログラムをインストールします。何行か表示されます。場合によって何もインストールされない場合があります。

<code>sudo apt-get install pip</code>
　pipというパッケージ管理ソフトをインストールします。途中で「Do you want to continue?」と聞かれますので「Y」と答えて下さい。しばらく経ってコマンド権が戻ってきます。

<code>sudo add-apt-repository ppa:longsleep/golang-backports</code>
　aptというパッケージ管理ソフトのリポジトリにGo言語最新版の情報を追加します。
　結構待たされます。何か聞かれたらエンターを押して下さい。

<code>sudo apt install golang</code>
　Go言語をaptというパッケージ管理ソフトを使ってインストールします。
　Algiaの最新版はGo言語が最新ではないとエラーを吐くのでこの方法を使ってインストールします。

<code>export PATH=$PATH:/usr/local/go/bin</code>
　環境パスを追加します。プログラムの在り処をOSに知らせるコマンドです。コマンドラインには何も表示されません。

<code>export PATH=$PATH:./go/bin/</code>
　環境パスを追加します。プログラムの在り処をOSに知らせるコマンドです。コマンドラインには何も表示されません。

<code>source .profile</code>
　.profileというファイルを読み込み直します。おまじないだと思って下さい。このおまじないを怠けると動かなくなります。

<code>pip install flask</code>
　pipというパッケージ管理ソフトを使って「flask」というシステムを導入します。これは後で使うシステムなのでインストールが必須です。

<code>sudo apt-get install gcc-multilib</code>途中でY
　apt-getというパッケージ管理ソフトを使ってgcc-multilbというシステムを導入します。途中で「Do you want to continue?」と聞かれますので「Y」と答えて下さい。
　ここまででAlgiaが依存するパッケージのインストールが終わりました。

### Algiaをインストールする
　では続いてAlgiaをインストールしていきます。そのまま続けて以下を実行していきます。

<code>go install github.com/mattn/algia@latest</code>
　いよいよGo言語を使ってAlgiaをインストールします。そのコマンドがこれです。
　物凄く待たされますが、特にエラーなくAlgiaがインストールされます。
　もし「unknown toolchain」というエラーが出た場合、使っているGo言語が古いのが原因なので、<code>sudo apt install golang</code>が上手く行っているか確認してみて下さい。
　次にAlgia用に設定ファイルを作ります。

<code>cd ~/</code>
　カレントディレクトリを変更します。

<code>mkdir .config</code>
　「.config」というディレクトリをカレントディレクトリに掘ります。

<code>cd .config</code>
　カレントディレクトリの中にある「.config」に移動します。コマンドラインの表示文字に「.config」が追加されます。

<code>mkdir algia</code>
　現在いる「.config」ディレクトリの中に「algia」フォルダを掘ります。

<code>cd ~/</code>
　カレントディレクトリを変更します。

<code>sudo vi ~/.config/algia/config.json</code>
　viというエディタを使ってAlgiaの設定ファイルを作成します。
　最初何も表示されていないと思うので、半角で「a」キーを押して挿入モードにし（画面下に--INSERT--と表示されるので挿入モードになったとわかります）、以下の内容をコピペして下さい。
　最後の「あなたのNostrの秘密鍵（nsecから始まる文字列）を貼り付ける」の部分だけ、あなたが使っているNostrの秘密鍵に書き換えて使って下さい。
　入力が終わったら、「ESC」キーを押して挿入モードを解除し（画面下の--INSERT--が消えます）、半角で「:w」と入力してリターンしてファイルに書き込み、そのまま半角で「:q」と打ち込んでリターンしてviを終了して下さい。

貼り付ける内容：
<code>{
"relays": {
"wss://relay-jp.nostr.wirednet.jp": {
"read": true,
"write": true,
"search": false
},
"wss://relay.damus.io": {
"read": true,
"write": true,
"search": false
},
"wss://nos.lol": {
"read": true,
"write": true,
"search": false
},
"wss://relay.snort.social": {
"read": true,
"write": true,
"search": false
},
"wss://nostr.h3z.jp": {
"read": true,
"write": true,
"search": false
},
"wss://nostr.holybea.com": {
"read": true,
"write": true,
"search": false
},
"wss://yabu.me": {
"read": true,
"write": true,
"search": false
},
"wss://nostr.holybea.com": {
"read": true,
"write": true,
"search": false
},
"wss://nostr-relay.nokotaro.com": {
"read": true,
"write": true,
"search": false
}
},
"privatekey": "あなたのNostrの秘密鍵（nsecから始まる文字列）を貼り付ける"
}</code>
　これでAlgiaのインストールと設定が終わりました。
　詳しく調べてみるとわかるんですが、Algiaの設定ファイルは本来もっと少なくてシンプルなのですが、Algiaの設定そのものだと登録リレーが少なく日本語情報が少ないので、私の判断で登録リレーを増やしてあります。
　全てwss形式というセキュリティの高いリレーを指定しているので、リレー指定としては悪くないものを選んでいると思います。
　では、インストールと設定が終わったので、テスト投稿に進みましょう。

### Algiaでポストができるかを確認する
　Algiaのインストールと設定が終わったので、テスト投稿をしましょう。
　以下コマンドを実行して下さい。

<code>algia -n “test”</code>
　サーバーによってちょっと待たされますが、何もエラーなくコマンドプロンプトに戻ると思います。
　その後Nostrクライアントをリフレッシュすると自分の発言として「test」と表示されていると思います。
　これでAlgiaのインストールが終わりました。
　次はこのAlgiaを使ってWebhook経由でNostrに投稿するスクリプトを組みます。

### 「PostToNostrFromAlgia」を作成する
　では実際にAlgiaを使ってWebhook経由でNostrに投稿するスクリプトを組みます。
　これは[https://github.com/darktribe/PostToNostrFromAlgia](https://github.com/darktribe/PostToNostrFromAlgia)というURLで「PostToNostr.py」というPythonのスクリプトを配布しているので、それを改造して使っていきます。
　というかコピペします。
　以下のコマンドを実行して下さい。

<code>cd ~/</code>
　ディレクトリを移動します。

<code>mkdir PostToNostr</code>
　カレントディレクトリに「PostToNostr」というディレクトリを掘ります。

<code>cd PostToNostr</code>
　ディレクトリを「PostToNostr」に移動します。コマンドプロンプトに「PostToNostr」と追加されます。

<code>sudo vi PostToNostr.py</code>
　PostToNostr.pyというPythonプログラムを作成します。
　内容としては[https://github.com/darktribe/PostToNostrFromAlgia/blob/main/PostToNostr.py](https://github.com/darktribe/PostToNostrFromAlgia/blob/main/PostToNostr.py)の内容をそのまま貼り付けてオーケーです。
　貼り付け方としては、まず画面になにもないので半角の「a」キーを押して挿入モードにし（画面下に--INSERT--と出るのでモードが確認できます）、コピーしてきたスクリプトをそのまま貼り付けてばオーケーです。
　書き換えるべきポイントがいくつかあり、このWebhookはURLとテストキーとポート番号が一致すると本人と見なしてAlgiaを使ってポストしてしまうので、その三点を書き換える必要があります。

・URL：<code>@app.route('/webhook', methods=['POST'])</code>の「/webhook」部分です。
　これを好きなURLに変えます。このままだと「独自ドメインかIPアドレス:5000/webhook」でアクセスできますが、ここを書き換えれば「独自ドメインかIPアドレス:5000/指定した文字列」でアクセスできるようになります。先頭の文字は必ず「/」にする事、文字化けを避けるためにアルファベット大文字小文字（大文字小文字は区別します）と数字だけで組み合わせたものを使うと良いです。

・テストキー：<code>if idkey != "test-key":</code>の「test-key」部分です。
　これを好きな文字列に変えられます。このままだと投稿の際に「id-key」に「test-key」を指定してWebhookを蹴るとポストするようになるんですが、そのままにしておくとアクセスURLとポートが人にバレた時に利用されてしまう恐れがあります。
　この「id-key」はいわゆるパスワードに当たる文字なので、好きな文字に設定して下さい。文字化けを避けるためにアルファベット大文字小文字（大文字小文字は区別します）と数字だけで組み合わせたものを使うと良いです。

・ポート番号：<code>app.run(host='0.0.0.0', port='5000', debug=False)</code>の「5000」の部分です。
　URLを蹴るには特定のポート番号と呼ばれる「入り口」を指定しないといけないのですが、このスクリプトの場合ポート番号5000を指定しています。
　このポート番号は「独自ドメインかIPアドレス:5000/webhook」と指定してアクセスする事でアクセスできるのですが、ポート番号を5000から変えた場合は独自ドメインかIPアドレス:指定したポート番号/webhook」でアクセスする必要があるようになります。
　ポート番号は使うアプリごとに特定の番号が指定されているのでバッティングするとサーバーが上手く動かない原因になるので、だいたい「1000〜15000」以内の数値で指定するといいでしょう。

　これらを記入して、メモを取った上で「esc」キーを押して挿入モードを解除し、半角で「:w」を打ってリターンして書き込み、その後半角で「:q」を打ってリターンしてコマンドプロンプト画面に戻ってきて下さい。
　今ディレクトリが「PostToNostr」になっているので次コマンドでカレントディレクトリを変更します。
<code>cd ~/</code>

### 「PostToNostrFromAlgia」をForever関数で永久起動する
　では、「PostToNostr」ディレクトリに「PostToNostr.py」というPythonスクリプトができたのですが、これはPython3で指定して実行したら即座にWebhookになり、そこからアクセスを待機して稼働するんですが一つ問題があって、待機している間に何かキーを打ったら終了してしまうのです。
　スクリプトが動いている間は稼働するんですが、せっかくのサーバーなのでWebhookを稼働しながら他も色々したいニーズがありますよね。
　なので、「forever」というコマンドを使って、サーバーが動いている間ずっとスクリプトが動いている状態にしてあげると便利です。
　そこでforever関数をインストールするために依存関係のプログラムをインストールします。
　次の手順を行って下さい。

<code>sudo apt-get install npm</code>
　apt-getというパッケージ管理ソフトを使ってnpmというパッケージ管理ソフトを導入します。
　色々行が表示され、最終的に「Do you want to continue?」と聞かれるので「Y」を押して下さい。
　結構待たされてからコマンドプロンプトに戻ってくるので、そうしたら次のコマンドを入力して下さい。

<code>sudo npm install -g forever</code>
　これも結構待たされるんですが、最終的にはコマンドプロンプトに戻ってきます。エラーやワーニングが結構出るんですが、最終的にはインストールされているので安心して下さい。

### Webhookが完成する
　これでWebhookは完成です。
　実際に永久稼働するために以下のコマンドを入力して下さい。

<code>forever start -c python3 PostToNostr/PostToNostr.py</code>
　これでforever関数を使ってPostToNostr.pyが永久稼働します。
　ワーニングを吐きますが普通に動くので問題ありません。
　ちゃんと動いているかどうか確認したい場合、次のコマンドで確認できます。

<code>forever list</code>
　これで表示される領域に赤文字で「STOPPED」と出ていなければ永久動作しています。

　ここで1点、Google Cloud Engineで設定しなければならない事があり、それはVMのファイアーウォールの設定です。
　私の記事を参考にGoogle Cloud EngineのVMを設定した場合、そのVMはWeb通信にしか対応しておらず、外部からhttpとhttps以外のポートでアクセスしても蹴られてしまうのです。
　なので、先程設定したポートを開けておく必要があります。

　今VMの画面が開いていない人は[https://cloud.google.com/compute?hl=ja](https://cloud.google.com/compute?hl=ja)にアクセスしてログインし、VMに移動して下さい。
　VMに移動したら使っているVMの横にあるハンバーガーメニュー（点が縦に3つ並んでいるメニューです）から「ネットワークの詳細の表示」をクリックして画面を切り替え、その画面の左側に「ファイヤーウォール」の項目があるのでそれをクリックします。
　画面上に「ファイアウォール ルールの作成」の項目があるのでそれをクリックすると「ファイアウォール ルールの作成」の画面が開きます。
　名前を適当に入力し、「ターゲット」は「ネットワーク上のすべてのインスタンス」を選び、「送信元IPv4範囲」には「0.0.0.0/0」を入力し、「プロトコルとポート」の部分、「指定したプロトコルとポート」の「TCP」にチェックを入れて、ポートに先程PostToNostr.pyで書き換えた番号を入力します（書き換えていない場合は5000を入力します）。
　その状態で画面下の「作成」を押せばファイヤーウォールルールが追加されてアクセスできるようになります。
　これでWebhookは完成です。後はIFTTTを使ってクロスポストの準備をするだけです。

### IFTTTでNostrにクロスポストするための設定をする
　あとはIFTTTを使ってクロスポストの準備をするだけです。
　早速TwitterのポストをIFTTTを使ってNostrにクロスポストする設定を行いましょう。
　IFTTTにアクセスして、ProかPro+の契約をした後トップ画面に戻ってきて、画面右上にある「Create」をクリックします。
　そうすると「If This」「Then That」と書かれた画面になるので、「IF This」の横に書かれた「Add」ボタンを押します。
　「Choose Service」という画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Twitter」と入力すると大きなアイコンでXのアイコンに「Twitter」と書かれたパネルが出てくるのでそれを押します。
　「Choose a trigger」という画面になるので、画面左下にある「New tweet by you」をクリックします。
　画面が切り替わり「Complete trigger fields」という画面になるので、「Twitter account」の欄が空欄だと思いますので、その右下にある「Add new account」をクリックすると画面がTwitterの認証画面になるので「連携アプリを認証」をクリックすると画面が切り替わり、「Twitter account」の欄が入力されます。
　それを確認して、「Include」の「retweets」と「@replies」は外れたままの状態で「Create trigger」ボタンをクリックします。
　画面が切り替わり、「IF」が入力された状態で「Then that」のボタンが表示された画面になるので右の「Add」ボタンを押します。
　また画面が切り替わり「Choose a service」の画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Webhook」と入力すると大きなアイコンで「Webhooks」と書かれたパネルが出てくるのでそれを押します。
　切り替わった画面は「Make a web request」という項目しかないのでそれをクリックし、切り替わった画面で次を入力して下さい。

・「URL」：「http://独自ドメインかIPアドレス:ポート番号/指定したURL」を入力します。
　例：http://posttonostr.show-ya.blue:5000/webhook
・「Method」：「POST」を選択します。
・「Content Type」：「application/json」を選択します。
・「body」：<code>{
 "id-key" : "PostToNostr.pyで設定したid-keyの内容",
 "text" : "<<<{{Text}}>>>"
}</code>を入力します。
　そこまで入力したら「Create action」をクリックします。

　切り替わった画面した、「Continue」のボタンがあるので押します。
　更に画面が切り替わり、「Applet Title」の入力欄があるので、好きなアプレット（IFTTTのアプリ）名をつけて入力します。私の場合「Twitter to Nostr」としました。入力したら画面下の「Finish」ボタンを押すと画面が切り替わり、画面中央に「Connected」と書かれた画面が表示されてアプレットが完成します。
　これでTwitterに投げたポストがNostrにも投稿されるようになります。お疲れ様でした。

　ちなみに、1つのTwitter投稿を複数のSNSにまとめてクロスポストしたい場合、アプレットを1個にまとめる方法があります。
　IFTTTのメイン画面から「My Applets」をクリックして、まとめたいアプレットをクリックして切り替わった画面右側にある「Settings」をクリックしてアプレットの詳細画面に入ります。
　「Edit Applet」の画面、下にWebhookの欄がありますが、その下に「＋」ボタンがありますのでそれをクリックし、開いた画面から「Choose a service」の検索欄に「Webhooks」を入力して「Webhooks」を選び、そこから他のSNSへのクロスポストの設定をして「Create action」までやって切り替わった画面で「Update」を選ぶと、「Twitter」からポストすると一連の流れで他のSNSにクロスポストするようになります。
　一連の記事でmstdn.jp/mastodon-japan.net/misskey.io/trpger.usなんかを設定している人はそのまま追加する事でNostrにも追加でクロスポストができるようになるのでおすすめです。

