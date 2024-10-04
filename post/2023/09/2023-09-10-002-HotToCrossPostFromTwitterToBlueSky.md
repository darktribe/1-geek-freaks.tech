---
title: "ITFFF(Pro以上）を使ってTwitter(現X)へのポストをBlueSkyにクロス投稿する方法"
categories: 
  - "アプリ・ソフト"
  - "webサービス"
  - "アプリ紹介"
slug: "Hot-To-CrossPost-From-Twitter-To-BlueSky"
status: publish
---
# ITFFF（Pro以上）を使ってTwitter（現X）へのポストをBlueSkyにクロス投稿する方法
　どうもこんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。

　今回の記事は前々回記事：「[ITFFF(Pro以上）を使ってTwitter(現X)へのポストをmastodonにクロス投稿する方法](https://techblog.show-ya.blue/2023/09/08/howtocrosspostfromtwittertomastodon/)」と前回記事[ITFFF(Pro以上）を使ってTwitter(現X)へのポストをMisskeyにクロス投稿する方法](https://techblog.show-ya.blue/2023/09/10/hottocrosspostfromtwittertomisskey/)記事の続きというか、関連記事になります。
　前々回はIFTTTのPro以上のプランを使ってTwitterへの投稿をMastodonサイトにクロスポストするための設定方法、前回はITFFFのPro以上のプランを使ってTwitterへの投稿をMissKeyサイトにクロスポストするための設定方法でしたが、今回は同じくIFTTTのPro以上のプランを使ってTwitterへの投稿をBlueSkyにクロスポストするための設定方法についての記事です。
　BlueSkyはMastodonやMissKeyと違ってWebhookの仕組みが実装されていないので、今回はそのWebhookをCloudFlareというサイトを使ってWebhookを構築してから、そのWebhookへのアクセス設定をIFTTTを使って行うという流れになっているので、前々回の記事や前回の記事よりも難易度が高めになっています。
　こんかい参考にさせていただたサイトについてはリンクを貼っていますが、できるだけ噛み砕いて説明しますので、諦めないで手を動かして着いてきていただければ必ずゴールに辿り着くと思いますのでどうかよろしくお願いします。

## BlueSkyとは
　BlueSkyとは最近流行りのマイクロブログ・SNSの一種類で、Twitter起業者のジャック・ドーシー氏が関与しているSNSとして・今クローズドで運営されていてとても雰囲気の良いマイクロブログとして有名なSNSです。
　iOS、Androidにも専用クライアントがあり、Webからもアクセスでき、また色々な種類のWebクライアントが用意されているのも特徴で、私は[tokimeki.blue](https://tokimeki.blue)を使って楽しんでいます。
　BlueSkyの難点はクローズドなSNSなので、加入するにはBlueSkyの既存ユーザーさんに招待して貰うしか方法がない（一応ウェイトリストはありますが正直期待できません……）点で、私も参加するには結構苦労しました。
　そういう意味で一番の問題というか障壁はBlueSkyのアカウントを作成する点なのですが、逆に言えばそこをクリアしている人であればあとは無料サービスの連携でWebhook作成までは行けますし、IFTTTのPro以上のプランに加入しているのであればWebhookを蹴ってBlueSkyにクロス投稿するのは簡単にできるので、今回の記事の障壁は「BlueSkyのアカウントを用意する」「IFTTTのPro以上に加入する」の2点だけだと言えます。

## IFTTTとは
　今回の記事は前回記事：「[ITFFF(Pro以上）を使ってTwitter(現X)へのポストをmastodonにクロス投稿する方法](https://techblog.show-ya.blue/2023/09/08/howtocrosspostfromtwittertomastodon/)」の記事の続きというか、関連記事になります。
　前回はIFTTTのPro以上のプランを使って、Twitter（現X）にポストを投稿するとマストドン（具体的には[mastdn.jp](https://mstdn.net)と[mastodon-japan.net](https://mastodon-japan.net)）に自動でクロスポストしてくれる方法についてのガイドでしたが、今回は別のSNSである[BlueSky](https://blueskyweb.xyz/)にクロス投稿してくれる仕組みを作るガイドです。
　今回も前回と同じくITFFFのPro以上のプランを使い、Webhookという仕組みを使ってクロス投稿を実現する方法ですので、IFTTではなく他のサービス連携サービスを使っている方は適宜読み替えてご利用下さい。

## IFTTTとは
　[IFTTT](https://ifttt.com)とは、サービス連携サービスの一種です。
　何かのサービスを使った時、それをトリガーにして別のサービスを起動する事ができるサービスで、例えば今回のようにTwitterにポストを投稿した時、それをトリガーにしてBlueSkyに同じ内容のポストを投げる事ができるようになるシステムなのです。
　ITFFFは連携するサービスが800個以上あり、そして老舗で使っている人も多く、公開されているアプレット（ITFFFのWeb系システムの設計書の事です）もかなりの数が公開されているので、Twitterに関連するものは有料でしか使えませんが、それ以外でもかなり使えるサービスがあるのでぜひ試してみると良いと思います。
　今回は手順として
- GitHubのアカウントを作る方法
- CloudFlareのアカウントを作り必要データをコピーする方法
- CloudFlare用のツールwranglerをインストールする方法
- GitHubで必要なリポジトリをテンプレートとしてコピーする方法
- コピーしたGitHubで行う設定
- ローカルにGitHubのデータをクローンしてくる方法
- wranglerでPublishする方法
- wranglerで変数を設定する方法
- Webhookの完成
- IFTTTでBlueSkyにクロスポストするための設定
　の順でガイドします。手順が多いですが順を追って作業すれば確実に成功させる事ができますので、落ち着いて手を動かしていきましょう。

## 参考URL
　今回の記事は以下のウェブ情報を元に作成しています。
　ブログ筆者はMacの環境で試しているので環境としてはMacがメインでご紹介しますが、Linux環境ではほぼそのままの方法で設定ができますし、Windows環境についても以下のウェブ情報を参考にしていただければ不可能ではないと思います。

参考記事
- [blog.bridgey.devの記事：「X (旧Twitter) への投稿を Mastodon と Bluesky にクロスポストする仕組み」](https://blog.bridgey.dev/2023/07/28/crosspost-twitter-mastodon-bluesky/)
- [quiitaの記事：「【超初心者】macOS での Cloudflare Worker 用 wrangler インストール方法」](https://qiita.com/katzueno/items/83d8b8e3f0904194eb8f)
- [Zenn.devの記事：「Windows 環境でCloudflare 開発ツール Wranglerを設定する方法とHello World!の実行まで」](https://zenn.dev/kameoncloud/articles/1fac9762aab4ec)
　とくにWindows環境の方は[Zenn.devの記事：「Windows 環境でCloudflare 開発ツール Wranglerを設定する方法とHello World!の実行まで」](https://zenn.dev/kameoncloud/articles/1fac9762aab4ec)をしっかり読んでから事に及ぶ事をお勧めします。

## CloudFlareとは
　[CloudFlare](https://www.cloudflare.com/ja-jp/)とは本質的に言うと全然違うんですが、物凄くざっくり説明するとWebサーバとの間に入ってデータを処理して別のサーバに届けてくれるサービスで（理解が違ったらすみません）、今回はIFTTTでWebhookを蹴りに行った時、BlueSKyのサーバとの間に入ってWebhookとしてデータと受け取って加工し、それをBlueSkyに送ってポストとして流すための役割をするウェブサービスです。
　このブログでは「[BlueSkyでContrail（とGitHubとCloudFlare）を使ってカスタムフィードを発行する方法](https://techblog.show-ya.blue/2023/08/05/howtomakeblueskycustomfeedwithcontrails/)」という記事でBlueSkyのカスタムフィードを発行するためにCloudFlareを使っているのでご存知の方もいるかも知れませんが、今回も申し込みの手順から説明していきますので、もうCloudFlareのアカウントを持っている方は適宜読み飛ばしてご対応下さい。

## GitHubのアカウントを作る方法
　まずGitHubのアカウントを取得します。アカウントを取得している人はスキップして構いません。
　最初に[https://github.co.jp/](https://github.co.jp/)にアクセスします。
　そうすると画面右上に「サインアップ」というボタンがあるのでそれをクリックします。
　「First, let’s create your user account」という画面になり、「Username」「Email Address」「Password」「Email preferences」「Verify your account」と書かれた画面が表示されます。
　ここで「Username」にはお好みのユーザー名を入力し、「Email Address」にはメールが届くメールアドレスを（後で認証に使うので有効なメールアドレスを入れてください）、「Password」にはログインするためのパスワードを入力し。「Email preferences」はチェックなしでオーケー、「Verify your account」にはチェックを入れて、もしかしたらReCapchaが走るかも知れませんが落ち着いて対応してください。Recapchaが済むと画面一番下の「Create account」が緑色になって押せるようになるので押します。
　「Create account」を押すと画面が切り替わり、8文字の数字を入れる認証画面になりますので、今入力したメールアドレスに8桁の数字が書かれたメールが届いているのでそれをコピーして貼り付けてください。手打ちでもいいです。
　この認証が終わると「dashboard」に移動します。移動したらアカウントができた証拠なので「GitHub」のアカウントは取れた事になります。準備段階1個目が終了です。GitHubの画面は一回閉じましょう。

## CloudFlareのアカウントを作り必要データをコピーする方法
　次にCloudFlareのアカウントを取ります。アカウントを持っている人は「CloudFlareから必要な情報をコピーする」までスキップして構いません。
　最初に[https://www.cloudflare.com/ja-jp/](https://www.cloudflare.com/ja-jp/)にアクセスします。
　そうすると画面右上にサインアップという文字があるのでそれをクリックします。
　「Cloudflare で始めましょう」というページになるので、「メールアドレス」と「パスワード」を入力します。
　メールアドレスは後で認証に使うので有効なものを設定し、パスワードはログインに使用するものを設定する必要があるのでしっかり考えて設定してください。入力したら「サインアップ」のボタンを押します。
　そうすると画面が切り替わって「CloudFlare」の画面になるのですが、まずメールを確認してCloudFlareからのメールを探し、「Please verify your email address」というメールが届いているのでその中のリンクをクリックして認証を行っておいてください。
　メール認証を済ませた後もう一度[https://www.cloudflare.com/ja-jp/](https://www.cloudflare.com/ja-jp/)にアクセスすれば認証が終わっており次の設定ができます。

### CloudFlareから必要な情報をコピーする
　CloudFlareからは「アカウントID」「「APIトークン」をコピーしてきます。
　まず[https://www.cloudflare.com/ja-jp/](https://www.cloudflare.com/ja-jp/)にアクセスし、画面右上の「ログイン」を押します。
　画面が切り替わり、前回と違って画面左と下にも情報が書かれているのを確認して、それから画面左側にある「Worker & Pages」をクリックします。
　すると「Workers & Pages を始めましょう」のページに移動するので、画面中央にある「ワーカーの作成」をクリックします。
　画面が切り替わり「作成”Hello World” スクリプト」という画面になるので、画面中程にある「名前」の項目を好きなアプリ名に書き換えます。これは内部的にしか使わないので、私は「showyabskysocial」としました。入力したら、「名前」の項目を「ワーカー名」としてメモしておきます。その上で画面右下の「デプロイ」を押します。
　画面が切り替わって「おめでとうございますワーカーはリージョンにデプロイされます: Earth」という表示になったらワーカー名が有効になったという事なので成功です。
　次に画面右上にある人物アイコンをクリックして「マイ　プロフィール」をクリックします。画面が切り替わりますので、画面左側の「APIトークン」をクリックします。
　「API トークン」の画面に切り替わり、画面内に「トークンを作成する」というボタンがあるので押します。
　切り替わった画面、「Cloudflare Workers を編集する」の横にある「テンプレートを利用する」をクリックします。
　切り替わった画面を下まで送り、「アカウント リソース」の項目を「包含」はそのままに右の項目を「すべてのアカウント」に変更し、次の「ゾーン　リソース」の項目を「包含」はそのままに右の項目を「すべてのゾーン」に変更し、画面一番下の「概要に進む」をクリックします。
　そうすると「Cloudflare Workers を編集する API トークンの概要」の画面に進むので、画面下の「トークンを作成する」をクリックします。
　これで「Cloudflare Workers を編集する API トークンの作成に成功しました」という画面に進み、画面内にCopyと書かれた領域にAPIトークンが表示されるので、これをコピーしてAPIトークンとしてテキストにコピペしておきます。
　最後にアカウントIDを取得します。一度画面左上の「CloudFlare」のアイコンをクリックしてホーム画面に戻り、そして画面左側の「Workers & Pages」をクリックします。
　すると概要のページが表示され、画面右側のアカウントの詳細の中に「アカウントID」という項目が表示されているので、このコピー枠をコピーして、「アカウントID」としてテキストに保存します。
　これでCloudFlareで必要な情報が集まりました。

## CloudFlare用のツールwranglerをインストールする方法
　今回CloudFlareでBlueSkyのWebhookを使うにはCloudFlare用のツールである「wrangler」をインストールする必要があります。
　「wrangler」をインストールするにはパソコンに「Node.js」と「npm」というツールをインストールする必要があるのでまず「Node.js」と「npm」をインストールします。
　MacとLinuxの場合、「HomeBrew」というツールを使ってインストールするのが楽です。
　HomeBrewは[https://brew.sh](https://brew.sh)からアクセスが可能で、そこにインストール方法が書いてあるのですが英語で、今日本語ページがエラーを吐いている状態なので端的にインストールするコードを貼っておきます。
　ターミナルから以下を実行して下さい。
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
　以上を実行するとHomeBrewがインストールされます。
　そして続いて以下を実行すると、「node.js」と「npm」が一気にインストールされます。
`brew install node`
　最後に次を実行すると「wrangler」がインストールされます。
`npm install -g wrangler`

　Windows環境の方は次のリンクを参考にして「wrangler」をインストールして下さい。
　参考リンク：[Windows 環境でCloudflare 開発ツール Wranglerを設定する方法とHello World!の実行まで](https://zenn.dev/kameoncloud/articles/1fac9762aab4ec)
　全部実行しなくても大丈夫で、「wrangler」のインストールまで行えれば大丈夫です。

## GitHubで必要なリポジトリをテンプレートとしてコピーする方法
　今回Webhookを使うにあたってbridge-y氏の作成した「bluesky-worker」というプロジェクト（正式にはリポジトリといいます）からデータをテンプレートとして自分のGitHubにコピーしてきて、それをリモートのPC（今使っているPCです）にコピー（クローンと言います）してきて、リモートのPCで作業を行う必要があります。
　まずGitHubでbridge-y氏の作成した「bluesky-worker」をコピーしてきて、それからリモートにコピーします。
　[https://github.com/bridge-y/bluesky-worker](https://github.com/bridge-y/bluesky-worker)にアクセスします。
　するとGitHubの画面になるので（必要なら認証して下さい）、画面右上にある「Use this template」をクリックします。開いた選択肢で「Crreate new repository」を選ぶと画面が切り替わります。
　切り替わった画面、中程に「owner」の項目に貴方のGitHubアカウントが表示されており、となりの「Repository name」が空欄になっているので好きな名前を入力しましょう。私は「TwitterToBlueSky」と入力しました。
　下の選択項目は「Public」を選択し、画面右下の「Create repository」をクリックするとちょっと時間がかかってリポジトリがコピーされ、自分の新しいリポジトリの画面が表示されます。
　この画面の右上、緑色で囲まれた「Code」をクリックすると開かれるメニューで「GitHub CLI」が開いている部分を「HTTPS」に切り替えると下にコードが書かれており、コピーボタンでコピーができるのでそれを「クローン元のURL」としてコピーしておいて下さい。

## コピーしたGitHubで行う設定
　その後、この画面を途中までスクロールすると「Setup」の項目の中に「Deploy to Cloudflare Workers」というボタンがあるのでそれを押します。
　新しい画面が開き、CloudFlareの画面で「Deploy to Workers」という画面になるので、まず「Authorize Workers」をクリックします。
　「Authorize Deploy to Cloudflare Workers」という画面になるので、「Authorize cloudflare」のボタンを押します。
　CloudFlareの画面に戻ってきて「Deploy to Workers」の画面で「Configure Cloudflare Account」が選択された状態になるので、「I have an account」をクリックします。
　「Add account information」の画面になり「Account ID」と「API Token」入力欄がありますので、先程CloudFlareで取得した「アカウントID」を「Account ID」欄に、「APIトークン」を「API Token」欄に入力して「Connect account」ボタンを押します。
　「Deploy with GitHub Actions」の欄に移動し、「Fork repository」のボタンがあるのでそれを押します。
　「Enable GitHub Actions」の欄になり、「Enable Workflows」の項目がクリックできるのでクリックします。
　そうするとGitHubの画面に飛んで、「Workflows aren’t being run on this forked repository」の画面になるので、「I understand my workflows, go ahead and enable them」のボタンを押して下さい。
　画面が切り替わり、「There are no workflow runs yet.」の画面になるのでこの画面は閉じてオーケーです。
　これでこの手順は終了しました。お疲れ様でした。

## ローカルにGitHubのデータをクローンしてくる方法
　続いて先程作成したGitHubのリポジトリをローカルのPC（今使っているパソコン）にクローンしてくる必要があります。
　コピーを行うツールとして「git」をインストールする必要がありますのでターミナルを開いて以下コマンドを実行します。（Mac/Linxの場合。Windows環境の方は[WindowsにGitをインストールする手順(2023年09月更新)](https://www.curict.com/item/60/60bfe0e.html)を参考にインストールして下さい）
`brew install git`
　「git」がインストールされたら続いて次のコマンドを実行して下さい。
`git clone 「クローン元のURL」`
　するとローカルにデータがコピーされ、「Repository name」で入力した名前のディレクトリにファイルがコピーされます。
　これで殆ど全ての作業は整いました。

## wranglerでPublishする方法
　今開いているターミナルで続いて作業します。まず、「wrangler」の認証を行います。次のコードを実行して下さい。
`wrangler login`
　ブラウザが開いて新しい画面が表示されます。もしメールアドレス（ユーザー名）とパスワードを聞かれる画面だった場合CloudFlareに登録したメールアドレスとパスワードを入力し続行して次の画面に進んで下さい。人によっては次の画面が直接開く事があります。
　「Allow Wrangler to make changes to your Cloudflare account?」という画面が表示されたら画面の一番下、「Allow」ボタンを押して下さい。
　「You have granted authorization to Wrangler!」の画面が出たら認証成功です。次に進みます。

　今開いているターミナルの画面で、先程できた「Repository name」で入力した名前のディレクトリに移動します。
`cd 「クローン元のURL」`
　ディレクトリを移動した後、Wranglerを実行します。次のコマンドを実行して下さい。
`wrangler publish`

　しばらく時間がかかった後、最後に「Published bluesky-worker」と書かれた後にURLが表示されます。これがWebhookのURLの元になります。（例：https://bluesky-worker.xxxxxxxxxx.workers.dev）
　このWebhookの元のURLは後で使うのでメモしておいて下さい。
　長い作業お疲れ様でした。あとはwranglerで変数を設定した後、IFTTTでクロス投稿の設定をすればおしまいです。

## wranglerで変数を設定する方法
　Webhookの元のURLは発行されて、サーバーも動いていますが、変数を設定してあげないとWebhookは上手く動作しません。
　wranglerを使って変数を設定していきます。
　先にBlueSkyにログインして、BlueSKyのユーザー名（username.bsky.social形式）と、Settingからアプリパスワードを取得しておいて下さい。
　その後、次のコマンドを連続で行います。
`wrangler secret put REQUEST_PATH`
　この項目はWebhookの完全なURLを決めるのに必要な文字列です。
　Webhookの完全なURLは「Webhookの元のURL/好きな文字列」になるので、ここの入力欄では「好きな文字列」を入力します。半角英数で入力して下さい。
　例えばURLの元が「https://bluesky-worker.xxxxxxxxxx.workers.dev」で好きな文字列が「testpost」であればWebhookの完全なURLは「https://bluesky-worker.xxxxxxxxxx.workers.dev/testpost」になります。
　コマンドラインで入力を求められるので、WebhookのURLを入力してエンター（リターン）を押します。
　「/」は不要なので好きな文字列を直接入力して下さい。
`wrangler secret put FULL_USERNAME`
　コマンドラインで入力を求められるので、BlueSkyのユーザー名を入力してエンター（リターン）を押します。
`wrangler secret put PASSWORD`
　コマンドラインで入力を求められるので、アプリパスワードを入力してエンター（リターン）を押します。

## Webhookの完成
　全部の設定が終わったら、これでWebhookの完成です！
　Webhookの完全なURLにPOST形式でJSONを投げれば自動でBlueSkyに投稿してくれるようになります。
　何を言っているかはわからなくて大丈夫です、ただIFTTTで次の設定をすれば、TiwtterのポストがBlueSkyに自動でクロスポストされるようになります。

## IFTTTでBlueSkyにクロスポストするための設定
　ではIFTTTで行う設定をガイドします。
　IFTTTにアクセスして、ProかPro+の契約をした後トップ画面に戻ってきて、画面右上にある「Create」をクリックします。
　そうすると「If This」「Then That」と書かれた画面になるので、「IF This」の横に書かれた「Add」ボタンを押します。
　「Choose Service」という画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Twitter」と入力すると大きなアイコンでXのアイコンに「Twitter」と書かれたパネルが出てくるのでそれを押します。
　「Choose a trigger」という画面になるので、画面左下にある「New tweet by you」をクリックします。
　画面が切り替わり「Complete trigger fields」という画面になるので、「Twitter account」の欄が空欄だと思いますので、その右下にある「Add new account」をクリックすると画面がTwitterの認証画面になるので「連携アプリを認証」をクリックすると画面が切り替わり、「Twitter account」の欄が入力されます。
　それを確認して、「Include」の「retweets」と「@replies」は外れたままの状態で「Create trigger」ボタンをクリックします。
　画面が切り替わり、「IF」が入力された状態で「Then that」のボタンが表示された画面になるので右の「Add」ボタンを押します。
　また画面が切り替わり「Choose a service」の画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Webhook」と入力すると大きなアイコンで「Webhooks」と書かれたパネルが出てくるのでそれを押します。
　切り替わった画面は「Make a web request」という項目しかないのでそれをクリックし、切り替わった画面で次を入力して下さい。

- 「URL」：Webhookの完全なURLを入力します。
- 「Method」：「POST」を選択します。
- 「Content Type」：「application/json」を選択します。
- 「Additional Headers」：は空欄のままにします。
- 「body」：は以下を入力します。
`{ “text” : “<<<{{Text}}>>>” }`
　全てを入力したら画面下にある「Create action」をクリックします。
切り替わった画面した、「Continue」のボタンがあるので押します。
　更に画面が切り替わり、「Applet Title」の入力欄があるので、好きなアプレット（IFTTTのアプリ）名をつけて入力します。私の場合「Twitter to BlueSky」としました。入力したら画面下の「Finish」ボタンを押すと画面が切り替わり、画面中央に「Connected」と書かれた画面が表示されてアプレットが完成します。
　これでTwitterに投げたポストがBlueSkyにも投稿されるようになります。お疲れ様でした。

## まとめ
　というわけで物凄く長い作業の末にですが、ようやくTwitterのポストをBlueSkyに自動でクロスポストしてくれる仕組みを作ることができました。
　本当にお疲れ様でした。
　折り返しなしの文章でおおよそ200行のブログになるので、おおよそ70〜80ステップの作業をして貰ったものと思います。本当に、本当にお疲れ様でした。
　IFTTTだけは有料プランが必須ですが、ほかはフリーの範囲内でできる作業なので、手間賃で成り立っているサービスだと理解していただければ幸いです。

　では延々とブログを書いてきましたが、お付き合い頂いてありがとうございました。また何かの記事でお会いできれば幸いです。