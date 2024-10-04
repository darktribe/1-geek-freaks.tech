---
title: "ITFFF(Pro以上）を使ってTwitter(現X)へのポストをSSLを使ってNostrにクロス投稿する方法"
categories: 
  - "アプリ・ソフト"
  - "webサービス"
  - "アプリ紹介"
slug: "Hot-to-CrossPost-from-Twitter-to-Nostr-By-SSL"
status: publish
---
　どうもこんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。
　今日「も」かなりマニアックなお話なのですが、基本自分向けのメモとしてお送りします。
　昨日のポストである「ITFFF（Pro以上）を使ってTwitter（現X）へのポストをNostrにクロス投稿する方法」[https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/](https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/)の関連記事というか、この記事の続きです。
　この記事って基本的にはIFTTT（Pro以上）でTwitterに投げたポストを拾って、Google Cloud Engineのヴァーチャルマシン上に作ったWebhookに情報を流す事で、ヴァーチャルマシン上に作ってあるプログラムが動いてNostrに自動でクロスポストを投げてくれる仕組みを作る記事なんですが、一点だけ問題があって、ヴァーチャルマシン上にあるWebhookへの通信がHTTPSではなくHTTPなので通信が暗号化されておらず、途中で情報が抜かれる恐れがあるんですよね。そして書き換えられる恐れもあるんです。
　これを解消するには、手っ取り早く言うとWebhookへの通信をSSLで行うのが一番手っ取り早いです。
　なので、今回は前回記事で作ったWebhookの仕組みを書き換える事でSSLに対応させ、SSLでWebhookに対して通信できるようにする方法のガイドになります。
　ニーズどこにあるねん、という記事なのですが、「それができる」という事に気づいた時点でしない方法が私にはなく、実際に構築してしまったのでその方法をシェアしたいと思います。

