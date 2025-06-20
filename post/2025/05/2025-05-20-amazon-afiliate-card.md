---
title: "WordPressにAmazonの商品リンクカードをブックマークレットを使って作る方法"
date: "2025-05-20T21:00:00"
slug: "20250520-amazon-affiliate-card"
categories: 
  - "ソフト・サービス"
  - "Webサービス"
  - "WordPress"
status: publish
---

　こんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。
　今日はWordPressに限らないんですが、Amazonアソシエイトを使っている人がAmazonの商品を紹介する際、ツールバーから入手したURLを貼るだけだと商品の名前も画像も値段も表示されないので自分で記入しないといけず面倒くさいので、それをnoteにAmazonアソシエイトを貼り付けた時のように名前と画像と値段付きの商品カードを表示するHTMLを作るブックマークレットを作ったので、それを共有するのと同時にブックマークレットとは何か、どう設定するのか、コードそのもの、どう使うのかについて説明して行こうと思います。
　作成される商品カードのコードは純粋なHTMLなので、WordPressで使うなら「HTMLブロック」を作ってそれを貼り付ければいいですし、なんなら手打ちで作っているサイトにも商品カードをそのままHTMLで貼り付けて表示できるので、一回設定して使い方を覚えれば便利だと思います。

## Amazonアソシエイトはnoteが強い理由について

　いきなり話が飛ぶんですが、WordPressでAmazonアソシエイトを使うよりもnoteでAmazonアソシエイトを使う方が結果に繋がりやすいです。
　これはなぜかと言うと、noteではAmazonアソシエイトで作ったリンクを「貼り付け」ると内部解析して商品カードにして表示してくれるので、製品名・値段・商品画像が表示され、手間なく読者に訴求する情報を提供できるからなんですよね。
　ただ、WordPressや他のブログだとAmazonアソシエイトのリンクを自動で商品カードにしてくれるものがなく、一生懸命記事で商品を説明しつつ最終的な商品リンクは文字だけだったり、あるいは商品リンクに手を加えて整形してデータをコピーして、という大きな手間をかけないと読者に訴求する商品カードはできないのです。
　その点でnoteは物凄く優秀かつ戦略的なんですが、しかし世の中半分のサイトはWordPressを使っており、WordPressはプレーンではAmazonアソシエイトの商品リンクを商品カードにはできないのでとても勿体ないのです。
　そこで、今回はWordPressでもAmazonアソシエイトの商品を商品カードにするための手段を用意しました。

## 今回使う方法はブックマークレットです。

　今回使う方法は「ブックマークレット」という仕組みを使い、紹介したい商品を開きツールバーから短縮リンクを開いた状態で、「ブックマークレット」で作ったブックマークをクリックすると商品カードのHTMLがクリップボードにコピーされるので、それをWordPressの「HTMLブロック」に貼り付ければ商品カードが表示される、というシステムです。

### ブックマークレットって何？

　ここで最初に「ブックマークレットって何？」という疑問に突き当たると思うのですが、「ブックマークレット」というのは、ブックマーク機能には実は「JavaScript」というブラウザ上で動くプログラムをブックマークとして登録しておき、使いたい時にそのブックマークを押す事で実行できるという機能があるのです。
　これにより、ブックマークを1つ作ってそれを押すだけでブラウザ上で様々な動作をさせる事ができるのでブックマークレットは一部界隈では結構使われており、今回は商品リンクを扱う都合上ブラウザを使って商品リンクを開いているはずなので、ブラウザから使えるブックマークを押すだけで処理が終わるブックマークレットを使う事にしました。
　ブックマークレットで使うJavaScriptはそんなに破壊的な事ができるプログラム言語ではないので、多少のミスがあっても致命的な結果にはなりませんし、今回紹介する物は僕が作ったと言うか、僕が骨格を作ってChatGPTに修正して貰って作った物なのですが、動作確認はちゃんとしているので心配はいらないと思います。心配であればChatGPTに後で紹介するコードを読んで貰って危険な部分がないかを聞いてみて貰うと良いと思います。

### ブックマークレットの作り方

　ブックマークレットの作り方は簡単で、まずブラウザの機能で適当なページをブックマークします。
　ブックマークしたらブックマークバーからできたブックマークを右クリックして「編集」します。
　すると、「名前」と「URL」を入力する欄が表示されます。
　ここで「名前」には好きな名前（「Amazonアソシエイトカード作成」、など）をつけ、URLには以下に表示するコードを丸ごとコピーして貼り付けて、画面下の「保存」を押します。

### ブックマークレットのコード

　ブックマークレットのコードは以下です。

