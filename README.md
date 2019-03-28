# はじめに
2018年度研究生の武市による改良型PGA-EAXのプログラムです．
本リポジトリはeclipseプロジェクトをそのまま貼り付けたものです．

# 内容物?
## data
インスタンスファイルである`.tsp`ファイルが入っています．

このディレクトリはローカルにあっても無意味で，ジョブの投入先で参照できるようにする必要があります．
適宜クラスタやTSUBAMEにコピーしてください．

## escape
expSuiteに入れる`clusterinfo.txt`のクラスタ版とTSUBAME版が入っています．
クラスターで実行したいときは`cluster/clusterinfo.txt`を，TSUBAMEで実行したいときは`tsubame/clusterinfo.txt`をコピーしてください．

## expSuite
クラスタライブラリのアレです．

## lib/.svn
同上

## remoteShells
同上

## src
ソースコードが入っています．

### src/gaeax
GA-EAXのコアなアルゴリズムに関するソースコードです．
AB-Cycleの構成や，E-Set戦略，データ構造などが記述されています．
AB-Cycleを改良するときは,ここをいじります．

### src/gaeax2018
GA-EAXのフレームワークに関するソースコードです．
以下に列挙するディレクトリの，`*Main.java`を実行すると然るべきところにジョブが投入されます．

- **src/gaeax2018/nagata12Stage1**：永田先生のGA-EAX Stage1の実装です．
- **src/gaeax2018/nagata12Stage2**：永田先生のGA-EAX Stage2の実装です．
- **src/gaeax2018/honda12Stage1**：本田さんのPGA-EAX Stage1の実装です．
- **src/gaeax2018/yamakoshi14Stage1**：山越さんのGA-EAX Stage1の実装です．
- **src/gaeax2018/yamakoshi14Stage2**：山越さんのGA-EAX Stage2の実装です．
- **src/gaeax2018/propose17Stage1**：森下のPGA-EAX Stage1の実装です（枝エントロピーを使用）．
- **src/gaeax2018/propose17Stage2**：森下のPGA-EAX Stage2の実装です（枝エントロピーを使用）．
- **src/gaeax2018/propose18Stage1**：森下のPGA-EAX Stage1の実装です（独自の多様性指標を使用）．
- **src/gaeax2018/propose18Stage2**：森下のPGA-EAX Stage2の実装です（独自の多様性指標を使用）．
- **src/gaeax2018/takeichiStage1** : 武市のPGA-EAX Stage1の実装です（AB-Cycleの生成を改良）.
- **src/gaeax2018/takeichiStage2** : 武市のPGA-EAX Stage2の実装です.

## benchmark_setting.txt
src/gaeax2018にあるベンチマークを走らせるときに必要なテキストデータです．
左からインスタンス名，集団サイズ，打ち切り改善失敗率r-base,　子個体生成数,　既知最良解（適当でもよい）を列挙しています．
`#`で始めればその行を無視します．

## designDirectory
src/gaeax2018にあるベンチマークを実行する際，
そのプログラムが実行されたのがローカルかリモートかを判別するためのファイルです．

# 実行方法
`.tsp`ファイルが手元にある状態から，PGA-EAXの最終的な解を得るまでの手順を以下に示します．
プログラムの実行はsrc/gaeax2018の中身で完結します．

なお，**ファイルの出力先を指定する場合は，あらかじめディレクトリを作っておいてください．**

## １ 近傍リストの生成
`TNeighborListGeneratorMain.java`で`.tsp`ファイルから`.neighbor.obj`ファイルを生成します．

### パラメータ
- **instanceFilePath**：.tspファイルパス
- **neighborNum**：近傍数

### 出力
指定した`filename.tsp`ファイルと同じ場所に`filename.neighbor.obj`の形式で出力されます．

## ２ 2-Optの実行
`TNagata12InitializePopulation.java`で初期集団ファイル`.pop`を生成します．

### パラメータ
- **instanceFilePath**：.tspファイルパス
- **outputFilePath**：出力先.popファイルパス
- **populationSize**：集団サイズ
- **seed**：乱数シード

### 出力
指定したファイルパスに`.pop`ファイルが出力されます．

## ３ Stage1の実行
`*Stage1Main.java`でstage1を実行します．
`*`の部分には，nagata12, honda12, yamakoshi14, propose17, propose18等が入ります．

### パラメータ
全ての手法に共通するパラメータを以下に示します．
それぞれの手法特有のパラメータについては，`design()`メソッドの`designer.begin()`の引数を見てください．

- **instanceFilePath**：.tspファイルパス
- **optimum**：最適解の経路長(わからない場合は指定しない／適当で良い)
- **initFilePath**：初期集団.popファイルパス／初期集団.popファイルがあるディレクトリのパス
- **outputPath** : データの出力先のパス

### 出力
リモートに生成された`expSuite/Exp*/*/`（指定した出力先）以下に，終了時の集団ファイルを初めとした色んなデータが出力されます．

## ４ Stage2の実行
`*Stage2Main.java`でstage2を実行します．
`*`の部分には，nagata12, yamakoshi14, propose17, propose18等が入ります．

### パラメータ
全ての手法に共通するパラメータを以下に示します．
それぞれの手法特有のパラメータについては，`design()`メソッドの`designer.begin()`の引数を見てください．

- **instanceFilePath**：.tspファイルパス
- **optimum**：最適解の経路長(わからない場合は指定しない／適当で良い)
- **initFilePath**：初期集団.popファイルパス／初期集団.popファイルがあるディレクトリのパス
- **populationSize**：集団サイズ
- **childNum**：子個体生成数
- **failPath** : データの出力先（子フォルダとしてpopフォルダとresultフォルダを事前に作ります）

### 出力
リモートに生成された`expSuite/Exp*/*/`（指定した出力先）以下に，終了時の最良解を示した`.result`ファイルを始めとしたログファイルが出力されます．
`.result`ファイルの読み方を次に示します．

#### nagata12, yamakoshi14の場合
`最良経路長,世代数.result`の形式で出力されます．

#### propose17, propose18の場合
`最良経路長,世代数,交叉回数.result`の形式で出力されます．


#### 

