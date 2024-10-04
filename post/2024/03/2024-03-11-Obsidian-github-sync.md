---
title: "ObsidianをGitHubを使ってWindows/MacOS/Linux/iOS/Androidで同期する方法"
categories: 
  - "Mac"
  - "Windows"
  - "Linux"
  - "iOS"
  - "Android"
  - "アプリ・ソフト"
  - "Webサービス"
  - "アプリ紹介"
slug: "obsidian-sync-by-github"
status: publish
---
どうもこんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。
　前回の記事で「メモ同期ソフト」としていくつかのアプリを取り上げていたんですが、あえて取り上げなかったアプリがあります。
　それは「Obsidian」というアプリです。
　これは厳密にはメモ同期アプリではなく、ナレッジマネジメントのアプリで、ノートとノートを繋げて知識を連鎖させていくのに使うアプリなんですが、MarkDown記法が使えて気軽にメモが取れる仕様になっており、かつ設定方法によってはウェブ経由でノートを同期する事ができるので、本来であれば前回記事で紹介してもいい内容のアプリでした。
　しかし、ObsidianではMac/iOSで同期を取る時にiCloudを使うか、あるいは有料のObsidian Syncを使うしか方法がなく、そしてiCloudを使う場合、WindowsでもiCloudを使う事になるのですが、WindowsでのiCloudは安定性が低く実用的ではなく、そしてObsidian Syncは月8ドルの契約なのでちょっとお値段的に安くもなく、そしてそれが原因で僕自身が試していないのが理由でアプリとして取り上げなかったんです。
　でも、Obsidianはかなり熱狂的な支持者が多く、その中でも技術者コミュニティでは物凄い知名度があり、そして他のアプリと違ってプラグインが積極的に開発されているので拡張して使うという観点では非常に強いアプリでもあるので、なんとか無料でかつiCloud以外で使えないかを考え、色々試した結果できたので、今回ご紹介します。

## Obsidianとは
　Obsidianとはメモサービスであると同時に、メモ同士をリンクしてナレッジを蓄積してそれをグラフ化したりしてナレッジを有効活用するためのツールで、一面の事実としてメモサービスで、これを深く使って行く事で蓄積された個々の情報がメモのリンクを通して一塊の情報となり、知識となり、それを有効活用するためのプラットフォームです。
　また、MarkDownでテキストを書けるエディタとしての側面も持っており、まあエディタ自体はVS Codeに投げるような使い方もできるんですが、Obsidian自体には熱烈な愛好者がおり、その中には剛腕の技術者が多数おり、その技術者さん達がプラグインを作っているので色々機能拡張が出来て便利であるというものです。
　僕も今日ようやくクロスOSで使えるようになったばかりでまだ各種の機能拡張は十分に試せていないんですが、僕が熱烈にお勧めするNotionとも棲み分けができるアプリだという事で、せっかくセットアップもできたのでしばらくObsidianも使って行こうと思います。

## Obsidianの対応OSごとの制限
　Obsidianには対応OSごとにちょっとした制限があり、Obsidianで書いたメモを各種OSで同期したい場合、まず全OSで使える方法として「Obsidian Sync」があるんですが、これは有料です。月8ドルくらいの課金になります。
　そして、iOSとMacOS版の制限なんですが、「Obsidian Sync」を使わない場合、同期に使えるクラウドストレージはiCloudのみなのです。
　Windows版、Linux版、Android版はiCloud以外も使えるのですが、同期したいデバイスの中にiOSかMacOSが入った瞬間、使えるクラウドはiCloud限定になってしまいます。
　そして、Windows版なんかのiCloudは動作が不安定で、同期を行う場合に他デバイスでの入力が反映されなかったり、コンフリクトが発生してデータが蒸発する恐れがあるのでちょっと恐ろしくて使えないんですよね。
　そうすると結局Obsidianで全OS貫通で同期するにはObsidian Syncしかない、という結論になり、お金が勿体ないので今回は避けよう……。という動線になってしまったのです。

## しかし拡張機能で制限を超えられるのです
　ですが、調べてみると、Obsidianには有志の方の作成した機能拡張があり、その中で「Git」という機能拡張を使うと、GitHubで作ったリポジトリを使って各OSのObsidianを同期する事ができるのです。
　なので、今回は、「Git」の機能拡張を使って各種OSで同期設定をする方法をガイドします。

## 今回必要なもの
　今回前準備として必要なのは、「GitHubへの登録」と「リポジトリの作成」、「アクセストークンの作成」と「「.gitignore」を編集する」という4段階です。
　これらは全てGitHubでできますので、1サイトで準備が全部できます。
　翔也ガジェットブログでは何度かGitHubへの登録方法、リポジトリの作成方法をガイドした経緯があるんですが、その記事を読むのも面倒だと思うのでこの記事内でもう一度解説してしまいます。
　「知ってるよ」という方は読み飛ばして進めて頂いて大丈夫です。

