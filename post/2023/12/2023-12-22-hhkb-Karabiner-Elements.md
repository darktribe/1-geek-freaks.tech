---
title: "HHKB Studio日本語配列のキーマップ変更とKarabinar-Elementsを使ったキーコンビの設定"
categories: 
  - "Mac"
  - "アプリ・ソフト"
  - "その他機械系"
  - "ガジェット紹介"
  - "PC系"
slug: "HHKB-Japansese-and-KeyMap-and-Karabiner-elements"
status: publish
---
　どうもこんにちは、如月翔也（[@showya_kiss](https://twitter.com/showya_kiss)）です。
　今日はHHKB Studio用のキーマップ変更ツールと、Karabiner-Elementsという無料ソフトを使ってHHKB Studio日本語版のキーバインドを変更する方法についてお話したいと思います。

## HHKB Studio日本語配列/墨のキーマップ変更ツールについて
　昨日のブログで「HHKB Studio日本語配列/墨」についてファーストインプレッションをして、その中でHHKB Studio用のキーマップ変更ツールについて、「公式サイトに再生停止を含めたメディアキーが設定できると記載がある」旨を書いていましたが、実際何を指定すればそのキーになるのか全く分からなかったのでメールで問い合わせをしていたのですが、衝撃の事実がわかりました。公式サイトの記載が間違っており、「再生停止」「早送り」「巻き戻し」については該当キーを設定する方法がありません。Mac本体の音量大・音量小・トグル式のミュートはMac設定のHHKBであればfnキーとのコンボで打てます。
　これについては昨日のブログにも注記を入れたのですが、HHKB Studio用のキーマップ変更ツールでは設定できないので、メディアキーを使いたい場合は別のソフトを使って単独キーをマッピングするか、あるいはキーコンビネーションで設定しなければなりません。
　僕はMacで作業する時Apple Musicで音楽を流しっぱなしにし、その気分ではない曲がかかった時は気軽にスキップしたいニーズなのでキーボードにはメディアキーが必要で、これに対応するために「[karabiner-Elements](https://karabiner-elements.pqrs.org/)」というアプリを入れて、ちょっとした理由があってfnキーとのコンボができなかったのでCommandキーを押しながら矢印キーを押す事で「Command+←：巻き戻し」「Command+↓：再生停止」「Command+→：早送り」のメディアキー入力ができるようになりました。
　ので、今日は僕がHHKB Studioのキーマップ変更ツールで行った変更についてと、あとKarabiner-Elementsを使ってキーコンボでメディアキーを入力する方法について記事を書きたいと思います。

## まずHHKB Studioでデフォルトで設定されているキーバインドについて
　そもそもまずMac本体の音量大・音量小・トグル式のミュートはMac設定のHHKBであればfnキーとのコンボで打てるので、設定の必要なく以下のメディアキーは使用可能です。
- fn+a：音量小
- fn+s：音量大
- fn+d：ミュート（トグル式です）

## 僕がいじった設定について
　これはHHKB Studioのキーマップ変更ツールで行った変更ですが、以下を行いました。
- CapsLock → Commandキー
　打ち間違いが多発して面倒くさいので。主にCommandと打ち間違うのでCommandをバインドしました。
- fn+CapsLock → CapsLockキー
　CapsLockはCapsLockで使うので、fnキーで明示して使いたいニーズです。意図して使わない限り使えません。
- fn+↑キー → PageUpキー
- fn+↓キー → PageDownキー
- fn+←キー → Homeキー
- fn+→キー → Endキー
　正式にはHHKB自体のバインドで他のキーが設定されているキーなんですが、こっちに設定した方が直感的にわかりやすいのと、そもそもfn+矢印キーでバインドされているキーマップがわりとどうでもいいキーなので書き換えても良いという判断です。

　HHKB Studioのキーマップツールでいじった設定はこれくらいで、後はまあ慣れていけばいいかな、という感じです。

## Karabiner-Elementsを使う理由
　ここから、HHKB Studioのキーマップツールでは変更できない部分についてキーを変更していきます。
　それにあたって、Mac用の無料ソフト（正式には寄付ソフトなのですが無料で使えます）として「[karabiner-Elements](https://karabiner-elements.pqrs.org/)」というソフトを使います。
　今回これを採用する理由として、まず無料である事があげられるのと、あと色々設定しやすい部分と、もうひとつ、僕はいわゆる「尊師スタイル」と呼ばれる（らしい）ノートパソコンの上にHHKB Studioを置いてHHKB Studioでキー入力をしているのですが、今まで使っていたKeychron K8と違ってHHKB Studioは一回り小さく、そのままMacBookPro15インチ（2018年Intelモデル）のキーボードの上に置くと、HHKB Studioの底面がMacBookProのキーボード部分に干渉し、HHKB Studioをタイプすると勝手にMacBookProの内蔵キーボードが入力されて入力がめちゃくちゃになるのです。
　この状態、Karabiner-Elementsを使うと「特定の外付けキーボードが認識されている間内蔵キーボードの入力を無視する」という事ができるので、それが目当てな部分も大きいです。
　では、Karabiner-Elementsのインストールから実際の設定までをガイドします。

## Karabiner-Elementsのインストール
　Karabiner-Elementsのインストールについては「PC設定のカルマさんの[Macアプリ「Karabiner-Elements」のダウンロードとインストール](https://pc-karuma.net/mac-app-karabiner-elements-install/)」について非常に詳しく書いてあるのでそれを参照する事をおすすめします。

　Karabiner-Elementsをインストールするには、まず「[karabiner-Elements](https://karabiner-elements.pqrs.org/)」の公式サイトにアクセスして「Downkiad v14.13.0」（最新バージョンが変わると数字が変わります）のボタンからDMGファイルをダウンロードしてきて、ダウンロードされたDMGファイルをダブルクリックすると「Karabiner-Elements.pkg」というファイルが出てくるのでダブルクリックします。
　そうするとインストーラー画面が開くので、まず「続ける」を押して、次の画面で「インストール」を押します。
　パスワードや指紋認証が走るので認証を抜けるとインストールが進み、最終的にインストールが終わって「インストールが完了しました」の画面になるので、「閉じる」を押すと、元ファイルを消すかどうかの確認が行われるので、ゴミ箱に送ってもいいですし残してもいいです。
　初めてKarabiner-Elementsをインストールした人はしばらくしてから「機能拡張がブロックされました」という画面が表示されると思います。
　この場合、画面内の「セキュリティ環境設定を開く」をクリックすると「セキュリティとプライバシー」画面が開くので、画面左下にある鍵マークをクリックし、ポップアップする認証画面にユーザー名が入っているはずなのでパスワードを入力して「ロックを解除」をクリックして、「開発元〜〜のシステムソフトウェアの読み込みがブロックされました。」という文字の横にある「許可」をクリックします。
　その操作が終わるとKarabiner-Elementsの画面が開くのですが、画面の上の方にクリックできる領域があるのでそれをクリックすると新しく許可画面が開きますので、「karabiner grabbeer」と「karabiner observer」を許可し（スイッチを音にします。青くなれば許可されています）、画面を閉じます。
　これでインストールは完了です。

## Karabiner-Elementsで「HHKB Studio」が認識されている間だけ内蔵キーボードを無効にする
　先程書きましたが、僕はいわゆる尊師スタイルでノートPCの上にHHKB Studioを置いて入力しているのですが、置く場所と内蔵キーボードのサイズの問題でHHKB Studioでキー入力時に内蔵キーボードも入力されてしまうので、「HHKB Studio」が認識されている間だけ内蔵キーボードを無効化したいのです。
　ここで注意して欲しいのは、単純に内蔵キーボードを無効化するのではない点です。条件なしで内蔵キーボードを無効化すると、何かの事情で「外付けキーボードを認識しない」場合に内蔵キーボードも動かなくなりパソコンが文鎮化するので、ただ単純に内蔵キーボードを無効化すればいいわけではありません。
　「HHKB Studio」が認識されている間だけ内蔵キーボードを無効化するには、HHKB Studioが接続されている状態で、今インストールされたKarabiner-Elementsを起動して設定してあげればいいです。
　アプリケーションフォルダに「Karabiner-Elements.app」と「Karabiner-EventViewer.app」ができているはずなので、「Karabiner-Elements.app」を起動して下さい。
　表示された画面左側に「Devices」項目があるのでそれをクリックすると右側が変わり、スクロールしていくと「HHKB-Studio(PFU Limited)」という項目があります。
　その中に「Disable the buikt-in keynoard while this device is conected」というチェック項目がありますので、それにチェックを入れると、HHKB Studioが認識されている間は内蔵キーボードが無効化されて入力が発生しないので非常に良いです。

## Karabiner-Elementsでキーコンボを使って「巻き戻し」「再生停止」「早送り」キーを実装する方法
　では、早速Karabiner-Elementsを入れたので、これを使う事で以下のキーコンボを実装しましょう。
- 左Command+←キー：「巻き戻し」
- 左Command+↓キー：「再生停止」
- 左Command+→キー：「早送り」
　実は最初「fnキープラス矢印キー」でメディアキーを実装しようと思ったのですが、HHKB Studioはfnキー（厳密にはfn1キー）を押してキーボードを入力すると、fnキー自体は入力された扱いにならず、コンボで打ったキーそのものが入力されてしまいます。
　なのでHHKB Studioのマッピングツールでfn1キーを押した場合のマッピングで各矢印に使わないキーをマッピングして、その文字をKarabiner-Elementsで拾ってメディアキーに変換する方法もあるのですが、先にHHKB Studioでfnキー+矢印キーにはPageUp/PageDown/Home/Endをマッピングしていますし、そもそもCommand+矢印って基本的には何もマッピングされていないはずなので、今回は左Command+矢印キーでマッピングする事にします。

　Karabiner-Elementsは単体キーの差し替えなら「Simple Modifications」を使ってドロップダウンリストから差し替えるキーを選択できるので、もしHHKBでfnキーとのコラボで何か変換したい場合、変換したいキーコードを調べて、「巻き戻し」なら「rewind」に、「再生停止」なら「play_or_pause」に、「早送り」なら「fast_foward」に変換してあげればいいのでそれをお試し頂いていいんですが、今回は左Command+矢印キーのコンボについて説明します。

### 前準備
　キーコンボを打つには自分でコードを書く必要があるのですが、今回はコードはそれそのものを貼り付けるので書かなくてオーケーです。
　ただし、キーコンボ1個ごとに設定の名前が必要で、その名前を用意するのは結局ファイルを作らないといけないので、よそからファイルだけ貰ってきて、中身を書き換えて作っていきます。
　まず画面左側の「Complex Modifications」をクリックします。
　右画面がほぼ何もない画面になりますが、画面上部に「Add predefined rule」というボタンがあるので押します。
　新しい画面が開き、画面下に「Examples」という項目があり、その中に「Change caps_lock to command+control+option+shift.」という項目の横に「enable」というボタンがあるのでそれを押します。
　すると画面が戻り、先程空っぽだった項目の中に「Change caps_lock to command+control+option+shift.」という項目があり、その横に「Edit」という項目があるのでそれをクリックします。
　そして出てきた画面をCommand+aを押して全選択肢、バックスペースで消して、次以降に提示するコードをそのままコピペしてから画面右上の「save」でセーブすると保存され、名前も変わって保存されます。
　次以降の項目をする場合、「Add predefined rule」から「Change caps_lock to command+control+option+shift.」の横の「enable」を押すところからやり直せばいいので、この手間を3回やればメディアキーは設定できます。

### 左Command+←キーで「巻き戻し」キー
　次のコードを貼り付けて「save」します。
```
{
    "description": "Command + Left to rewind",
    "manipulators": [
        {
            "from": {
                "key_code": "left_arrow",
                "modifiers": {
                    "mandatory": [
                        "left_command"
                    ],
                    "optional": [
                        "caps_lock",
                        "option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "rewind"
                }
            ],
            "type": "basic"
        }
    ]
}
```

### 左Command+↓キーで「再生停止」キー
　次のコードを貼り付けて「save」します。
```
{
    "description": "Command + Down to Play/Stop",
    "manipulators": [
        {
            "from": {
                "key_code": "down_arrow",
                "modifiers": {
                    "mandatory": [
                        "left_command"
                    ],
                    "optional": [
                        "caps_lock",
                        "option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "play_or_pause"
                }
            ],
            "type": "basic"
        }
    ]
}
```

### 左Command+右キーで「早送り」キー
　次のコードを貼り付けて「save」します。
```
{
    "description": "Command + Right to fastforward",
    "manipulators": [
        {
            "from": {
                "key_code": "right_arrow",
                "modifiers": {
                    "mandatory": [
                        "left_command"
                    ],
                    "optional": [
                        "caps_lock",
                        "option"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "fastforward"
                }
            ],
            "type": "basic"
        }
    ]
}
```

## 以上でHHKB StudioのキーマップツールとKarabiner-Elementsを使ってのカスタマイズ終了です
　以上でHHKB StudioのキーマップツールとKarabiner-Elementsを使ってのカスタマイズは終了です。
　本当はHHKB Studioのキーマップツールだけでメディアキーを実装できれば良かったんですが。2023年12月22日現在の状態ではキーマップツールでは「巻き戻し」「再生停止」「巻き戻し」は設定できないという回答をメールで貰っているので（公式サイトもこれから修正予定だそうです）、現時点でメディアキーを実装したければKarabiner-Elementsを使うのが一番手っ取り早いと思います。

　6500文字の長文にお付き合い頂きありがとうございました。
　ではまた何かの機会にお会いしましょう。