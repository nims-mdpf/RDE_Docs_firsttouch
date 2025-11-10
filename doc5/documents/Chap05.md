<div class="page" />

# 本番用Dockerイメージ作成

構造化処理プログラムの開発が一通り済んだら、次にそのプログラムファイルを組み込んだ、新しいDockerイメージを作成して、その確認を行います。

ここまで使っていたDockerイメージでは構造化処理プログラム、つまりPythonスクリプトはホスト側(Dockerを実行しているサーバ上、つまりローカル環境上)のフォルダ上にあり、Dockerコンテナを再起動してもその変更内容が消えずに有効になるような状態で開発をしていました。つまり、いわば`開発用コンテナ`を利用していたことになります。

実際のRDE実行環境で利用されるDockerイメージ、すなわち`本番用コンテナ`では、構造化処理プログラムはDockerイメージの中に含まれている必要があります。

本章では、構造化処理プログラムをDockerイメージ内に含めるようにDockerfileを作成し、Dockerイメージを再生成します。その後、そのDockerイメージを使ったコンテナを使い構造化処理に問題無いことを確認します。

最初に、この時点でコンテナ内にいる場合は、抜けます。

```bash
root@38dc486bc3a5:/app2# exit
exit
$ 
```

## フォルダ移動

`app/container`フォルダに、新しいDockerイメージを作成するためのひな形が用意されていますので、これを変更していきます。

`app/container`に移動します。

```bash
cd ${HOME}/rde-docker
cd app/container
```

> 前述の様に、コンテナ外での編集にはオーナーの変更、もしくはsudoを利用した編集が必要となる場合があります。適宜実施してください。

## requirements.txt

RDEToolKitおよびその依存ライブラリ**以外**のPythonライブラリを利用する場合は、requirements.txtにそのライブラリ名を追記してください。

> 特に追加ライブラリがない場合は、変更不要です。
>
> また、指定時は、以下の様にバージョン番号も含めての記述を推奨します。

requirements.txtの例：

```text
rdetoolkit==1.3.4
pydantic-xml==2.12.1
```

上記の例の場合、`rdetoolkit バージョン1.3.4`とそれに伴う依存ライブラリの他に、`pydantic-xml バージョン2.12.1` およびその依存ライブラリが導入されます。

> "#"で始まる行はコメント行です。そのままでもいいですし、削除してしまっても構いません。

```bash
vi requirements.txt
```

前述の開発環境と同様に、`matplotlib`を指定します。

```text
rdetoolkit==1.3.4
matplotlib==3.10.7
```

> この時点で、開発時のrequirements.txtを参照するには、以下の様にします。

```bash
$ cat ${HOME}/rde-docker/dev_image/requirements.txt 
rdetoolkit==1.3.4
matplotlib==3.10.7
```

## Dockerfile

`app/container/`フォルダにある、ひな形として提供されているDockerfileは以下の内容となっています。

```dockerfile
FROM python:3.11.9

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY main.py /app
COPY modules/ /app/modules/
```

以下の様に書き換えます。

```dockerfile
FROM python:3.12-bookworm

# OSパッケージの更新
RUN apt-get update && apt-get upgrade -y \
&& rm -rf /var/lib/apt/lists/*

# WORKDIR指定
WORKDIR /app

# requirements.txt設置
COPY requirements.txt /app

# 必要なPythonライブラリのインストール
RUN pip install --upgrade pip \
 && pip install -r requirements.txt

# プログラムや設定ファイルなどをコピーする
COPY main.py /app
COPY modules/ /app/modules/
```

1. OSパッケージのアップデートは任意です。必要と思われる場合に追記してください。その他必要なOSパッケージが有る場合はここでインストール処理を追記してください。

> 開発用イメージとは異なり、`treeコマンドのインストール`や、vimの設定ファイルの設置などは実施していません。

2. pip自体のアップデート処理を追加します。

> 構造化処理プログラムの中で"RDEToolKitが必要とするPythonライブラリ以外のPythonライブラリ"を利用する場合は、requirements.txtにバージョン番号を含めて追記します。

3. `modules/`フォルダのコピー処理を追加します。

`container/`フォルダにいることを確認し、Dockerイメージを再構築します。

```bash
cd $HOME/rde-docker/app/container

docker build -t rde-sample:v1.0 ./
```

> 本番用イメージということで、バージョン番号を"v1.0"に変更しています。
> Proxy環境下にいる場合は、ローカル環境の環境変数`$http_proxy`や`$https_proxy`などに適切な設定を行ってから再構築作業を行ってください。

## 確認

先に、開発中に作成されたファイルが残っているので、削除しておきます。

```bash
cd ${HOME}/rde-docker/

sudo chown -R $USER data/

rm -rf ./data/logs
rm -f ./data/job.failed
rm -rf ./data/main_image ./data/other_image ./data/meta ./data/raw ./data/nonshared_raw ./data/structured ./data/thumbnail
rm -rf ./data/attachment ./data/invoice_patch ./data/temp
```

> 前章で作成したプログラムでは、`data/raw/`フォルダにファイルを出力しませんが、RDEToolKitの設定を変更する前のテスト実行などでコピーされている場合があります。上記の様に削除して実行した後、意図せず`data/raw/`フォルダ下にファイルが存在する場合は、構造化処理プログラムを見直してください。

コンテナを実行します。

```bash
$ cd ${HOME}/rde-docker/

$ docker run -it --rm -v ${PWD}/data:/app2/data rde-sample:v1.0 /bin/bash
root@4443d3a722af:/app# cd /app2
root@4443d3a722af:/app2# python /app/main.py 
```

> `docker run`実行前に、正しいフォルダに移動する必要があることに注意してください。

treeコマンドはインストールされていないため実行できませんが、仮にDockerイメージ作成時にtreeコマンドをインストールした場合の実行例を以下に示します。

```bash
root@4443d3a722af:/app2# tree
.
└── data
    ├── attachment
    ├── inputdata
    │   └── sample.data
    ├── invoice
    │   └── invoice.json
    ├── invoice_patch
    ├── logs
    │   └── rdesys.log
    ├── main_image
    │   └── all_series.png
    ├── meta
    │   └── metadata.json
    ├── nonshared_raw
    │   ├── invoice.json.orig
    │   └── sample.data
    ├── other_image
    │   ├── series1.png
    │   ├── series2.png
    │   └── series3.png
    ├── raw
    ├── structured
    │   ├── series1.csv
    │   ├── series2.csv
    │   └── series3.csv
    ├── tasksupport
    │   ├── invoice.schema.json
    │   └── metadata-def.json
    ├── temp
    └── thumbnail
        └── thumbnail.png

16 directories, 16 files
```

> RDEToolKitを利用した構造化処理プログラムでは、`data/`フォルダとして**カレントディレクトリにあるもの**が使われます。そのため、`python main.py`ではなく`cd /app2`の後に、`python /app/main.py`を実行しています。
>
> また、本来は入力データを`nonshared_raw/`フォルダまたは`raw/`フォルダのいずれかにコピーします。今回は、`main.py`の中で双方へのコピー処理を無効にしましたので、双方無効にした場合は、コピー処理を追記する必要があることに注意してください。
