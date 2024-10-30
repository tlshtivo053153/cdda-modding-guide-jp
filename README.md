# CDDA MOD ガイド
この文書は Cataclysm:DDA のMOD制作ガイドである。

環境構築、基本的なMOD制作の知識を説明する。
この文書のファイルパスはCDDAの実行ファイルがあるフォルダを基準にする。
例えば、`data/json/effects.json`、`data/mods/Magiclysm/modinfo.json`などと表記する。

## 環境構築

### テキストエディタ
MOD制作に必ず必要なツールはテキストエディタである。
すでに慣れているエディタがあれば、それを使ってもいい。
このガイドではVisual Studio Code(以下VSCodeと表記)を使うものとする。

- [VSCodeのインストール](vscode/INSTALL.md)
- [VSCodeの拡張機能](vscode/EXTENSIONS.md)

### Git
Gitをインストールするかどうかは任意である。

CDDAはGitというバージョン管理システムで開発されている。
CDDAは、よく破壊的変更が起こり、外部MODの修正が必要になる。
Gitを使えば変更差分から、どう修正すればいいのか参考にできる。

github.comを使用してもGitと同様のことができるが、
文字列検索で大きいファイルを検索しないことがある。
Gitを使うデメリットとしては、CDDAが巨大なリポジトリなので
ストレージを数GB消費することである。

自分で開発しているMODをGitで管理してもよい。

TODO: Gitのインストール、基本的な使い方

## 最小限のMOD構成
MODを作るためには`data/mods/`や`mods/`にディレクトリを作る。
そこに`modinfo.json`というファイルを作り、ゲームを起動すると認識される。
ファイルパスが`data/mods/MOD名/modinfo.json`や`mods/MOD名/modinfo.json`
などとなっていれば良い。
`modinfo.json`は次のようなオブジェクトが必要である。

``` json
[
  {
    "type": "MOD_INFO",
    "id": "Mod_ID",
    "name": "MOD名",
    "authors": [ "John", "Jane" ],
    "description": "MODの説明。",
    "category": "graphical",
    "dependencies": [ "dda" ],
    "version": "1.3.bacon"
  }
]
```

必須となるフィールドは `type`, `id`, `name` である。
`type`は`MOD_INFO`で固定である。
`id`は他のMODと衝突しないような固有の文字列にする必要がある。
`name`はゲーム内で表示されるMOD名である。

`authors`はMOD製作者の名前である。

`description`はMODの説明である。

`category`のデータは次のいずれかである:
`content`, `total_conversion`, `items`, `creatures`, `misc_additions`, 
`buildings`, `vehicles`, `rebalance`, `magical`, `item_exclude`, 
`monster_exclude`, `graphical`。
省略した場合は、`MOD/分類なし`(英語:`NO CATEGORY`)に分類する。

`dependencies`は依存MODである。
MODのIDを記述する。
依存MODはこのMODより先にロードする。

`version`はバージョンを表す任意の文字列である。
ユーザーのための情報であり、MODの処理などには影響しない。

## JSONファイル構造
jsonファイルは`modinfo.json`と同じフォルダやサブフォルダに保存することで
読み込まれる。
CDDAのjsonファイルは次のようなオブジェクトのリストである:
``` json
[
  {
    "type": "...",
    "id": "...",
    "...": "..."
  },
  {
    "type": "...",
    "id": "...",
    "...": "..."
  }
]
```
オブジェクトは`{}`で囲まれた部分、
リストは`[]`で囲まれた部分である。
`type`が`"MONSTER"`ならモンスターの定義ができ、
`"recipe"`ならクラフトレシピの定義ができる。
`id`や`name`などのフィールドは`type`によって異なり、公式ドキュメントの
[JSON_INFO.md](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INFO.md)
を参照して確認できる。

## 既存の要素の変更
既存の要素を変更するためには、同じ`id`で再定義する必要がある。

