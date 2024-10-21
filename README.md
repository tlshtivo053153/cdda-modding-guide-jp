# CDDA MOD ガイド
この文書は Cataclysm:DDA のMOD制作ガイドである。

環境構築、基本的なMOD制作の知識を説明する。

## 環境構築

### テキストエディタ
MOD制作に必ず必要なツールはテキストエディタである。
すでに慣れているエディタがあれば、それを使ってもいい。
このガイドではVisual Studio Codeを使うものとする。

TODO: VSCodeのインストール、拡張機能の紹介

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
