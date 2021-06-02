# ReDoS


- [ReDoS](#redos)
  - [説明](#説明)
  - [事例](#事例)
  - [原因](#原因)
  - [問題点](#問題点)
  - [ReDoSが起こるケース](#redosが起こるケース)
  - [対策](#対策)
    - [正規表現を使わない](#正規表現を使わない)
    - [DFA型エンジンを使う](#dfa型エンジンを使う)
    - [ライブラリを最新にする](#ライブラリを最新にする)
    - [タイムアウトを設定する](#タイムアウトを設定する)
    - [繰り返し表現に注意する](#繰り返し表現に注意する)
  - [ReDoS関連ライブラリ](#redos関連ライブラリ)
  - [資料](#資料)


## 説明
ReDoS攻撃 とは，正規表現エンジンに対して処理時間が多くかかる入力を与えることで，サービス停止が起こってしまう脆弱性をついた攻撃手法である．


## 事例
2016年，Stack Overflowのサーバが，連続した空白を2万個含む投稿が原因でダウンした．
[Stack Exchange Network Status — Outage Postmortem - July 20, 2016](https://stackstatus.net/post/147710624694/outage-postmortem-july-20-2016)

2019年，Cloudflareが提供する全世界のCDNが，ファイヤーウォールに追加した脆弱性のある正規表現が原因でダウンした．
[Details of the Cloudflare outage on July 2, 2019](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/)

口コミのテキストを整形する処理で，脆弱性のある正規表現を使っていたために本番サイトがダウンした．
[【俺の屍を】クソ正規表現で本番サイトを吹っ飛ばした話【超えていけ】 - Qiita](https://qiita.com/paddy-oti/items/399a0bbd16f3f062c666)


## 原因
VM型エンジンのバックトラックが指数関数的に増加してしまうことが原因で起こる．


## 問題点
- 正規表現は，ブラウザやファイヤーウォール，Webサーバ，データベースなど，様々な処理系で使用されているために攻撃の対象範囲が広い．
- ReDoS脆弱性のある表現は，文法的に正しくても実行時にエラーにならない．
- VM型正規表現エンジンは，マッチング処理が無限に続くのか，近い将来停止するかを事前に知ることができない．
- 正規表現の難読性から問題に気づけない．

## ReDoSが起こるケース
次に示す正規表現では，マッチング処理時間が指数関数的に増加する文字列を作成することができる．
1. 量指定子がネストされている．
  - `/(a*)*b/`
  - `/(a+)+b/`
  - `/(a*){9}/`
2. 選択の両方にサブマッチし得るパターンが繰り返されている．
複数の選択肢が同じテキストに一致する可能性がある場合，正規表現の残りの部分が失敗すると，エンジンは両方を試行するためにバックトラックが壊滅的に増加する．
  - `/(.|\w)*/`
  - `/(a|aa)*/`

次に示す正規表現では，マッチング処理時間が2次関数的（またはそれ以上）に増加する文字列を作成することができる．
1. 繰り返し表現が連結している
   - `/(a|b)*(a|c)*d/` 
   - `/.*.*d/`
   - `/abc.*def.*/` 



## 対策
### 正規表現を使わない
正規表現を使わない他の文字列処理方法がないか考える．


### DFA型エンジンを使う
VM型エンジンではReDoS脆弱性が生じ得るが，DFA型エンジンではそもそもバックトラックによるマッチングは行わないため，そのような脆弱性もない．
したがって，非バックトラッキングエンジンを用いて防ぐ対策も有効である．

JavaScript環境では，次の対策が考えられる．
Node.jsやGoogle ChromeのJavaScriptエンジンである**V8**のv8.8以降では，線形時間での実行を保証する 非バックトラッキング型の正規表現エンジンが付属された．
Node.jsのv16ではV8エンジンがv8.6→v9.0になったため，オプション指定で非バックトラッキングエンジンでの実行が可能になった．
詳細は，[追加の非バックトラッキング正規表現エンジン・V8](https://v8.dev/blog/non-backtracking-regexp)を参照．

他にも，[node-re2](https://www.npmjs.com/package/re2)を用いると，DFA型正規表現エンジンであるGoogle RE2でマッチングを行える．

また，Python環境では，Google RE2をラップしたPython拡張機能である**pyre2**を使用する方法が考えられる．
RE2エンジンは，厳密に正規表現を決定性有限オートマトンにコンパイルするため，線形時間の動作が保証される．
詳細は，[pyre2・PyPI](https://pypi.org/project/pyre2/)を参照．

しかし，これらのDFA型エンジンでは，VM型では実装されている後方参照や先読み後読みなどの機能が提供されていないことが多い．

### ライブラリを最新にする
標準ライブラリでさえもReDoS脆弱性がある場合もある．
[VuXML: Python -- multiple vulnerabilities](https://vuxml.freebsd.org/freebsd/bffa40db-ad50-11eb-86b8-080027846a02.html)
では，Pythonの標準ライブラリであるurllibでのReDoS脆弱性が修正されたと報告されている．

### タイムアウトを設定する
処理時間が長過ぎる場合にエラーを起こすようにする．

### 繰り返し表現に注意する
繰り返し表現に関連したパターンにおいてReDoS脆弱性が生じ得るため，特に，量指定子を含む正規表現は避けるべきである．
しかし，やむを得ず使う場合は，特に次の点に注意することが必要である．
- 量指定子がネストされていないことを確認する
- `|`演算で異なるパスで文字列が真になる入力がないか確認する
- 繰り返し表現が連結していないか確認する


また，曖昧な表現を取り除いて表現を厳密にする．
- 可視化ツールで確認する
- 入力文字数を制限して長い文字列で処理させない
- `*?`や`+?`などの控え目な量指定子を使う（欲張り量指定子は使わない）
- アトミックグループを使う（JavaScriptやPythonでは未実装）
- `{n}`などの範囲量指定子を使って繰り返し回数の上限を決める．
- `^`と`$`を指定する
- `.`の代わりに文字クラス`[]`を使用する




## ReDoS関連ライブラリ
- [@makenowjust-labo/recheck](https://makenowjust-labo.github.io/recheck/)
  - 正規表現の脆弱性解析ができる
  - ScalaとJavaScriptをサポート
  - かなり良さそう
  - [@makenowjust-labo/redos](https://npm.io/package/@makenowjust-labo/redos) は旧リポジトリ名っぽい？(要出典)
- [@makenowjust-labo/recheck](https://makenowjust-labo.github.io/recheck/)
- [@makenowjust-labo/eslint-plugin-redos](https://github.com/MakeNowJust-Labo/eslint-plugin-redos)
  - ReDoS脆弱性を検出するESLintプラグイン
  - @makenowjust-labo/redos で解析
- [safe-regex](https://github.com/substack/safe-regex)
  - ReDoS脆弱性の解析ができる
  - JavaScript用
  - 欠点：繰り返し表現のネストしかチェックしてくれない
- [regex-test](https://github.com/venomaze/regex-test)
  - ミリ秒単位の正規表現テストのタイムアウトができる
  - JavaScript用
  - 欠点：safe-regex で脆弱性判断している
- [RegEx-DoS](https://github.com/jagracey/RegEx-DoS)
  - .jsファイルの正規表現脆弱性解析ができる
  - 欠点：safe-regex で脆弱性判断している
- [safe-regex-parser](https://github.com/b0uh/safe-regex-parser)
  - .jsファイルの正規表現脆弱性解析ができる
  - 欠点：safe-regex で脆弱性判断している
- [rxxr2](https://github.com/superhuman/rxxr2)
  - ソースコードはOCaml製で，実行プログラムはバイナリ形式で配布されている
  - テキストファイルの一行を読んでReDoS脆弱な正規表現か判定する
- [ReScue](https://2bdenny.github.io/ReScue/)
  - ソースコードはJava製で，実行プログラムはjar形式で配布されている
  - テキストファイルの一行を読んでReDoS脆弱な正規表現か判定する

## 資料
- [The Regular Expression Denial of Service (ReDoS) cheat-sheet | by James Davis | Level Up Coding](https://levelup.gitconnected.com/the-regular-expression-denial-of-service-redos-cheat-sheet-a78d0ed7d865)
- [Static Detection of DoS Vulnerabilities inPrograms that use Regular Expressions](https://www.cs.utexas.edu/~isil/redos.pdf)
- [Exploiting ReDoS. Downing Servers With Evil Regular… | by Vickie Li | The Startup | Medium](https://medium.com/swlh/exploiting-redos-d610e8ba531)
- [Regular expression Denial of Service - ReDoS Software Attack | OWASP](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS)
- [How can I recognize an evil regex? - Stack Overflow](https://stackoverflow.com/questions/12841970/how-can-i-recognize-an-evil-regex)

- [ReDos検出プログラムの作成とOSSへの適用 #seccamp](https://www.slideshare.net/ssuser9ef9aa/redososs)
- [ハイブリッド型ReDoS検出プログラムの実装 / PROSYM62 - A Hybrid Approach to Detect ReDoS - Speaker Deck](https://speakerdeck.com/makenowjust/prosym62-a-hybrid-approach-to-detect-redos?slide=8)

- [正規表現を使ったDoS – ReDoS – yohgaki's blog](https://blog.ohgaki.net/regex-dos-redos)
- [ReDoSの回避 – yohgaki's blog](https://blog.ohgaki.net/avoiding-redos)
- [正規表現をより安全に使う方法 – yohgaki's blog](https://blog.ohgaki.net/%E6%AD%A3%E8%A6%8F%E8%A1%A8%E7%8F%BE%E3%82%92%E3%82%88%E3%82%8A%E5%AE%89%E5%85%A8%E3%81%AB%E4%BD%BF%E3%81%86%E6%96%B9%E6%B3%95)
- [StackExchangeが攻撃されたReDoSの効果 – yohgaki's blog](https://blog.ohgaki.net/stackexchange-redos-attack)

- [ReDoSから学ぶ，正規表現の脆弱性について - Qiita](https://qiita.com/flat-field/items/f5b0c803ba0b7030d97a)
- [正規表現の落とし穴（ReDoS - Regular Expressions DoS） - Qiita](https://qiita.com/prograti/items/9b54cf82a08302a5d2c7)
