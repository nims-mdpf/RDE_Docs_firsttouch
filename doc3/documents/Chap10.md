<div class="page" />

# 実験データの可読化

データ登録者によりアップロードされる実験データのファイルは、必ずしもテキストファイルとは限りません。バイナリファイルの場合は、そのまま保存すると、データをダウンロードしたユーザ(データ利用者)が中のデータを確認したい場合、ユーザが読める形に変換する必要があり不便です。

そのため、構造化処理プログラムでバイナリデータの実験データを読み込んで処理するのであれば、それを誰もが読みやすいファイル書式に変換して保存する、つまり、可読化するとよいと考えられます。

本章では、そういった"可読化"の例として、CSV形式でのファイル出力を実施します。

## 想定

* 前章までの処理にて、実験データの読み込みにpandas というデータ解析ライブラリを使用したので、そのライブラリが用意しているCSV 出力関数（DataFrame クラスのto_csv メソッド）を使用し処理する。
* CSV の保存先は`structured`フォルダとする。

> 構造化処理では、可視化した画像ファイルと実験の生データ以外は`structured`フォルダに保存します。

## custom_module()にロジックを実装

modules/datasets_process.py 

```python
:
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
    # Write CSV file(s)
    for d in raw_data_df:
        fname = d.columns[1].replace(" ", "") + ".csv"
        csv_file_path = resource_paths.struct / fname
        d.to_csv(csv_file_path, header=True, index=False)
```

> 変数`raw_data_df`を使っていますので、当該変数をセットする処理以降に記述する必要があります。

## (参考)出力結果の確認

上記のロジックを追加後、構造化処理プログラムを実行すると、"data/strucutured/"フォルダの下に以下のようなファイルができます。

```bash
(tenv) $ python main.py

(venv) $ tree data/structured/
data/structured/
├── series1.csv
├── series2.csv
└── series3.csv

1 directory, 3 files
```

それぞれのファイルは以下の様な内容となります。

```bash
(venv) $ cat data/structured/series1.csv 
x,series1
1.0,101.0
2.0,102.0
3.0,103.0
4.0,104.0
5.0,105.0
6.2,106.0
7.0,
8.0,108.0
9.5,109.0
10.0,101.0
11.0,103.0
12.0,105.0
13.0,104.0
14.0,100.0
15.0,98.0
16.0,82.0

(venv) $ cat data/structured/series2.csv 
x,series2
1,101
2,102
3,103
4,104
5,105
6,106
9,109
10,90
12,60
13,52
14,41

(venv) $ cat data/structured/series3.csv 
x,series3
1.0,101.0
1.5,101.2
2.0,102.0
3.0,103.0
4.0,104.0
5.0,105.0
6.0,106.0
9.0,109.0
10.0,90.0
12.0,60.0
13.0,52.0
14.0,41.0
```