```
javascript:(function(){try{const titleEl=document.getElementById("productTitle");if(!titleEl){alert("商品タイトルが見つかりません。商品ページで実行してください。");return}const name=titleEl.textContent.trim();const shortlinkEl=document.getElementById("amzn-ss-text-shortlink-textarea");let link;if(shortlinkEl){link=shortlinkEl.value.trim()}if(!link){alert("アソシエイトツールバーから「リンク生成 > テキスト」をクリックしてポップオーバーを表示してください");return}let thumbnailSrc="";const imgTagWrapper=document.getElementById("imgTagWrapperId");if(imgTagWrapper){const imgTag=imgTagWrapper.getElementsByTagName("img")[0];if(imgTag){thumbnailSrc=imgTag.getAttribute("src")}}if(!thumbnailSrc){const mainImg=document.querySelector("#landingImage, .a-dynamic-image");if(mainImg){thumbnailSrc=mainImg.getAttribute("src")||mainImg.getAttribute("data-src")}}let price="";const priceSelectors=[".a-price-whole",".a-price .a-offscreen","#apex_offerDisplay_desktop .a-price-whole",".a-price-range .a-price-whole"];for(const selector of priceSelectors){const priceEl=document.querySelector(selector);if(priceEl){price=priceEl.textContent.trim().replace(/[,\.]/g,"");break}}const htmlCode=`<div style="border: 1px solid #aaa; padding: 10px; overflow: hidden;"><a href="${link}" target="_blank" rel="noreferrer noopener nofollow" style="text-decoration: none; color: inherit;">${thumbnailSrc?`<img src="${thumbnailSrc}" alt="${name}" style="width: 200px; float: right; margin-left: 10px; margin-bottom: 5px;" />`:''}<div style="font-size: 14px; color: #333;">${name}</div>${price?`<div style="font-size: 14px; color: #000; font-weight: bold; margin-top: 4px;">¥${price}</div>`:''}<div style="display: inline-block; background: #fdaf17; color: #111; padding: 6px 12px; margin-top: 8px; border-radius: 8px;">Amazonで購入</div></a></div>`;function fallbackCopy(text){const el=document.createElement("textarea");el.value=text;el.style.position="fixed";el.style.left="-999999px";el.style.top="-999999px";document.body.appendChild(el);el.select();el.setSelectionRange(0,99999);try{const successful=document.execCommand("copy");if(successful){alert("HTMLをクリップボードにコピーしました")}else{alert("コピーに失敗しました")}}catch(err){alert("コピーに失敗しました: "+err)}finally{document.body.removeChild(el)}}if(navigator.clipboard&&navigator.clipboard.writeText){navigator.clipboard.writeText(htmlCode).then(function(){alert("HTMLをクリップボードにコピーしました")}).catch(function(){fallbackCopy(htmlCode)})}else{fallbackCopy(htmlCode)}}catch(error){alert("エラーが発生しました: "+error.message)}})();
```

　長いですが登録できるはずです。
　もしこれで動作がうまくいかなかった場合、上のコードを[https://www.minifier.org/](https://www.minifier.org/)で圧縮して、圧縮したコードを登録すれば間違いなく動きます。
　これを貼り付けたら画面下の「保存」をクリックしてブックマークレット（実際はブックマークとして登録されます）を保存します。
　これで準備は完成です。

### ブックマークレットの使い方

　では実際のブックマークレットの使い方です。

1. Amazonにログインして紹介したい商品ページを開く
2. 商品ページに「Amazonアソシエイトツールバー」という文字が表示されているのを確認し、画面右の「リンク作成」ボタンをクリックして「このページへのリンク作成」画面が表示され、「テキストリンクを作成しました。」の下のテキストボックスに「https://amzn.to」から始まるURLが表示されているのを確認する
3. 先ほど作成したブックマークレットをクリックする。エラーがなければ何も表示されないが、商品カードのHTMLがクリップボードにコピーされている
4. WordPressに貼る場合、形式がHTMLなのでブロックエディタで「HTMLブロック」を選択し、HTMLブロック内にコピーされたHTMLコードを貼り付ける
5. これで商品リンクのコードが貼付けされる。終了
6. なお、何故かブラウザSidekickの場合、ブックマークバーから押してもエラーも吐かずうまく実行できないんですが、サイドパネルからブックマークを選択するとうまく動きます。恐らくこれはSidekickだけの問題です。

　という感じで、そんなに手間はないです。紹介ページはどっちにしろ開くはずなので、その時にブックマークボタンを1回押すだけです。

## 一応どういう感じになるのかのサンプルを表示しておきます。

　一応どういう感じの商品カードが作成されるかのサンプルを表示しておきます。
　僕の一押しのキーボードであるHHKB Studioです。とんでもなく値段が高いので万人にはお勧めしませんが、僕の知る限り世界最高のキーボードで、耐久性を考えても一生物の素晴らしい製品なので思い切って買って長く大事に使う事をお勧めします。

<!--!

 <div style="border: 1px solid #aaa; padding: 10px; overflow: hidden;">   <a href="https://amzn.to/43lKX9x" target="_blank" rel="noreferrer noopener nofollow" style="text-decoration: none; color: inherit;">     <img src="https://m.media-amazon.com/images/I/61vJvilj-fL._AC_SY450_.jpg" alt="PFU キーボード HHKB Studio 英語配列／墨 （ポインティングスティック メカニカルキーボード）" style="width: 200px; float: right; margin-left: 10px; margin-bottom: 5px;" />     <div style="font-size: 14px; color: #333;">PFU キーボード HHKB Studio 英語配列／墨 （ポインティングスティック メカニカルキーボード）</div>     <div style="font-size: 14px; color: #000; font-weight: bold; margin-top: 4px;">44,000 円</div>     <div style="display: inline-block; background: #fdaf17; color: #111; padding: 6px 12px; margin-top: 8px; border-radius: 8px;">       Amazonで購入     </div>   </a> </div>

!-->

　どうでしょう、ブログ用の商品カードとしてはそれなりの出来に整形できていると思います。

## という訳で、これからはAmazonアソシエイトの時代です。

　これは僕の個人的な見解なんですが、Amazonを含む複数サイトのカードを作ってくれるアフィリエイトはAmazonアソシエイトと比べると単価が10分の1〜100分の1ですし、Google Adsenseも100万PVとかないとまともな収入にならないんですが、Amazonアソシエイトは直接契約のためか還元割合が物凄く高く、もし審査に合格できるのであればAmazonアソシエイトで色々紹介して行った方が圧倒的に換金額が高いのです。
　というわけで、今まで商品カードが作れないため別のアフィリエイトサービスを使っていましたが、馬鹿らしくなって自分で商品カードを作るためのブックマークレットを作ったので、同じ悩みを持つ人がいたら手助けになるかと思い共有します。
　もし誰かのお役に立てれば幸いです。