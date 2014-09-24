# JSON棋譜フォーマット(JKF)
JSONで将棋の棋譜を取り扱う標準形式JKFを定義し，また既存のKIF, KI2, CSA等との相互変換を行うライブラリを提供します．(予定)

##概要
現在仕様策定と実装を進めつつあります．

* 現状の問題として，既存のKIF, KI2, CSA形式が持つ情報はバラバラであり，アプリケーション側の対応に手間がかかる．
* 上記形式の情報に加え，棋譜再生や表示に必要な情報を持った形式をJSONで表し，これを流通用の**JSON棋譜フォーマット(JKF)**とする．
	* JKFが一般的になれば，棋譜再生を行うアプリケーションの作成が容易になる．
	* 一例として，JKFを元に棋譜再生を行うJKFPlayerクラス(JavaScript)を提供する．
* KIF, KI2, CSA各形式のパーサ(JavaScript)を用意し，JKFへ変換できるようにする．
	* これはパーサジェネレータによるパーサである．つまり，**KIF, KI2, CSA形式のEBNF表現**を提供している．これは将棋界において画期的である．

### 棋譜形式の比較

| フォーマット | KIF | KI2 | CSA | JKF |
| --- | --- | --- | --- | --- |
| 0) 1) 元座標(from) | ○ | △(要相対逆算) | ○ | ○ |
| 1) 取った駒(capture) | × | × | × | ○ |
| 1) 2) 成り(promote) | ○ | ○ | △ | ○ |
| 2) 相対情報(relative) | △ | ○ | △ | ○ |
| 2) 同〜(same) | ○ | ○ | × | ○ |
| 消費時間(time) | ○ | × | ○ | ○ |

△=現在の局面を見れば可能 ×=不可能か，以前の局面を見れば可能

* 0) 局面を1手進めるために必要な情報
* 1) 局面を1手戻すために必要な情報
* 2) 可読棋譜にするために必要な情報
* 相対情報: 上下左右など，同じ座標に複数の駒が移動できる場合に駒を区別するための情報．
* 相対逆算: 局面と座標と相対情報から，その位置に移動する駒を特定すること．

##形式の定義
`JSONKifuFormat.d.ts`にある内容です．変更され得ます．

* JKFの定義
	* header `string=>string` ヘッダ情報
	* moves `[以下]` n番目はn手目の棋譜(0番目は初期局面のコメント用)
		* comments `[string]` コメント
		* move? 駒の動き
			* from `PlaceFormat?` 移動元 打った場合はなし
			* to `PlaceFormat` 移動先
			* piece `string` 駒の種類(`FU` `KY`等のCSA形式)
			* same? `boolean` 直前と同じ場合
			* promote? `Bool` 成るかどうか true:成, false:不成, 無いかnull:どちらでもない
			* capture? `string` 取った駒の種類
			* relative? `RelativeString` 相対情報
		* time? 消費時間
			* now `TimeFormat` 1手
			* total `TimeFormat` 合計
		* special? `string` 特殊棋譜(CSAのTORYO, CHUDAN等)
	* result 文字列 結果(先手,後手,上手,下手,千日手,持将棋) // フォーマット調査不足により未定
* `TimeFormat` 時間を表す
	* h? `Integer` 時
	* m `Integer` 分
	* s `Integer` 秒
* `PlaceFormat` 座標を表す
	* x `Integer` 1から9
	* y `Integer` 一から九
* `RelativeString` 文字列で以下の情報を連結
	* 左, 直, 右: それぞれL, C, R(Left, Center/Choku, Right)
	* 上, 寄, 引: それぞれU, M, D(Up, Middle, Down)
	* 打: H(Hit 何か違う気もする)

## プログラム

* `kifuplayer.js`: KIF/KI2/CSA/JKFから棋譜再生を行うライブラリ(下5つとshogi.jsを連結したもの．)
* `kif-parser.{pegjs/js}`: KIFをJSON形式に一対一変換するパーサ
* `ki2-parser.{pegjs/js}`: KI2をJSON形式に一対一変換するパーサ
* `csa-parser.{pegjs/js}`: CSAをJSON形式に一対一変換するパーサ
* `normalizer.{ts/js}`: {KIF/KI2/CSA}と同等の情報しか持たないJKFを完全なJKFに変換するプログラム
* `player.{ts/js}`: JKFを扱う棋譜再生盤の例

## 必要条件
### 使用に際して
同梱予定です．

* [na2hiro/Shogi.js](https://github.com/na2hiro/Shogi.js): 将棋の盤駒を扱うライブラリ

#### 開発に際して
* [PEG.js](http://pegjs.majda.cz/): パーサコンビネータ


## TODO
KIF, KI2, CSAの詳しい仕様を知らないので，漏れがあったら教えて下さい．

* パーサテスト追加
* 棋譜形式対応
	* 初期局面
	* 分岐
	* resultの情報(例: "まで先手勝ち")をどう持つか

## reference

* [CSA標準棋譜ファイル形式](http://www.computer-shogi.org/protocol/record_v22.html)
* [shogi-format](https://code.google.com/p/shogi-format/): こちらは昔自ら提案したもの．大風呂敷だったため挫折しました．今回はより小さく洗練された形式を目指しており，また実装を用意し実用第一で進めていきます．
* [棋譜の表記方法](http://www.shogi.or.jp/faq/kihuhyouki.html): 相対情報の書き方