## 前提
　この記事の前提として、次の作業を行っている必要があります。
- [Google Cloud Engineを使って無料でVMインスタンスを立ち上げる方法](https://techblog.show-ya.blue/2023/11/24/how-to-start-google-cloud-engine-instance/)の記事に従ってGoogle Cloud Engineでヴァーチャルマシンを立ち上げている事
- SSLを使う都合上、独自ドメインを持っていて、そのDNSをGoogle Cloud EngineのVMに向けている事
- [Google Cloud Engineを使って無料でWordPressを立ち上げる方法](https://techblog.show-ya.blue/2023/11/25/how-to-setup-wordpress-with-google-cloud-engine/)の記事に従ってWordPressの立ち上げまでを行っている事
　（厳密にはWordPressまで立ち上げる必要はないのですが、個別に説明すると大変なので、捨てブログになったとしても立ち上げちゃった方が早いです）
- [ITFFF(Pro以上）を使ってTwitter(現X)へのポストをNostrにクロス投稿する方法](https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/)の記事に従って、最低でも「Algiaのインストール」が終わっている事

　以上の作業を行っている必要があります。一応記事としては「もうNostrへのWebhookが動いている」という前提でお話をしますが、まだWebhookが動いていないならそれでも構いません。

## 手順
　今回の作業は全てすでに立ち上がっているヴァーチャルマシン上で行うので、まず　[https://cloud.google.com/compute?hl=ja](https://cloud.google.com/compute?hl=ja)にアクセスし、必要があればGoogleアカウント名とパスワード、二段階認証で認証を通り「Compute Engine」の画面を開きます。
　画面が開いたら「コンソールへ移動」をクリックすると新しい画面が開き、「VM インスタンス」の画面が表示されるので、使うヴァーチャルマシンの欄にある「SSH」をクリックします。
　すると新しい画面が開いて「承認」という画面が出ますので「Authorize」をクリックするとサーバー画面が開きます。
　普段サーバーをいじっている人ならSSH接続でおなじみの画面ですが、知らない人は知らない画面なので、驚かないで見て下さい。

　ここで実行してメモをして欲しい事があるので、それを書いておきます。
　以下のコマンドを順に実行します。

<code>cd $pwd</code>

<code>cd nginx-prosy/certs</code>

<code>ls</code>

　順に実行し、「ls」を実行するとファイルリストが表示されるのですが、暗めの水色であなたの使っている独自ドメインのディレクトリが表示されていると思います。
　この独自ドメインのディレクトリ名を「ドメイン名」としてメモっておいて下さい。後で使います。
　メモをとったら次のコマンドでディレクトリを戻します。

<code>cd ~/</code>

　ディレクトリが移動するので、次コマンドを打ちます。

<code>sudo su</code>

　このコマンドを打つとコマンドプロンプトが切り替わります。
　切り替わったコマンドプロンプトは「root@ホスト名/home/ユーザー名」という表示になっています。
　この「ユーザー名」を「ユーザー名」としてメモっておいて下さい。後で使います。
　今root権限でサーバーにアクセスしている状態なので、このまま使うと色々危険なので次のコマンドで一回元のユーザー権限に戻します。

<code>exit</code>

　もとのコマンドプロンプトに戻ったと思います。
　これで必要情報が揃ったので、実際にSSLのWebhookを作る作業に入っていきます。

## すでにNostr用のWebhookを動かしている人に必要な作業
　その前に、前の記事ですでにNostr用のWebhookを動かしている人の場合、今回作るWebhookはポートが被るので起動しない結果になるので、今動かしているWebhook用の「PostToNostr.py」を止める必要があります。
　記事のとおりに作った人の場合、「forever」コマンドで永久起動していると思いますので、この永久起動しているプロセスを止める必要があります。
　動かしていない人はこの項目を飛ばして構いません。ただ、以下のコマンドは打っても別に問題が起きるコマンドではないので打ってしまっても構いません。

<code>forever stopall</code>
　このコマンドを打って何か表示が出た場合まだ止まり切っていないので、表示がでなくなるまで同じコマンドを繰り返して下さい。
　表示が出なくなった場合foreverで動いているプロセスはもう止まったので次のステップに進んで問題ないです。

## SSL対応のWebhookを作る方法
　SSL対応のWebhookを作る場合、次の手順を行います。
　今回は「PostToNostr-SSL」というフォルダを掘り、その中に「PostToNostr-SSL.py」というスクリプトを作ります。
　まず次コマンドを打ちます。

<code>mkdir PostToNostr-SSL</code>
　カレントディレクトリに「PostToNostr-SSL」というディレクトリを掘ります。

<code>cd PostToNostr-SSL</code>
　ディレクトリを「PostToNostr-SSL」に移動します。コマンドプロンプトに「PostToNostr-SSL」と追加されます。

<code>sudo vi PostToNostr-SSL.py</code>
　PostToNostr-SSL.pyというPythonプログラムを作成します。
　内容としては[https://github.com/darktribe/PostToNostrFromAlgia-SSL-/blob/main/PostToNostr-SSL.py](https://github.com/darktribe/PostToNostrFromAlgia-SSL-/blob/main/PostToNostr-SSL.py)の内容をそのまま貼り付けてオーケーです。
　貼り付け方としては、まず画面に何もないので半角の「a」キーを押して挿入モードにし（画面下に--INSERT--と出るのでモードが確認できます）、コピーしてきたスクリプトをそのまま貼り付けてばオーケーです。
　書き換えるべきポイントがいくつかあり、このWebhookはURLとテストキーとポート番号が一致すると本人と見なしてAlgiaを使ってポストしてしまうので、以下の点を書き換える必要があります。

- URL:<code>@app.route('/webhook', methods=['POST'])</code>の「Webhook」の部分
　これを好きなURLに変えます。このままだと「独自ドメインかIPアドレス:5000/webhook」でアクセスできますが、ここを書き換えれば「独自ドメインかIPアドレス:5000/指定した文字列」でアクセスできるようになります。先頭の文字は必ず「/」にする事、文字化けを避けるためにアルファベット大文字小文字（大文字小文字は区別します）と数字だけで組み合わせたものを使うと良いです。

- テストキー：<code>if idkey != "test-key":</code>の「test-key」部分
　これを好きな文字列に変えられます。このままだと投稿の際に「id-key」に「test-key」を指定してWebhookを蹴るとポストするようになるんですが、そのままにしておくとアクセスURLとポートが人にバレた時に利用されてしまう恐れがあります。
　この「id-key」はいわゆるパスワードに当たる文字なので、好きな文字に設定して下さい。文字化けを避けるためにアルファベット大文字小文字（大文字小文字は区別します）と数字だけで組み合わせたものを使うと良いです。

- ユーザー名：<code>command_word='/home/[username]/go/bin/algia n \"' + pulltext + '\"'</code>の「[username]」の部分
　これを先程メモした「ユーザー名」に書き換えます。「/home/darktribe/go/bin」のように書き換えられます。
　これは、後でroot権限で実行するのでユーザー名が必要になるので書き換えが必要です。

- 書き換え箇所多数：<code>app.run(host='0.0.0.0', port='5000', debug=False,ssl_context=('/home/[username]/nginx-proxy/certs/[YourDomainName]/fullchain.pem', '/home/[Yusername]/nginx-proxy/certs/[YourDomailName]/key.pem'))</code>の部分
  - 「port='5000'」の「5000」を変えるとポート番号を変えられます。主に1000〜15000の範囲で変えて良いです。
  - 「/home/[username]/nginx-proxy」の「[username]」の部分（2箇所）
 　これを先程メモした「ユーザー名」に書き換えます。「/home/darktribe/nginx-proxy」のように書き換えられます。
  - 「certs/[YourDomailName]/」の「[YourDomainName]」の部分（2箇所）
 　これを先程メモした「ドメイン名」に書き換えます。「certs/gce.show-ya.blue/」のように書き換えられます。

　これらを記入して、「esc」キーを押して挿入モードを解除し、半角で「:w」を打ってリターンして書き込み、その後半角で「:q」を打ってリターンしてコマンドプロンプト画面に戻ってきて下さい。
　これでSSLのWebhookのスクリプトとなる「PostToNostr-SSL.py」というファイルができました。
　では実際に動かしていくためにちょっと設定をします。
　今ディレクトリが「PostToNostr-SSL」になっているのですが、これから作業するにあたって今いるディレクトリにいる方が都合が良いのでディレクトリを戻すコマンドは打ちません。
　次の手順に進んで下さい。


## PostToNostr-SSL.pyを動かす方法
　今作った「PostToNostr-SSL.py」については、SSLで動かさないといけないのでroot権限で動かさなければなりません。
　root権限で動かすには一回rootになる必要がありますので、次コマンドを打ってrootユーザーにユーザー変更をします。

<code>sudo su</code>

　コマンドプロンプトが書き換わるので、次コマンドを打ちます。
　打つ時に「[username]」を先程メモしたユーザー名に書き換えて下さい。

<code>cp /home/[username]/.config/algia/config.json /root/.config/algia/config.json</code>

　実際のコマンドは「cp /home/darktribe/.config/algia/config.json /root/.config/algia/config.json」のような形になります。
　このコマンドは、ローカルユーザーのディレクトリにあったAlgia用のコンフィグファイルをrootのディレクトリにコピーするコマンドです。
　これでroot権限でも同じ設定でAlgiaを使えるようになるのでWebhookが動くようになります。

　このWebhookを永久実行するには「forever」というコマンドが必要です。SSL対応ではない方のWebhookを作る記事を最後まで実行した人はもうインストールされているので無視して構いませんが、その記事を読んでSSL対応でないWebhookを作っていない人は恐らく「forever」コマンドがインストールされていないので、[https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/#outline_1__2_5](https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/#outline_1__2_5)の項目だけ読んで「forever」をインストールして下さい。
　コマンド2つでできるのでそんなに難しくないと思います。

注記：root権限でもflaskがインストールされていないとflaskは動かないので、root権限でflaskをインストールしていない人はここで次のコマンドを打ってforeverをrootにもインストールしておいて下さい。
　わからない人は、二重にインストールしても何も問題ないコマンドなので次のコマンドをおまじないとして入力しておいて下さい。

<code>pip install flask</code>

　flaskとforeverがインストールされているなら、ついにSSL対応版のWebhookが動くようになります。次のコマンドを打って下さい。

<code>sudo forever start -c python3 PostToNostr-SSL.py</code>

　ちゃんと稼働しているかどうか不安な人は次コマンドを行って見て下さい。全く表示なしか、あるいは赤文字で「STOPPED」と出ていなければ永久稼働しています。

<code>forever list</code>

　動作を確認した場合も、していない場合も、root権限で居続けるのは危険なので以下コマンドでroot権限を終了しましょう。

<code>exit</code>

　ついでにディレクトリも移動しましょう。

<code>cd ~/</code>

　そうしたらもうヴァーチャルマシンの画面は閉じて大丈夫です。

　前回のSSL対応でないWebhookを作る記事でGoogle Cloud Engineのファイヤーウォール設定をしている人はポート番号を変えていなければそのままWebhookを使えますが。まだ設定していない人やポート番号を変えた人は[https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/#outline_1__2_6](https://techblog.show-ya.blue/2023/11/25/hot-to-crosspost-from-twitter-to-nostr/#outline_1__2_6)を読んでファイヤーウォールの設定を行って下さい。

## IFTTTでNostrにSSLでクロスポストするための設定をする
　あとはIFTTTを使ってクロスポストの準備をするだけです。
　早速TwitterのポストをIFTTTを使ってSSL経由でNostrにクロスポストする設定を行いましょう。
　IFTTTにアクセスして、ProかPro+の契約をした後トップ画面に戻ってきて、画面右上にある「Create」をクリックします。
　そうすると「If This」「Then That」と書かれた画面になるので、「IF This」の横に書かれた「Add」ボタンを押します。
　「Choose Service」という画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Twitter」と入力すると大きなアイコンでXのアイコンに「Twitter」と書かれたパネルが出てくるのでそれを押します。
　「Choose a trigger」という画面になるので、画面左下にある「New tweet by you」をクリックします。
　画面が切り替わり「Complete trigger fields」という画面になるので、「Twitter account」の欄が空欄だと思いますので、その右下にある「Add new account」をクリックすると画面がTwitterの認証画面になるので「連携アプリを認証」をクリックすると画面が切り替わり、「Twitter account」の欄が入力されます。
　それを確認して、「Include」の「retweets」と「@replies」は外れたままの状態で「Create trigger」ボタンをクリックします。
　画面が切り替わり、「IF」が入力された状態で「Then that」のボタンが表示された画面になるので右の「Add」ボタンを押します。
　また画面が切り替わり「Choose a service」の画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Webhook」と入力すると大きなアイコンで「Webhooks」と書かれたパネルが出てくるのでそれを押します。
　切り替わった画面は「Make a web request」という項目しかないのでそれをクリックし、切り替わった画面で次を入力して下さい。

- 「URL」：「https://独自ドメインかIPアドレス:ポート番号/指定したURL」を入力します。
　例：https://posttonostr.show-ya.blue:5000/webhook
- 「Method」：「POST」を選択します。
- 「Content Type」：「application/json」を選択します。
- 「body」：<code>{
 "id-key" : "PostToNostr-SSL.pyで設定したid-keyの内容",
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


　というわけで、かなり誘導リンクで情報を省いたのですが、それでも約1万文字の長文ブログになってしまいました。
　ですがこれでSSL経由で安心してNostrにクロスポストできるので、ぜひ試してみて下さい！