## GitHubで行う準備
　GitHubでは以下の準備を行います。
1. GitHubへの登録
2. Obsidianを格納するリポジトリの作成
3. アクセストークンの作成
4. 「.gitignore」を編集する

　これらについては小項目として説明していきます。

### GitHubへの登録
　まずGitHubのアカウントを取得します。アカウントを取得している人はスキップして構いません。
　最初に[https://github.co.jp/](https://github.co.jp/)にアクセスします。
　そうすると画面右上に「サインアップ」というボタンがあるのでそれをクリックします。
　「First, let’s create your user account」という画面になり、「Username」「Email Address」「Password」「Email preferences」「Verify your account」と書かれた画面が表示されます。
　ここで「Username」にはお好みのユーザー名を入力し、「Email Address」にはメールが届くメールアドレスを（後で認証に使うので有効なメールアドレスを入れてください）、「Password」にはログインするためのパスワードを入力し。「Email preferences」はチェックなしでオーケー、「Verify your account」にはチェックを入れて、もしかしたらReCapchaが走るかも知れませんが落ち着いて対応してください。Recapchaが済むと画面一番下の「Create account」が緑色になって押せるようになるので押します。
　「Create account」を押すと画面が切り替わり、8文字の数字を入れる認証画面になりますので、今入力したメールアドレスに8桁の数字が書かれたメールが届いているのでそれをコピーして貼り付けてください。手打ちでもいいです。
　この認証が終わると「dashboard」に移動します。移動したらアカウントができた証拠なので「GitHub」のアカウントは取れた事になります。準備段階1個目は終了です。次にそのままリポジトリを作成します。

### Obsidianを格納するリポジトリの作成
　次に同期するファイルを格納する場所を作ります。GitHubでは一つのプロジェクトに対して1つのリポジトリという箱を用意するので、今回はObsidianのデータを格納するためのリポジトリを作成します。
　今GitHubのダッシュボードにいるはずなので、画面の左側に「Top Repositories」という文字があり、その横に「NEW」というボタンがありますのでそれを押します。
　「Create a new repository」という画面になりますので、「Repository name*」の欄にあなたのリポジトリ一覧内で被らない名前を設定して下さい。何も考えずに「obsidian」とかでいいと思います。
　「Public」と「Private」については、「Public」を選んでしまうとあなたの取ったメモがGitHub経由で世界中に見られてしまうので、「Private」を選んで下さい。
　「Add .gitignore」の項目は後で書き換えるので「AL」を選んでおいて下さい。
　他はいじらずに画面右下の「Create Repository」をクリックすれば新しいリポジトリが作成されます。
　リポジトリ名は後で使うのでメモっておいて下さい。これで第二段階終了です。

### アクセストークンの作成
　ObsidianからGitHubのリポジトリを操作するにはアクセストークンが必要になりますので作成します。
　これは基本90日で使えなくなるので、90日ごとに再作成して新しく設定し直す必要があるので、この操作だけはしっかり覚えておくか、ブックマークするかメモするかで再度実行できるようにしておいて下さい。
　まず画面右上に自分のアイコンがありますので、それをクリックしてメニューを開き、「Setting」を選びます。
　画面が切り替わり、左画面の一番下に「Developer settings」がありますのでクリックします。
　更に画面が切り替わるので、画面左側の「Personal access tokens」をクリックし、開いたメニューで「Fine-grained tokens」をクリックします。
　画面右上に「Generate new token」というボタンがあるのでクリックすると画面が変わるので、以下を選びます。
　「Token name」：好きに付けて構いません。
　「Expiration」；90daysを選択します。
　「Description」：自由に書いて構いません。
　「Resource owner」：自分である事を確認します。
　「Repository access」：「Only select repositories」を選びます。すると下に「Select Repositories」と表示されるので、そこをクリックして検索ができるので先程作ったリポジトリ名を検索してクリックします。
　「Permissions」は以下を指定します。
　「Repository permissions」：「Contents」を「Read and write」にします。
　そのまま画面一番下まで移動し、「Generate token」をクリックします。
　すると画面が切り替わり、画面の中にコピーできる領域とコピーボタンが表示されるので、コピーボタンで内容をコピーし、これは後で使うのでなにかに保存しておきます。
　これでアクセストークンが取得できました。

### 「.gitignore」を編集する
　今アクセストークンを作った状態だと「Setting」にいるはずなので、画面左上のGitHubマークをクリックしてダッシュボードに戻り、検索画面で自分の作ったリポジトリを検索してクリックしてリポジトリの画面に移動します。
　画面内に「.gitignore」というファイルがあるのでそれをクリックし、内容が表示されるので、画面右上の鉛筆マークをクリックして編集モードに入り、CTRL（またはCommand）+aで全部を選択して消去し、代わりに以下をコピペして画面右上の「Commit Changes」をクリックして上書き保存します。
<!--!<pre><code># to exclude Obsidian's settings (including plugin and hotkey configurations)
# .obsidian/

# OR only to exclude workspace cache
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# Add below lines to exclude OS settings and caches
.trash/
.DS_Store</code></pre>!-->
　これで全ての準備が整いました。後はObsidianをインストールして設定するだけです。

## Obsidianをインストールして設定する
　ObsidianのインストールについてはApp Store/Google Playからのインストールか、パソコンの場合インストーラーを落としてきてインストールするだけです。
　起動後の動作は一緒なのでまとめて説明します。

- 起動後の最初の画面：「Create new Vault」を選択します。
    モバイルじゃない人は言語を聞かれると思うので「日本語」を選択します。
- 「Vault Name」：自由に入力していいです。
    「Store in iCloud」はオフのままにして「Create」を選択します。
- モバイルの場合画面左上のメニューボタンを押し、「設定」の歯車マークを選択します。
- モバイルの場合「Setting」が開くので「General」を選び、「Language」を「日本語」にして「Relaunch」を選択します。画面が切り替わります。
- もう一度左上のメニューボタンを押し、「設定」の歯車マークを選択します。
- 「コミュニティプラグイン」を選択し、「コミュニティプラグインを有効化」を選び、切り替わった画面で「コミュニティプラグイン」の「閲覧」を選択します。
- 検索画面が出るので「Git」を入力し（少し遅延があると思います）、「開発者 Vinzent,(Denis Olehov)」の「Git」を選びます。
- 「Git」の画面が出るので「インストール」を選択します。
- インストールが終わったら「有効化」を選択します。
- 次に「オプション」を選びます。
- 「Authentication/Commit Author」の「Username」欄にはGitHubで登録しているユーザーネームを入力します（Ex.darktribe@gmail.com）。
- 「Password/Personal access token」には先程取得したアクセストークンを貼り付けます。
- 貼り付けたら画面右上の「X」ボタンで画面を閉じます。
- モバイルの場合戻ってきた画面を右スワイプして画面右下にあるハンバーガーメニュー（三のマークです）を選択して「コマンドパレットを開く」を選択。モバイルじゃない場合は画面左側にコマンドパレットのアイコンがあるのでそれでコマンドパレットを開きます。
- 「Clone」を検索すると「Git: Clone an existing remote repo」がサジェストされるのでそれを選択
- 「Enter remote URL」にはGitHubのリポジトリのURLを入力します。「https://github.com/ユーザー名/リポジトリ名」になります。
- 「Enter directory for clone. it needs to be emp」は「Value Root」を選びます。
- 「Does your remote repo contain a .obsidian d」は、これが最初のデバイスの場合「No」を、2台目以降の同期に使う場合は「Yes」を選びます。
- 「To avoid conflicts, the local .obsidian detected」が表示された場合は「DELETE ALL YOUR LOCAL CONFIG AND PLUGINS」を選択します。
- 「Specify depth of clone. Leave empty for full」は何も入力せずエンター、または日本語入力の「完了」を押します。

　これでインストールから設定までが終了です。
　では次に同期の操作について説明します。

## 同期の操作
　Obsidianで同期をする場合、「コマンドパレット」から「Open」を検索して「Git: open source control view」を選択してコントロールビューを開きます。
　これで「ダウンロード」のボタンを押すとエラーを吐くので、エラーを修正します。
　設定ボタンから「コミュニティプラグイン」の中にある「Git」を選択し、「Authebticator/Commit Author」に「Author name for commit」の欄と「Author email for commit」の欄があるので、「Author name for commit」の欄には適切な名前、「Authoe email for commit」にはGitHubに登録したメールアドレスを入力してから画面右上の「X」ボタンで画面を閉じて下さい。
　この後は「コマンドパレット」から「Git: open source control view」を使ってGitHubにアップロード、GitHubからダウンロードができるようになるので、簡単に同期が行えるようになります。
　ダウンロード時に思ったファイルが降りてこなかった場合、Obsidianを再起動すると再起動後のフェッチで正しいファイルが落ちてくるので使えるようになると思います。

## という訳でObsidianをGitHubで同期する方法についてでした
　という訳でObsidianをGitHubで同期する方法についてでした。
　僕も今日確認したばかりなので再現を確認しながら記事を作りましたが、何かのお役に立てると幸いです。
　Notionの「なんでも」とUpNoteの「テキストと画像」に加えてObsidianで「ナレッジ」のアプリが入手できたので、うまく使い分けて便利に使っていこうと思います！