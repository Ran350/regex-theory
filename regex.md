# 正規表現 〜入門編〜

目次
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [正規表現 〜入門編〜](#正規表現-入門編)
  - [資料](#資料)
    - [公式ドキュメント](#公式ドキュメント)
    - [ツール](#ツール)
    - [解説記事](#解説記事)
  - [正規表現の基本三演算](#正規表現の基本三演算)
    - [連接](#連接)
    - [選択](#選択)
    - [繰り返し](#繰り返し)
  - [定義済み文字](#定義済み文字)
    - [位置にマッチするメタ文字](#位置にマッチするメタ文字)
  - [グループ化，キャプチャ](#グループ化キャプチャ)
    - [サブマッチの優先順位](#サブマッチの優先順位)
  - [拡張的な正規表現](#拡張的な正規表現)
    - [先読み，後読み](#先読み後読み)
    - [後方参照](#後方参照)

<!-- /code_chunk_output -->


## 資料

### 公式ドキュメント

- [Perl](https://perldoc.jp/docs/perl/5.18.1/perlreref.pod)
- [Ruby](https://docs.ruby-lang.org/ja/latest/doc/spec=2fregexp.html)
- JavaScript
  - [Mozillaドキュメント](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Regular_Expressions)
  - [ECMAScriptドキュメント](https://tc39.es/ecma262/#sec-regexp-regular-expression-objects)
- [Python](https://docs.python.org/ja/3/library/re.html)
- [Java](https://docs.oracle.com/javase/jp/8/docs/api/java/util/regex/package-summary.html)


### ツール
- [regex101: build, test, and debug regex](https://regex101.com/)
- [CyberZHG/toolbox : 正規表現 => NFA => DFA](https://cyberzhg.github.io/toolbox/nfa2dfa)
- [Regexper : 正規表現可視化ツール](https://regexper.com/#%5Cd%2B%E5%B9%B4%5B%5E%E9%96%93%5D)
- [正規表現チェッカー： PHP、Javascript、Pythonのregexを確認](https://rakko.tools/tools/57/)


### 解説記事
- [正規表現技術入門　技術評論社](https://gihyo.jp/book/2015/978-4-7741-7270-5)
- [基本的な正規表現一覧 | murashun.jp](https://murashun.jp/article/programming/regular-expression.html)
- [正規表現あれこれ - Qiita](https://qiita.com/ikmiyabi/items/12d1127056cdf4f0eea5#%E3%82%B0%E3%83%AB%E3%83%BC%E3%83%97%E5%8C%96)
- [ワイの正規表現入門 - Qiita](https://qiita.com/Yametaro/items/36493c107053ae996b47)



## 正規表現の基本三演算
正規表現は，連接，選択，繰り返しの基本演算を備えている．

### 連接
`re`という正規表現があった場合，`r`というパターンの**直後**に`e`というパターンが続く．

### 選択
`|`を用いると，それぞれのパターンのどれかに一致すればマッチが成立する． 
例えば，`r|e`という正規表現は，`r`または`e`というパターンを表す．
`(abcd|ABCD)EFG`という正規表現は，`abcdEFG`または`ABCDEFG`のどちらかにマッチすることを表す．

また，1文字だけの選択は文字クラス`[]`を用いると簡単に記述できる．
例えば，`[a-z]`はaからzまでの小文字アルファベットの選択を表す．


### 繰り返し
`*`や`+`などの量指定子を用いると，繰り返しを表現できる．
例えば，`a*b`という正規表現は，0回以上の`a`の繰り返しの後に`b`が続くパターンを表し，`b`や`ab`，`aab`などの文字にマッチする．

量指定子と各言語の対応表
|      構文      |           機能           | Perl | Ruby | JavaScript | Python | Java |
| -------------- | ------------------------ | ---- | ---- | ---------- | ------ | ---- |
| 欲張り量指定子 |                          |      |      |            |        |      |
| *              | 0回以上の繰り返し        | ○    | ○    | ○          | ○      | ○    |
| +              | 1回以上の繰り返し        | ○    | ○    | ○          | ○      | ○    |
| ?              | 0回か1回                 | ○    | ○    | ○          | ○      | ○    |
| {n}            | n回の繰り返し            | ○    | ○    | ○          | ○      | ○    |
| {n,}           | n回以上の繰り返し        | ○    | ○    | ○          | ○      | ○    |
| {n,m}          | n回以上m回以下の繰り返し | ○    | ○    | ○          | ○      | ○    |
| 控え目量指定子 |                          |      |      |            |        |      |
| *?             | 0回以上の繰り返し        | ○    | ○    | ○          | ○      | ○    |
| +?             | 1回以上の繰り返し        | ○    | ○    | ○          | ○      | ○    |
| ??             | 0回か1回                 | ○    | ○    | ○          | ○      | ○    |
| {n}?           | n回の繰り返し            | ○    | ○    | ○          | ○      | ○    |
| {n,}?          | n回以上の繰り返し        | ○    | ○    | ○          | ○      | ○    |
| {n,m}?         | n回以上m回以下の繰り返し | ○    | ○    | ○          | ○      | ○    |
| 強欲な量指定子 |                          |      |      |            |        |      |
| *+             | 0回以上の繰り返し        | ○    | ○    | ×          | ×      | ○    |
| ++             | 1回以上の繰り返し        | ○    | ○    | ×          | ×      | ○    |
| ?+             | 0回か1回                 | ○    | ○    | ×          | ×      | ○    |
| {n}+           | n回の繰り返し            | ○    | ×    | ×          | ×      | ○    |
| {n,}+          | n回以上の繰り返し        | ○    | ×    | ×          | ×      | ○    |
| {n,m}          | n回以上m回以下の繰り返し | ○    | ×    | ×          | ×      | ○    |



## 定義済み文字
言語によっても機能が違うので使うときは各ドキュメントを参照するとよい．
基本的な定義済み文字を一部示す．

| 文字 |                     意味                     |  等価な表現   |
| ---- | -------------------------------------------- | ------------- |
| .    | 任意の1文字                                  |               |
| \n   | 改行コード                                   |               |
| \d   | すべての数字                                 | [1-9]         |
| \D   | すべての数字以外の文字                       | [^1-9]        |
| \s   | 垂直タブ以外のすべての空白文字               | [ \t\f\r\n]   |
| \S   | すべての非空白文字                           | [^ \t\f\r\n]  |
| \w   | アルファベット、アンダーバー、数字           | [a-zA-Z_0-9]  |
| \W   | アルファベット、アンダーバー、数字以外の文字 | [^a-zA-Z_0-9] |

### 位置にマッチするメタ文字
| 文字 |            意味            |
| ---- | -------------------------- |
| ^    | 行の先頭の**位置**にマッチ |
| $    | 行の末端の**位置**にマッチ |


## グループ化，キャプチャ

|       構文       |            機能            | Perl | Ruby | JavaScript | Python | Java |
| ---------------- | -------------------------- | ---- | ---- | ---------- | ------ | ---- |
| ()               | グループ化(キャプチャあり) | ○    | ○    | ○          | ○      | ○    |
| (?:)             | グループ化(キャプチャなし) | ○    | ○    | ○          | ○      | ○    |
| (?&lt;name&gt;)  | 名前付きキャプチャ         | ○    | ○    | ×          | ×      | ○    |
| (?'name')        | 名前付きキャプチャ         | ○    | ○    | ×          | ×      | ×    |
| (?P&lt;name&gt;) | 名前付きキャプチャ         | ○    | ×    | ×          | ○      | ×    |


### サブマッチの優先順位
正規表現にキャプチャという機能を追加したために，サブマッチに複数の可能性が出てきてしまうという問題が生じた．
そこで，複数の可能性から1通りのサブマッチを選ぶための優先順位が定められている．
優先順位の高さは**左から右**で，原則として正規表現マッチングはこれに従う．

例えば，正規表現`(a*)(a*)`に対して文字列`aaa`をマッチングさせることを考えると，`("aaa", "")`，`("aa", "a")`，`("a", "aa")`，`("", "aaa")`の4通りのマッチングの可能性がある．
しかし，実際には，左側の`(a*)`が優先され，`("aaa", "")`のみがマッチングする．

また，0回以上の繰り返しを意味する控え目量指定子を用いた正規表現`(a*?)(a*?)`に対して文字列`aaa`をマッチングさせると，`("", "")`がマッチングする．
これに対して，正規表現`^(a*?)(a*?)$`に対して文字列`aaa`をマッチングさせると，`("", "aaa")`がマッチングする．
`(a*?)(a*?)`と`^(a*?)(a*?)$`で結果が異なるのは，少し奇妙な結果に見える．
しかし，これは，`^`と`$`を連接されたことにより2つの`(a*?)`のどちらかが`aaa`にマッチしなければならず，さらに，左側の`(a*?)`の**控え目な希望**が優先されたために，右側の`(a*?)`でマッチした．


## 拡張的な正規表現

### 先読み，後読み
先読み，後読みは，正規表現を使って**位置**を指定することができる機能．
例えば，先読みを用いた正規表現`a(?=B)`に対し，文字列`aAaBaC`をマッチングさせると，2つめの`a`がマッチする．
これは文字列`a`の後に`B`が続くのが2つめの`a`のみであったからである．

否定先読みは，パターンが直後に位置にマッチする．
例えば，正規表現`(?!1234)\d{4}`は，`1234`以外の4桁の数字にマッチする．
これに等価な正規表現は，`[2-9]\d{3}|1[13-9]\d{2}|12[124-9]\d|123[1-35-9]` であるが，一見して何を表しているのかわからない．
この例のように，否定先読みを使うと否定の正規表現が簡単に書ける．
また，ある正規表現`regex`の否定の正規表現も次のように簡単に書ける．
正規表現`regex`の完全一致での否定は，`^(?!regex$).*$`となる．

| 構文  |    機能    | Perl | Ruby | JavaScript | Python | Java |
| ----- | ---------- | ---- | ---- | ---------- | ------ | ---- |
| (?=)  | 先読み     | ○    | ○    | ○          | ○      | ○    |
| (?!)  | 否定先読み | ○    | ○    | ○          | ○      | ○    |
| (?<=) | 後読み     | △    | △    | ×          | △      | △    |
| (?<!) | 否定後読み | △    | △    | ×          | △      | △    |
△バージョンによって異なる


### 後方参照
後方参照は，キャプチャによって取得した部分文字列を，その正規表現の中で参照する機能．
例えば，正規表現`(.*)-\1`は任意の文字列をハイフン`-`で挟んだ文字列にマッチする．
マッチする文字列には，`hoge-hoge`などが考えられる．

|   文字   |                         意味                         |
| -------- | ---------------------------------------------------- |
| \0       | 一致した文字列全体の後方参照                         |
| \1 〜 \9 | 一致した文字列の１～９番目に対応する文字列の後方参照 |



