---
title: "ITFFF(Pro以上）を使ってTwitter(現X)へのポストをmastodonにクロス投稿する方法"
date: "2023-09-07"
categories: 
  - "アプリ・ソフト"
  - "webサービス"
  - "アプリ紹介"
---

どうもこんにちは、如月翔也（[@showya\_kiss]([http://twitter.com/showya_kiss](http://twitter.com/showya_kiss))）です。  
　最近私の中でマイクロブログが大盛りあがりなんですが、今現在10個近いマイクロブログを運用しており、全部客層が違うので全部のマイクロブログを使わないと行けないんですが、全部のマイクロブログに対してオリジナル投稿（リポストや引用を含まない）をブラウザを切り替えながら投稿するのは正直面倒くさいので自動化したいニーズがあるのです。  
　今使っているマイクロブログなんですが、実際「Twitter（現X）」、「BlueSky」、「t2.social」、「Nostr」、「mstdn.jp」、「mastodon-japan.net」、「MissKey.io」、「TRPGがすきー（misskey実装）」、「タイッツー」、「Threand」の10個あるんですが、色々頑張って「Twitter（現X）」に投稿したら「BlueSky」と「Nostr」と「mstdn.jp」と「mastodon-japan.net」と「MissKey.io」と「TRPGがすきー」にクロス投稿してくれる仕組みを作ったのでかなり楽になりました。  
　ので、方法を共有します。個数が多いので分けて投稿しますが、今回は「Twitter」を更新したら「mstdn.jp」と「mastodon-japan.net」にクロス投稿してくれる仕組みを解説します。  
　IFTTTというサービスを使うのですが、これ系の連携サービスでTwitterをトリガーにするのはほぼ全部有料プランなので、比較的老舗で安いプランがあるIFTTTでの設定をガイドします。  
　ちなみに詳しい話をすると「Webhook」という仕組みを使って投稿するので、Webhookの設定がわかる人ならIFTTT以外でも設定できると思います。  

## IFTTTとは

　[IFTTT](https://ifttt.com)とは老舗のサービス連携サービスで、意味が分かりづらいと思うのですが、「何かのサービスを利用した時」に連動して「他のサービスを動かす」事ができるサービスです。  
　基本は無料プランで全然問題なく使えるのですが、「Twitterを使った時」に連動したり、何かを使った時に「Twitterを連動して動かす」場合は有料プランが必要です。  
　有料プランとしてIFTTT ProとIFTTT Pro+があるんですが、有料プランで契約すればTwitterが使える他、1つのサービスに対して複数の連動サービスを設定する事ができるので（今回私はTwitterから一気に6個のマイクロブログを連動させています）、有料のProは月額370円とちょっとお高めなんですが、他にスマートホームなんか使っている人は物凄く多くのサービスが連携できるのでおすすめです。  
　本当は紹介キャンペーンとかあれば紹介したかったんですが、特にそういうキャンペーンはないので普通に契約していただくか、Webで検索すると10パーセントオフのキャンペーンをしている人がいるのでその人経由で応募するといいかもしれません。  
　IFTTTを有料で契約すると、例えばWordPressでブログを更新した時にTwitterにタイトルとURLを投稿したりできるので、いっぱいブログを持っている人は便利かもしれません。  
  
　ちなみに、IFTTTを申し込んで早速使おう！と思っても、連携サービスにmastodonが出てこないので騙された、という気持ちになりますが、mastodonそのものは対応していないんですが、mastodonを使うのにWebhookという仕組みが使えて、今回はそれを使ってmastodonに連携する方法なので安心して下さい。  

## mstdn.jpと連携する準備

　まず先に「mstdn.jp」と連携するための準備についてガイドします。  
　まだmstdn.jpを使っていない人は[https://mstdn.jp](https://mstdn.jp)にアクセスしてアカウントを作り、ログインして下さい。  
　アカウントを作りログインした人は、画面左上の歯車アイコンをクリックして画面を切り替え、設定画面が開きますので画面左下をスクロールしていくと「開発」という場所があるのでそこをクリックすると画面右が切り替わり、「アプリ」という画面になります。  
　この画面に「新規アプリ」というボタンがあるのでそれをクリックします。  
　切り替わった画面、「新規アプリ」という画面が出るので、この画面に必要な項目を入力していきます。  
  
　「アプリの名前」：自由に決めて構いません。私は「Twitter to mstdn.jp」としました。  
　「アプリのウェブサイト」：空欄で構いません。  
　「リダイレクトURI」：入っているのをそのままにして良いです。  
　「アクセス権」のチェック欄：「read」と「write」と「follow」にチェックが入っていますが、「read」と「follow」のチェックを外し「write」だけにします。  
　そして画面の一番下、「送信」をクリックします。  
  
　そうすると画面が切り替わり、「アプリが作成されました」と表示されます。 　戻ってきた画面で今作成したアプリ名が表示されるので、そこをクリックします。  
　切り替わった画面に「アクセストークン」という表示があるので、その欄に書かれたランダムな文字列をコピーして保存しておきます。これは後で使うのでメモしておいて下さい。  
　これで事前準備は終了です。  

## IFTTTでの設定

　では早速TwitterのポストをIFTTTを使ってmstdn.jpにクロスポストする設定を行いましょう。  
　[IFTTT](ifttt.com)にアクセスして、ProかPro+の契約をした後トップ画面に戻ってきて、画面右上にある「Create」をクリックします。  
　そうすると「If This」「Then That」と書かれた画面になるので、「IF This」の横に書かれた「Add」ボタンを押します。  
　「Choose Service」という画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Twitter」と入力すると大きなアイコンでXのアイコンに「Twitter」と書かれたパネルが出てくるのでそれを押します。  
　「Choose a trigger」という画面になるので、画面左下にある「New tweet by you」をクリックします。  
　画面が切り替わり「Complete trigger fields」という画面になるので、「Twitter account」の欄が空欄だと思いますので、その右下にある「Add new account」をクリックすると画面がTwitterの認証画面になるので「連携アプリを認証」をクリックすると画面が切り替わり、「Twitter account」の欄が入力されます。  
　それを確認して、「Include」の「retweets」と「@replies」は外れたままの状態で「Create trigger」ボタンをクリックします。  
　画面が切り替わり、「IF」が入力された状態で「Then that」のボタンが表示された画面になるので右の「Add」ボタンを押します。  
　また画面が切り替わり「Choose a service」の画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Webhook」と入力すると大きなアイコンで「Webhooks」と書かれたパネルが出てくるのでそれを押します。  
　切り替わった画面は「Make a web request」という項目しかないのでそれをクリックし、切り替わった画面で次を入力して下さい。  
  
　「URL」：「https://mstdn.jp/api/v1/statuses」を入力します。  
　「Method」：「POST」を選択します。  
　「Content Type」：「application/x-www-form-urlencorded」を選択します。  
　「Additional Headers」：「User-Agent:TwitterToMstdn/1.0」を入力します。ここが肝です。  
　「body」：「access\_token=メモしたアクセストークン&status={{Text}}」を入力し、画面下にある「Create action」をクリックします。  
  
　切り替わった画面した、「Continue」のボタンがあるので押します。  
　更に画面が切り替わり、「Applet Title」の入力欄があるので、好きなアプレット（IFTTTのアプリ）名をつけて入力します。私の場合「Twitter to mstdn.jp」としました。入力したら画面下の「Finish」ボタンを押すと画面が切り替わり、画面中央に「Connected」と書かれた画面が表示されてアプレットが完成します。  
　これでTwitterに投げたポストがmstdn.jpにも投稿されるようになります。お疲れ様でした。  

## mastodon-japan.netと連携する準備

　次に「mastodon-japan.net」と連携するための準備についてガイドします。  
　まだmastodon-japan.netを使っていない人は[https://mastodon-japan.net](https://mastodon-japan.net)にアクセスしてアカウントを作り、ログインして下さい。  
　アカウントを作りログインした人は、画面左上の歯車アイコンをクリックして画面を切り替え、設定画面が開きますので画面左下をスクロールしていくと「開発」という場所があるのでそこをクリックすると画面右が切り替わり、「アプリ」という画面になります。  
　この画面に「新規アプリ」というボタンがあるのでそれをクリックします。  
　切り替わった画面、「新規アプリ」という画面が出るので、この画面に必要な項目を入力していきます。  
  
　「アプリの名前」：自由に決めて構いません。私は「Twitter to mastodon-japan.net」としました。  
　「アプリのウェブサイト」：空欄で構いません。  
　「リダイレクトURI」：入っているのをそのままにして良いです。  
　「アクセス権」のチェック欄：「read」と「write」と「follow」にチェックが入っていますが、「read」と「follow」のチェックを外し「write」だけにします。  
　そして画面の一番下、「送信」をクリックします。  
  
　そうすると画面が切り替わり、「アプリが作成されました」と表示されます。 　戻ってきた画面で今作成したアプリ名が表示されるので、そこをクリックします。  
　切り替わった画面に「アクセストークン」という表示があるので、その欄に書かれたランダムな文字列をコピーして保存しておきます。これは後で使うのでメモしておいて下さい。  
　これで事前準備は終了です。  

## IFTTTでの設定

　では早速TwitterのポストをIFTTTを使ってmastodon-japan.netにクロスポストする設定を行いましょう。  
　[IFTTT](ifttt.com)にアクセスして、ProかPro+の契約をした後トップ画面に戻ってきて、画面右上にある「Create」をクリックします。  
　そうすると「If This」「Then That」と書かれた画面になるので、「IF This」の横に書かれた「Add」ボタンを押します。  
　「Choose Service」という画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Twitter」と入力すると大きなアイコンでXのアイコンに「Twitter」と書かれたパネルが出てくるのでそれを押します。  
　「Choose a trigger」という画面になるので、画面左下にある「New tweet by you」をクリックします。  
　画面が切り替わり「Complete trigger fields」という画面になるので、「Twitter account」の欄が空欄だと思いますので、その右下にある「Add new account」をクリックすると画面がTwitterの認証画面になるので「連携アプリを認証」をクリックすると画面が切り替わり、「Twitter account」の欄が入力されます。  
　それを確認して、「Include」の「retweets」と「@replies」は外れたままの状態で「Create trigger」ボタンをクリックします。  
　画面が切り替わり、「IF」が入力された状態で「Then that」のボタンが表示された画面になるので右の「Add」ボタンを押します。  
　また画面が切り替わり「Choose a service」の画面になるので、画面上にある虫眼鏡アイコンの入力欄に「Webhook」と入力すると大きなアイコンで「Webhooks」と書かれたパネルが出てくるのでそれを押します。  
　切り替わった画面は「Make a web request」という項目しかないのでそれをクリックし、切り替わった画面で次を入力して下さい。  
  
　「URL」：「https://mastodon-japan.net/api/v1/statuses」を入力します。  
　「Method」：「POST」を選択します。  
　「Content Type」：「application/x-www-form-urlencorded」を選択します。  
　「Additional Headers」：「User-Agent:TwitterToMstdn/1.0」を入力します。ここが肝です。  
　「body」：「access\_token=メモしたアクセストークン&status={{Text}}」を入力し、画面下にある「Create action」をクリックします。  
  
　切り替わった画面した、「Continue」のボタンがあるので押します。  
　更に画面が切り替わり、「Applet Title」の入力欄があるので、好きなアプレット（IFTTTのアプリ）名をつけて入力します。私の場合「Twitter to mstdn.jp」としました。入力したら画面下の「Finish」ボタンを押すと画面が切り替わり、画面中央に「Connected」と書かれた画面が表示されてアプレットが完成します。  
　これでTwitterに投げたポストがmastodon-japan.netにも投稿されるようになります。お疲れ様でした。  

## mstdn.jpにもmastodon-japan.netにも一気に投稿したい場合：アプレット数を減らす方法

　ちなみになんですが、ITFFFのProはアクティブにできるアプレット数が20という制限があるので、mstdn.jpもmastodon-japan.netも両方使っている人はアプレットが2個になってしまうので経済的ではないのですが、これを1個にまとめる方法があります。  
　IFTTTのメイン画面から「My Applets」をクリックして、msdtn.jpにポストするアプレットをクリックして切り替わった画面右側にある「Settings」をクリックしてアプレットの詳細画面に入ります。  
　「Edit Applet」の画面、下にWebhookの欄がありますが、その下に「＋」ボタンがありますのでそれをクリックし、開いた画面から「Choose a service」の検索欄に「Webhooks」を入力して「Webhooks」を選び、そこからmastodon-japanの設定をして「Create action」までやって切り替わった画面で「Update」を選ぶと、「Twitter」からポストすると一連の流れでmstdn.jpとmastodon-japan.netまで一気にクロスポストするようになります。  
　この設定をやった場合、mastodon-japan用のアプレットは不要になるので、「My Applets」からmastodon-japan.netのアプレットをクリックして表示された画面の「Connected」をクリックして「Connect」の表示に変えて、それから画面下の「Archive」をクリックすると「Are you sure you want to continue?」のポップアップがでるので「OK」を押せばアーカイブに入れる事ができます。アーカイブに入ったアプレットはProのアプレット数のカウントに入らないので、枠を1個開ける事ができます。  

## まとめ

　というわけで、今回はTwitterに投稿をしたらmstdn.jpとmastodon-japan.netに投稿をクロスポストする事ができるようになる方法をシェアしました。  
　IFTTTは結構使いでがあるので、色々試してみるといいと思いますよ！