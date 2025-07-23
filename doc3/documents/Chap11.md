<div class="page" />

# 実験データの可視化

構造化処理では、実験データから数値部分を読み込んだ"後処理"として、その可視化、すなわちグラフ画像作成を実施することがあります。

Python では"matplotlib"という広く使われているグラフライブラリがあり、折れ線、散布図、カラーマップ等、通常利用される種類のグラフ形式のほとんどに対応しています。

本書では、可視化作業として"matplotlib"を使ったグラフファイル(画像ファイル)の作成を実行します。

## 想定

* 複数の計測結果が実験データに収録されている。
* 個別の計測結果に対して、個別の画像を作成する。
* さらに、全計測結果を重ね書きした画像を作成し、それを`メイン画像`にする。

> "メイン画像"は、"data/main_image"フォルダに保存された画像ファイルで、基本的に1つのファイルだけが置かれます。

## custom_module()にロジックを実装

modules/datasets_process.py

```python
：
import matplotlib.pyplot as plt
:
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
    # Write Graph
    title = const_meta_info["data_title"]
    x_label = const_meta_info["x_label"]
    y_label = const_meta_info["y_label"]

    ## by series
    for d in raw_data_df:
        x = d.iloc[:, 0]
        y = d.iloc[:, 1]
        label = d.columns[1]
        fname = resource_paths.other_image / f"{label}.png"
        fig, ax = plt.subplots(figsize=(5, 5), facecolor="white")
        ax.plot(x, y, label=label)
        ax.set_title(title)
        ax.set_xlabel(x_label)
        ax.set_ylabel(y_label)
        ax.legend()
        fig.savefig(fname)
        plt.close(fig)

    ## all series
    fig, ax = plt.subplots(figsize=(5, 5), facecolor="lightblue")
    ax.set_xlabel(x_label)
    ax.set_ylabel(y_label)
    ax.set_title(title)
    for d in raw_data_df:
        x = d.iloc[:, 0]
        y = d.iloc[:, 1]
        label = d.columns[1]
        ax.plot(x, y, label=label)
    ax.legend()
    fname = resource_paths.main_image / "all_series.png"
    fig.savefig(fname)
    plt.close(fig)
```

>　"all_series.png"は、シリーズ毎の画像ファイルとは出力先が異なり、main_imageフォルダに出力することに注意してください。

`ModuleNotFoundError: No module named 'matplotlib'`が表示される場合があります。

```
(tenv) $ python main.py 
:
ModuleNotFoundError: No module named 'matplotlib'
```

この場合は、matplotlibが導入されていないと思われます。`pip list`コマンドでインストールされていないことを確認し、インストールしてください。

```
(tenv) $ pip list | grep matplotlib
(tenv) $ 

(tenv) $ pip install matplotlib
```

再度main.pyを実行します。

```bash
python main.py
```

## (参考)結果の確認

以下の様に、個別のグラフ画像が3つ、それらを重ね合わせた画像が1つ、計4つのファイルが出力されます。

```bash
(tenv) $ tree
：
├── data
│   ├── main_image
│   │   └── all_series.png
│   ├── other_image
│   │   ├── series1.png
│   │   ├── series2.png
│   │   └── series3.png
：
```

