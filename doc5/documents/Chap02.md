<div class="page" />

# 環境準備

## Dockerのインストール

Windows、MacおよびLinuxの環境において、Docker Desktopのインストールが最も簡単ですが、組織によっては有償ライセンスを購入する必要があるため、本書ではDocker Desktopを使わない方法でインストールします。

> 企業でDocker Desktopを利用する条件
>
> 大企業（従業員が 251 人以上、または、年間収入が 1,000 万米ドル以上 ）における Docker Desktop の商用利用には、有料サブスクリプション契約が必要です。
>
> 参考: [Docker is Updating and Extending Our Product Subscriptions - Docker](https://www.docker.com/blog/updating-product-subscriptions/)

Docker Desktopを使わない場合は、Docker公式サイトが提供しているDocker Engineを利用します。Docker Engineは、"dockerd デーモンプロセス"とそれを利用する"API"、コマンドラインインターフェースとしての"dockerコマンド"からなっています。

Docker Engineを具体的に利用する方法は、OSによって異なります。また同じOSであってもいくつもの利用方法があります。

本書では、以下のような環境での利用を想定します。

* Windows環境においては、WSL2環境を利用してLinuxを用意し、その環境上でDocker Engineを利用します。
* Mac環境においては、"lima + Docker Engine"、"Rancher Desktop"を利用する方法などがありますが、"lima + Docker Engine"を利用します。
* Linux環境においては、Docker Engineがそのまま利用できます。

### Windows 環境

Windows環境では、WSL2環境を使用したLinux環境を利用します。

WSL(Windows Subsystem for Linux、Linux用Windowsサブシステム）とは、LinuxのプログラムをWindows 10/11およびWindows Server上で実行するための仕組みです。最初のバージョンであるWSL1はLinuxシステムコールをWindowsシステムコールへ変換する方式でしたが、現在主流のWSL2はLinuxカーネルそのものをHyper-Vで実行する方式に変更されています。

WSL2環境が利用可能になっていない場合は、先にそちらの準備を行ってください。

> WSL2環境のインストールについては本書では触れません。マイクロソフトが提供しているドキュメント( https://learn.microsoft.com/ja-jp/windows/wsl/install )等を参考にインストールしてください。

以下では、WSL2環境が利用でき、その上で`Ubuntu`が利用可能になっていることを前提に進めます。

> 以下に示すやり方は、本稿作成時点のものになります。WindowsのバージョンやDocker Engineのバージョンにより変更される可能性があります。Docker Engineのインストール前に、公式ドキュメントを確認してください。
>
> 参考: [Install Docker Engine on Ubuntu - docker docs](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)

PowerShellを利用して、WSL2上のLinuxを起動します。

以下は、Ubuntu-24.04を起動する場合の例です。

```powershell
PS C:\Windows\System32> wsl -d Ubuntu-24.04
$ 
```

> プロンプトが"$"に変わりましたのでLinux環境に移行したことが分かります。なお、プロンプトは設定により変更できますので、上記とは異なるプロンプトが表示される可能性があります。

2つのコマンドを使ってDocker Engineをインストールします。

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

プロキシ環境下でインストールする場合は、先に環境変数を指定して実行してください。

> プロキシサーバを"proxy.example.com:8080"として記述します。環境に合わせて変更してください。

```bash
export http_proxy=http://proxy.example.com:8080
export https_proxy=http://proxy.example.com:8080

curl -fsSL https://get.docker.com -o get-docker.sh
sudo -E sh get-docker.sh
```

> 環境によっては、さらに事前設定を行う必要が有る場合があります。それぞれの環境で必要な処理を実施してください。
>
> デフォルトでは`sudo`コマンドは、環境変数を引き継ぎません。プロキシの設定は引き継ぐ必要がありますので、`-E`オプションを使う必要があります。

さらに、 Dockerをrootユーザー以外で実行できるように設定します。

```bash
sudo usermod -aG docker $USER
```

また、WSL2でdockerdをデーモンとして管理するためにsystemdのサポートを有効化します。`/etc/wsl.conf` にsystemdを有効化するオプションを以下の通り記述します。

> [systemd サポート - Microsoft Support](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config#systemd-support)

```bash
sudo vi /etc/wsl.conf
```

以下の内容を書き込んで保存します。

```bash
# /etc/wsl.conf
[boot]
systemd=true
```

> 新しいディストリビューション(Ubuntu 24.04LTSなど)では、上記設定が最初から入っている場合があります。

最後に、WSLを再起動するとDockerが使用できるようになります。

```bash
$ exit

wsl --shutdown

wsl -d Ubuntu-24.04
```

> "-d"オプションの後に指定する部分は、利用するディストリビューションによって変わります。上記は"Ubuntu 24.04LTS"を使う場合の例です。
> また、上の例では、`exit`はWSL2上のLinuxに対するコマンド、wslコマンドは、Windows上で実行するコマンドです。

### Mac 環境

limaは、Mac環境上で Linux 仮想マシンを動かす、つまり、Windows環境における`WSL2`的な機能を提供してくれるMac版のツールです。この仮想環境の上でDockerを動作させます。

limaとdocker、docker-composeをbrewでインストールします。

```bash
brew install lima docker docker-compose
```

次にlima上にdocker vmを作るための設定を入手し、一部設定を書き換え保存します。

> 設定ファイルには使うCPU core数、メモリなどが設定されており、自分の環境に合わせて設定することが可能ですが、ここではまず動作させるため標準設定を使用します。

```bash
curl https://raw.githubusercontent.com/lima-vm/lima/master/examples/docker.yaml | sed -e 's%- location: "~"%- location: "~"\n  writable: true%g' > ./tempconf.yaml
```

docker vmを作成します。

```bash
limactl start --name=docker ./tempconf.yaml
```

設定ファイルは一度作成したら不要であるため削除可能です。

```bash
rm ./tempconf.yaml
```

作成が完了すると、以下のようなサンプルコマンドが3行表示されるので実行します。

```bash
# [user]のところは自分のhomeのユーザー名を指定する。
docker context create lima-docker --docker "host=unix://Users/[user]/.lima/docker/sock/docker.sock"
docker context use lima-docker
```

環境変数を設定後、dockerコマンドがdocker vmに接続します。この設定をしないとdocker runができないためご注意ください。

```bash
export DOCKER_HOST="unix:///Users/[user]/.lima/docker/sock/docker.sock"
docker run hello-world
```

> exportコマンドによる環境変数設定は`.zshrc`または`.zprofile`(bashを利用している場合は`.bashrc`や`.bash_profile`)に記述することを推奨します。

VMを起動/停止し、また起動状態の確認するには、`limactl`コマンドを利用します。

```bash
limactl list
# 起動
limactl start doker
# 停止
limactl stop doker
```

> proxy環境下での注意
>
> vm作成時、LocalPCの環境変数`http_proxy`などを引き継ぎます。
> その結果、lima上のdocker vmに書き込まれてしまうので、設定を削除したい場合は以下の手順で実行します。
>
> `ssh -p 49476 -i ~/.lima/_config/user -o NoHostAuthenticationForLocalhost=yes 127.0.0.1`
>
> ログイン後、rootで作業したいときは、`sudo su`とする。
>
> limaのdocker vmの上で/etc/enviromentでproxyの設定が引き継がれていたのが原因であるため、これを無効にします。

また、limaのVMは初期状態では自動で起動してくれないので自動起動させます。やり方は複数ありますが、ここではショートカットアプリを使用する方法を示します。

* ショートカットアプリを起動し新規作成します。
* Appタブのターミナルから`シェルスクリプトを実行`をダブルクリックし、スクリプト入力欄に以下の処理を追加します。

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
limactl start docker
```

* ショートカット一覧画面に戻ると作ったショートカットが出来ているため、右クリックから[Dockに追加]を選択します。
* Dockに出たアイコンを右クリックし[オプション]->[ログイン時に開く]

### Linux 環境

Linuxでは、Docker Engineが利用できます。Linuxディストリビューションによって異なる場合がありますが、以下が一般的な手順です。

1. OS標準提供のDocker Engineのアンインストール (インストール済みの場合)
1. apt/dnf等インストールツールに、Dockerリポジトリの追加
1. apt/dnf等を使って、Docker Engineのインストール
1. dockerdデーモンの起動(および自動起動設定)

> 一般的にOS標準提供のDocker Engineは最新版より古いものであることが多いため、Docker公式から提供されている最新版をインストールして利用するようにします。
> ここでは、インストールの手順詳細については記述しません。公式ドキュメントを参照して作業を実施してください。
>
> [Install Docker Engine - docker docs](https://docs.docker.com/engine/install/)


## プロキシ設定

実行する環境によっては、Dockerに対してプロキシの設定を追加する必要があります。

ここでは、WSL2のLinux環境の場合の設定例を示しますが、他の環境のLinux上でもほぼ同じ設定が可能です。

> プロキシサーバを"proxy.example.com:8080"として記述します。環境に合わせて変更してください。

### dockerデーモン向けの設定

```bash
$ sudo systemctl edit docker

:
[Service]
Environment = 'http_proxy=http://proxy.example.com:8080' 'https_proxy=http://proxy.example.com:8080'
```

> "### Edits below this comment will be discarded"という行の、**上に**追加する必要があります。この行の下に追加した場合は追加されません。

編集後、上記設定内容を有効にし、dockerデーモンを再起動します。

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

`docker info`コマンドにより、有効になったことを確認します。

```bash
$ docker info | grep -i proxy
 HTTP Proxy: http://proxy.example.com:8080
 HTTPS Proxy: http://proxy.example.com:8080
```

### dockerコマンド向けの設定

ホームディレクトリに設定格納用のフォルダを作成し、そこに設定ファイルを作成します。

```bash 
mkdir .docker
vi .docker/config.json
```

設定する内容は以下の様にします。
```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:8080",
      "httpsProxy": "http://proxy.example.com:8080"
    }
  }
}
```

> 本書では、これ以降"WSL2 / Ubuntu 24.04LTS + Docker Engine"を使った環境を想定して記述することにします。その他の環境で利用される場合は、適宜読み替えてください。

## dockerコマンド確認

構造化処理の開発を進める前に、`docker`コマンドが利用できることを確認します。

Linux上で以下のコマンドを実行します。

```bash
$ docker --version
Docker version 28.5.1, build e180ab8
```

> バージョンは随時更新されますので、上記と同じバージョンとなるとは限りません。
>
> "Command 'docker' not found"のようなメッセージが出力される場合は、dockerのインストールを見直し、必要に応じて再度インストールしてください。

## dockerd稼働確認

`hello-world`コンテナを使って、`docker`コマンドおよび`dockerデーモン`の稼働確認を行います。

> `コンテナ`とはアプリやインフラなどを入れた箱であり、`イメージ`とはコンテナを実行するために必要なファイルシステムのことです。
>
> `hello-worldコンテナ`は、[dockerhub](https://hub.docker.com/)公式に提供されているコンテナの1つで、実行すると"Hello from Docker!"と表示するだけのコンテナです。

docker runコマンドでイメージを取得、コンテナの実行をします。


```bash
docker run hello-world
```

結果は以下の様になります。

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:d1b0b5888fbb59111dbf2b3ed698489c41046cb9d6d61743e37ef8d9f3dda06f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

> 中程に"Hello from Docker!"と表示されているのが分かります。`Digest:`など一部の項目は値が異なる可能性があります。

上の実行は、初回実行なのでDockerイメージがダウンロードされていませんでした。そのため最初にダウンロード処理が実行されました。2回目以降の実行では、すでにイメージをダウンロードしているので、表示が短くなります。

```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

(以下省略)
```

> "Hello from Docker!"からの表示となり、それ以降は1回目と同じ表示となります。

この表示により、Docker環境が準備完了であると判断できます。

失敗例をいくつかあげます。

### 失敗例1 ： dockerデーモンが起動していない

```bash
$ docker run hello-world
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

この場合は、dockerデーモンが起動してません。dockerデーモンを起動します。

```bash
sudo systemctl start docker
```

### 失敗例2 ： プロキシ設定不良

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": read tcp MMM.MMM.MMM.MMM:47868->NNN.NNN.NNN.NNN:443: read: connection reset by peer.
See 'docker run --help'.
```

> "MMM.MMM.MMM.MMM"や"NNN.NNN.NNN.NNN"は、環境に応じたIPアドレスが入ります。

この場合は、dockerコマンドやdockerデーモンに対するプロキシサーバの設定に不備があることが考えられます。前述のプロキシ設定の項を参考に設定を見直してください。

dockerデーモン設定を変更した場合は、dockerデーモンの再起動が必要です。

```bash
sudo systemctl restart docker
```

### 失敗例3 ： ソケットファイル(docker.sock)アクセスの権限不足

```bash
(venv) $ docker run hello-world
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.47/containers/json": dial unix /var/run/docker.sock: connect: permission denied
```

この場合は、idコマンドで`docker`グループに所属しているかを確認して、所属していない場合は`docker`グループへの追加(追加方法は前述)を実施してください。

```bash
$ id
uid=1003(devel) gid=1003(devel) groups=1003(devel),27(sudo),100(users),989(docker)
```

> ユーザ名`devel`での実行例です。他に所属しているグループやGIDなどは、実行する環境により変わります。上記のように`docker`グループに所属していれば問題ありません。

上記に問題がない場合でもエラーが消えない場合は、Linuxシステムを再起動してください。

```bash
sudo reboot
```

WSL2の場合は、一度抜けてwslのshutdownを実施後、再度起動します。

```bash
$ exit
logout

PS C:\Windows\System32> wsl --shutdown
PS C:\Windows\System32> wsl -d Ubuntu-24.04
```

> `$`で始まる行は、WSL上で稼働しているUbuntuで実行するコマンド、`PS`で始まる行は、Windows上のPowerShellのコマンドです。