例として、簡易木刀を魔改造するMODを作る。
ゲームのバージョンは0.Hを用いる。
`mods/cheated_sword_wood/`フォルダを作り、`modinfo.json`を作る:
``` json
[
  {
    "type": "MOD_INFO",
    "id": "cheated_sword_wood",
    "name": "簡易木刀の魔改造"
  }
]
```

まず、簡易木刀が定義されているjsonを見つける。
しかし、`data/json/`フォルダ内を`簡易木刀`という文字列で検索しても
見つけることはできない。
なぜなら、簡易木刀を定義している記述には`"name": { "str": "2-by-sword" }`
と英語で書いてあるからである。
ゲーム内では翻訳ファイルを用いて、`"2-by-sword"`という文字列を`"簡易木刀"`という
文字列に変換して表示している。
その定義部分を見つける方法は色々あるが、今回は
[Hitchhiker's Guide to the Cataclysm](https://cdda-guide.nornagon.net/?v=cdda-0.H-2024-10-14-1533&lang=ja)
を利用する。

Webページを開いたら、右上の検索欄で`簡易木刀`を検索する。
すると、1件だけ検索に合致したものが見つかるので、そのページを開く。
下方向にスクロールして`Raw JSONデータ`をクリックする。
これで表示されたjsonオブジェクトが簡易木刀の定義である。
このオブジェクトをコピーして`weapon.json`を作る。

コピーしたオブジェクトを`[]`で囲って、
`"longest_side"`の文字列を`"1 cm"`に、
`"melee_damage"`のオブジェクトの`"bash"`の値を1000に、書き換える:
``` json
[
  {
    "type": "GENERIC",
    "id": "sword_wood",
    "symbol": "!",
    "color": "brown",
    "name": {
      "str": "2-by-sword"
    },
    "description": "A plank with a handle and a whittled-down point.  Not much good for slashing, but much better than your bare hands.",
    "material": [
      "wood"
    ],
    "volume": "2400 ml",
    "weight": "1000 g",
    "longest_side": "1 cm",
    "to_hit": {
      "grip": "solid",
      "length": "long",
      "surface": "any",
      "balance": "neutral"
    },
    "price_postapoc": 50,
    "flags": [
      "BELT_CLIP"
    ],
    "weapon_category": [
      "MEDIUM_SWORDS"
    ],
    "techniques": [
      "WBLOCK_1"
    ],
    "melee_damage": {
      "bash": 1000,
      "cut": 1
    }
  }
]
```
これで、世界生成時にこのMODを適用すれば、簡易木刀を長さ1cm、打撃1000に変更できる。

しかし、これでは変更内容に対して、記述が冗長すぎる。
上記のMODは既存の定義をまるごと別の記述で上書きしている。
ゲーム内のインベントリ画面で簡易木刀の詳細を見ると、所属が`'簡易木刀の魔改造'`となっていて、このMODのアイテムであることがわかる。
このMODを導入しない場合は、所属が`'コア - Dark Days Ahead'`となる。
元の記述を継承して記述を簡略化できる`copy-from`を用いて改良していく。
次のように`weapon.json`を書き換える:
``` json
[
  {
    "copy-from": "sword_wood",
    "type": "GENERIC",
    "id": "sword_wood",
    "name": {
      "str": "2-by-sword"
    },
    "longest_side": "1 cm",
    "melee_damage": {
      "bash": 1000,
      "cut": 1
    }
  }
]
```
`"copy-from"`で継承元の簡易木刀("sword_wood")を指定している。
`"id"`も`"copy-from"`と同じ"sword_wood"を指定することで定義の上書きをすることができる。
`"type"`は`"copy-from"`を使っても省略はできない。
`"name"`は変更することはないが、ないとエラーになる。
`"longest_side"`と`"melee_damage"`は継承元の記述を上書きしている。
それ以外の省略したフィールドは、継承元の記述を参照している。
ゲーム内のインベントリ画面で、このアイテムを見ると、所属が
`'コア - Dark Days Ahead' > '簡易木刀の魔改造'`となり、
継承していることを確認できる。

`"copy-from"`に関する詳細は、公式ドキュメントの
[JSON_INHERITANCE.md](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INHERITANCE.md)
を参照すること。

慣れるまでは、正常に動作するjsonの記述見つけて、コピーして編集するという
開発方法がミスを抑えられると思う。
公式ドキュメントのJSON_INFO.mdを参照すれば、ゲーム内のたいていの要素は記述できる。
しかし、記述例には`//`でのコメントで説明しているものがあり、
そのままコピーして利用しようとすると邪魔になる。
さらに、ドキュメントの内容が古い可能性もある。
実装したいものに近いjsonの記述を見つけて編集すれば作業量もミスも抑えられるだろう。

## 参考となるjsonの見つけ方
上記の例では、Hitchhiker's Guide to the Cataclysmを用いて、元になるjsonを見つけた。
しかし、このWebアプリでは検索できない要素も存在している。
例えば、`type`が`item_group`、`mapgen`、`talk_topic`、`effect_on_condition`
などのjsonオブジェクトである。
また、本体同梱MODを含めたMOD内の検索もできない。

ゲーム内の情報とテキストエディタを用いて、目的の記述を見つける方法を紹介していく。

### キーコンフィグの設定
原語表示、デバッグメニュー、デバッグモードのキーを割り当てる。

ゲームを起動したら、[設定]->[キー割当]を選ぶ。
`+`キーを押して、`原語表示`を選択して、割り当てるキーを押す(例: Ctrl+L、LanguageのL)。
`デバッグ`と入力して、`+`キーを押して、`デバッグメニュー`を選択してキーを割り当てる(例: Ctrl+D、DebugのD)。
同様に、`切替/デバッグモード`にキーを割り当てる(例: F1)

### `data/json/`フォルダの検索
簡易木刀を例として`data/json/`フォルダ内を検索して目的のjsonファイルを見つけ出す。

まず、バニラ環境の世界を作り、ゲームを始める。
デバッグメニューを出して、[生成]->[入手 - アイテム]を選択する。
アイテムの生成画面に移るので、`/`キーを押して`簡易木刀`を入力して`Enter`キーを押す。
この画面の右上に`#数値: sword_wood`と書いてある。
この`sword_wood`が簡易木刀のidである。
この`#数値`の部分はMOD開発には全く関係ないので注意。
次は、テキストエディタでgrep検索を行う。

VSCodeを起動して、[ファイル]->[フォルダーを開く]を実行して、`data/json/`を開く。
次に、[編集]->[フォルダーを指定して検索]を選択して、検索欄に`sword_wood`と入力する。
検索欄の下側にある`エディターで開く`をクリックする。
エディタ上の検索欄の一番右にある合同記号(≡)のようなものをクリックして、
検索に一致した行のみを表示する。
[編集]->[検索]を選び、`id`と入力する。
これで、idが`"sword_wood"`と`"sword_wood_large"`の2つに絞り込めた。
簡易木刀のidは`"sword_wood"`なので`"id": "sword_wood"`の行にカーソルを合わせて、
次の操作のいずれかで定義を確認できる。

- マウスでダブルクリックして、ファイルを開く。
- マウスで右クリックして、定義へ移動を選びファイルを開く。
- マウスで右クリックして、[ピーク]->[定義をここに表示]を選び、その場に表示する。
- `Ctrl`キーを押しながら、マウスで左クリックしてファイルを開く。

これで簡易木刀の定義を見つけることができた。

### idがわからないjsonオブジェクトの見つけ方
idがわからない場合は固有の文字列を見つけて検索をする。

例として、避難シェルターにあるコンソールのダイアログの記述を調べて、
そのjsonオブジェクトがどう利用されているかを見ていく。
まず、`原語表示`で言語を英語に切り替える。
次に、避難シェルターにあるコンソールを調べる。
表示されている`"The computer awaits input…"`という文字列をエディタで検索する。
候補が2件見つかるが、どちらも完全に文字列が一致しているので、
このままではどちらが正しいものかわからない。
検索をエディタで開いて合同記号(≡)のような記号の左の欄に`10`と入力する。
これで検索行の前後10行が表示できる。
`"responses"`の要素を見ると`"Emergency Message."`、`"Contact Us"`、`"Log off."`
などのテキストが存在する方が避難シェルターのダイアログであることがわかる。
そのidは`"COMP_REFUGEE_CENTER_MAIN"`である。

次はこのidを、どんなところで使用しているかを見ていく。
`"COMP_REFUGEE_CENTER_MAIN"`を検索すると、次の2つのファイルにこの文字列が含まれる。

- mapgen/nested/shelter_nested.json
- npcs/computers/TALK_COMPUTER.json

前者の`shelter_nested.json`はmapgenを定義しているファイル、
後者の`TALK_COMPUTER.json`はtalk_topicを定義しているファイルである。

前者の検索が一致した行の記述を見ると、
避難シェルターのコンソールを調べたときに
idが`"COMP_REFUGEE_CENTER_MAIN"`の`talk_topic`を呼び出すという意味になっている。
mapgenの仕様は複雑で、このオブジェクトのidは
フィールド`"nested_mapgen_id"`の値`"shelter_computer"`である。
さらに、`"shelter_computer"`を検索する、という作業を繰り返せば、
jsonオブジェクトがどのような関係を持っているかがわかる。
文字列などから直接検索ができないものは、関係しているもののidから
たどっていくと見つけることができる。

後者の検索が一致した行は、1つはダイアログを定義しているid(先程見つけたid)で、
残りの4つはダイアログの選択肢を選んだときの次の`talk_topic`のidである。

## デバッグモード
デバッグモードを有効にすると、通常では非表示の情報を得ることができる。
その情報とは、モンスターの正確なHP、アイテムのフラグ、デバッグログ、など
である。
これらはMOD開発に役立つかもしれない。

## VSCodeでの文字列検索
文字列検索を行うときに検索欄の右側にある記号で、
大文字と小文字の区別、単語単位の検索、正規表現、
の機能を有効にできる。

### 大文字と小文字の区別
大文字と小文字の区別は`Aa`という文字列の記号をクリックして有効にできる。
デフォルトだと、大文字と小文字の区別がされない。
例えば、`monster`という文字列で検索をすると、
次の文字列に一致する。
`monster`、`Monster`、 `MONSTER`、`mOnStEr`、などである。

大文字と小文字の区別を有効にすると、大文字と小文字の区別を行う。
例えば、`monster`という文字列で検索すると、`monster`という文字列に一致するが、
`Monster`、`MONSTER`, `mOnStEr`には一致しない。

このゲームのidは大文字と小文字を区別する。
検索件数を絞り込みたいときに有効にするといいだろう。

### 単語単位の検索
単語単位の検索は`ab`という文字列が下線でつながったような記号を
クリックして有効にできる。
この機能を有効にすると、単語単位での検索ができる。
単語と認識するのは、文字列の前後が空白スペースや記号になっているものである。
例えば、この機能を使用しない場合`sword_wood`という文字列で検索すると、
`"sword_wood"`、`"sword_wood_large"`という文字列が結果に含まれる。
次は、この機能を有効にして`sword_wood`という文字列を検索すると、
`"sword_wood"`という文字列が結果に含まれているが、
`"sword_wood_large"`という文字列を含む行は見つからない。

idが1文字でも異なれば、それは別のidである。
正確なidがわかっているならば、有効することで検索結果を絞り込める。

### 正規表現
文字列検索を行うときに検索欄の右側にある`.*`という文字列の記号を
クリックして有効にすると正規表現が利用できる。
正規表現を有効にすると複雑な条件を満たす文字列を見つけることができる。
例えば、正規表現を有効にして`id.*sword_wood`という文字列で検索すると、
次の文字列を含む行を見つけることができる。

- `"id": "sword_wood"`
- `"id": "sword_wood_large"`

ただし、`"id":`と`"sword_wood"`の間に改行が含まれて複数行になっていると
一致しないので注意。

正規表現の文法に関する詳しい説明は、ここではしない。
