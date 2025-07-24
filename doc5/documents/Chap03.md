<div class="page" />

# 開発用Dockerイメージの作成

前述の様に、Dockerを利用した開発では、最初に開発用Dockerイメージを作成し、構造化処理プログラムの開発を行った後、それを含む形で本番実行用のDockerイメージを作成し直します。

本章では、まず最初に`開発用のDockerイメージ`を作成します。

## 作業用フォルダの作成

ホームディレクトリに、作業用のディレクトリを作成し、そこに移動します。

以下では、`rde-docker/`フォルダを作成し利用する場合の例を示します。

```bash
mkdir $HOME/rde-docker
cd $HOME/rde-docker
```

Dockerイメージを作成するための設定ファイルやPythonスクリプトを格納するフォルダとして、`dev_image/`フォルダを使うことにします。

フォルダを作成し、移動します。

```bash
mkdir dev_image
cd dev_image
```

## 必要な設定ファイルの用意

### 構造化処理プログラムの用意

これから開発していくので無くてもいいのですが、必要最低限のファイルとして"main.py"のみ用意することにします。

以下の内容で、`$HOME/rde-docker/dev_image/main.py`を作成します。

```python
import rdetoolkit


def main():
    rdetoolkit.workflows.run()

if __name__ == '__main__':
    main()
```

> Pythonスクリプトなので、インデントに注意してください。

### vimrcファイルの用意

後述するように、ファイルの編集にvimを使うことを想定しています。

今回は以下の内容で`$HOME/rde-docker/dev_image/vimrc`を用意し、後ほどDockerイメージに組み込みます。

```vimrc
syntax on
set clipboard=unnamed,autoselect
```

> 上の内容は、"構文ハイライトを有効"にし、"右クリックによるペーストを有効"にするものです。この設定は任意設定項目なので、必要が無ければ設定不要ですし、逆に他に必要な設定が有る場合は、ここに追加してください。

### requirements.txtの用意

同じフォルダに、以下の内容で`requirements.txt`を用意します。

```
rdetoolkit==1.2.0
```

> 上の例ではバージョン番号を指定してrdetoolkitをインストールしています。"=="の前後に空白を挟むとエラーになりますので注意してください。

### Dockerfileの用意

同じフォルダに、以下の内容で`Dockerfile`というファイルを用意します。

```dockerfile
FROM python:3.12-bookworm

# treeコマンドインストール
RUN apt-get update && apt-get install -y \
    tree \
    vim \
&& rm -rf /var/lib/apt/lists/*

# vim設定ファイルをコピー
COPY vimrc /root/.vimrc

# appディレクトリを作成
WORKDIR /app

# requirements.txt設置
COPY requirements.txt /app

# 必要なPythonライブラリのインストール
RUN pip install --upgrade pip \
 && pip install -r requirements.txt

# プログラムや設定ファイルなどをコピーする
COPY main.py /app
```

> 構造化処理プログラムの実行には`vim`や`tree`コマンドは不要ですが、本マニュアル作成の都合上インストールしています。また構造化処理プログラムを、コンテナ内で編集するため`vim`をインストールしています。他のエディタで編集する場合は、適宜置き換えてください。
>
> vim設定ファイルは、コピー時に名称が変える必要があります(先頭にドット(.)を付与する)。
>

この時点では、``$HOME/rde-docker/dev_image/`には、4つのファイルだけが存在している状態です。

```bash
$ ls $HOME/rde-docker/dev_image/
Dockerfile  main.py  requirements.txt  vimrc
```

## Dockerイメージの名前とバージョンの決定

作成するDockerイメージの名前とバージョンを決めます。ここでは以下を使うことにします。

| 項目 | 値 | 備考 |
| ------ | ------ | ----- |
| 名前 | rde-sample | |
| バージョン | v0.1 | 開発版はv0.x(xは1からはじめ、必要に応じ加算)、本番版でv1.0に変更 |

## Dockerイメージの作成

`Dockerfile`があるフォルダ上にいることを確認します。

```bash
$ pwd
/home/[ユーザ名]/rde-docker/dev_image
```

上記情報を使って、イメージを作成します。Dockerイメージを作成するには`docker build`コマンドを使います。

> そのため、Dockerイメージを作成することを、"ビルドする"という場合があります。

```bash
$ docker build -t rde-sample:v0.1  ./
：
：
 => exporting to image                                             0.9s
 => => exporting layers                                            0.9s
 => => writing image sha256:bcc3b7c02b33792f1b26 …… 98cc68         0.0s
 => => naming to docker.io/library/rde-sample:v1.0                 0.0s
```

> 実行の都度出力は変わります。
>
> `-t`オプションで、上で決めたイメージの名前とバージョンを指定して作成します。名前とバージョンはコロン(:)を挟んで連結する必要があります。
>
> プロキシ環境下で`docker build`する場合は、$http_proxyや$https_proxy環境変数を設定する必要があります。
> これは「`docker image pull`実行時は、systemctlで追加した設定を使う」が「`docker image build`コマンドがベースイメージを pull するときは、シェルの環境変数を参照する」という仕様のためです。

環境によっては、`docker build`コマンドが`SSLCertVerificationError`で正常に終了しない場合があります。その場合はDockerfileの`RUN pip install ～`の前に以下の行を追加してください。

```dockerfile
ARG PIP_TRUSTED_HOST="pypi.python.org files.pythonhosted.org pypi.org pypi.io"
```

`docker image ls`コマンドで、確認します。

```bash
$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
rde-sample    v0.1      b61962ef8995   7 minutes ago   1.64GB
hello-world   latest    74cc54e27dc4   5 weeks ago     10.1kB
```

> `IMAGE ID`や`CREATED`欄に表示される値は、実行される環境に合わせて表示されるので、実際の表記は上記とは異なります。

## コンテナ実行

構造化処理プログラムの開発に入る前に、`docker run`コマンドで、dockerイメージを実行し、問題無く稼働することを確認します。`-it`オプションを付けることで、イメージの中に移動します。

```bash
docker run -it <Dockerイメージ名>:<バージョン> <起動コマンド>
```

`<Dockerイメージ名>:<バージョン>`には、上記で指定したものを、`<起動コマンド>`には、"/bin/bash"を指定します。

```bash
$ docker run -it --rm rde-sample:v0.1 /bin/bash
root@50e097b072ce:/app#
```

> プロンプトが変わったことで、Dockerイメージ内にいることが分かります。"@"以降の文字列(上記例だと"50e097b072ce"の部分)は実行の都度変わります。
>
> `--rm`は、`exit`コマンドによりコンテナから抜けた際にコンテナを削除することを指定します。必須ではありません。
>
> Dockerfile内で`WORKDIR /app`と指定しているので、`/app`に移動した状態から始まります。

```bash
root@50e097b072ce:/app# ls
main.py  requirements.txt
root@50e097b072ce:/app# python --version
Python 3.12.10
```

> イメージをビルドした時期により、pythonのマイナーバージョンが異なる場合があります。

RDEToolKitがインストールされていることを確認します。

```bash
root@50e097b072ce:/app# python -m rdetoolkit version
1.2.0
```

> pypi上のRDEToolKitは随時バージョンアップが行われます。異なるバージョンが表示されることがあることに注意してください。
>
> pythonコマンドが利用出来ない場合は`python3`コマンドを使ってください。

このバージョンでは、この時点で`main.py`を実行すると以下のようなエラーが表示されます。"invoice.schema.json"は必須ファイルですが、当該ファイルを用意していませんので、このエラーが出るのは想定通りです。次章以降で開発作業を実施していきますので、このエラーは無視してください。

```bash
root@50e097b072ce:/app# python main.py
  0%|                                                                                             | 0/1 [00:00<?, ?it/s]
Traceback (most recent call last):
  File "/usr/local/lib/python3.12/site-packages/rdetoolkit/workflows.py", line 229, in run
    status = invoice_mode_process(str(idx), srcpaths, rdeoutput_resource, custom_dataset_function)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/rdetoolkit/modeproc.py", line 369, in invoice_mode_process
    invoice_validate(resource_paths.invoice.joinpath("invoice.json"), schema_path)
  File "/usr/local/lib/python3.12/site-packages/rdetoolkit/validation.py", line 267, in invoice_validate
    raise FileNotFoundError(emsg)
FileNotFoundError: The schema and path do not exist: invoice.schema.json


============================================================
Custom Traceback (simplified and more readable):
============================================================

Traceback (simplified message):
Call Path:
   File: /usr/local/lib/python3.12/site-packages/rdetoolkit/workflows.py, Line: 229 in run()
    └─ File: /usr/local/lib/python3.12/site-packages/rdetoolkit/modeproc.py, Line: 369 in invoice_mode_process()
        └─ File: /usr/local/lib/python3.12/site-packages/rdetoolkit/validation.py, Line: 267 in invoice_validate()
            └─> L267: raise FileNotFoundError(emsg) 🔥

Exception Type: FileNotFoundError
Error: The schema and path do not exist: invoice.schema.json
```

> 実行するRDEToolKitのバージョンにより、表示される行番号は変更される可能性があります。

コンテナから抜けます。

```bash
root@50e097b072ce:/app# exit
exit
```

### コンテナ内でのプロンプト表記について

上で示したように、コンテナ内に入っている場合はプロンプトが`root@(コンテナID):(カレントディレクトリ)#`のように変化します。コンテナIDは、実行する度に変更されますので都度異なるプロンプトが表示されます。

そのため、本書で示す手順通りにやっても、異なるプロンプトが表示されます。

本書において、一連の流れであってもプロンプトが変わっていることがありますが、転記のタイミングによるものですので無視してください。

> 以降の章でも同様となります。
