# practice-docker

実践 Docker - ソフトウェアエンジニアの「Docker よくわからない」で学習した内容

## サーバ仮想化とは

コンピュータのリソースを管理するための様々な技術。例えば 1 台のサーバの中にネットワークやストレージを仮想的に用意して、複数の異なるサーバが動いているように見せかけたりする技術。

### 仮想化の種類

#### ホスト型仮想化: VirtualBox などのようにホスト OS にインストールされ、ゲスト OS を管理

**特徴**:

- すでに使っているマシンに仮想サーバを構築する場合などに有用。
- ホスト OS があるのでブラウザやエディタも普通に使い続けられる。

#### ハイパーバイザー型仮想化: ハードウェアにインストールされ、ゲスト OS を管理

**特徴**:

- ホスト OS の起動をまったりリソースを割り当てたりしなくていい
- ゲスト OS 以外は動かせない
  ※ Microsoft Hyper-V や Linux の KVM など

#### コンテナ型仮想化: ホスト OS にインストールされ、アプリケーションを管理

**特徴**:

- ゲスト OS が存在しないので、起動のコストが大幅に少ない
- サーバではないので、1 つのコンテナに複数のアプリケーションをインストールしない
- ホスト OS があるのでブラウザやエディタも普通に使い続けられる

### コンテナ型仮想環境のメリット

- 起動が早い
- デプロイしやすい
  デプロイ先がイメージレジストリやマネージドなコンテナのデプロイ先があるので、本番稼働もローカル環境もコンテナ型仮想化を前提として構築することが多い。
  コンテナ方仮想化は機能ごとにコンテナを起動するため、不要な設定が混入することを避けられる。

### ホスト OS の違いがコンテナに影響する

**原因**: Linux カーネルの違いによるもの
Linux のカーネルはホスト OS と同じものをインストールするため、結果的にホスト OS が違うと起動するコンテナも違うものになる。
※コンテナは必ずしもホスト OS の影響を完全に受けないわけではない。

## Docker とは

- **Docker Engine**:
  コンテナ型仮想化ソフトウェアのことで、これによりアプリケーションをソフトウェアとして扱うことができる。
  Docker のコンテナは Linux のカーネルと機能を使用しているため、Docker Engine は Linux 上でしか動かない。
- **Docker CLI**: Docker に命令をするコマンド
- **Docker Desktop**: Docker Engine や Linux カーネルが含まれているため、Linux 以外の OS からでも Docker Engine を動かすことができる。
- **Docker Compose**: Docker CLI をまとめて実行してくれる便利なツール。
- **Docker Hub**: Docker のイメージレジストリである Saas サービス。
- **ECS/GKE**: コンテナ管理サービス。Docker Engine の入った Linux のことで、ローカル開発に使ったコンテナをそのままデプロイできる場所のこと。
- **ECR/GCR**: 非公開のイメージレジストリ。プライベートな Docker Hub のことで、「商用サービスのイメージを Docker Hub で公開したくないけど、レジストリに登録しないとデプロイできない」場合に使用される。
- **Kubernetes**: 多数のコンテナを管理するオーケストレーションソフトウェア。ロードバランサーの作成や、アクセス集中時のスケーリングなどが簡単に実現できる

## Docker を理解

### コンテナ

特定のコマンドを実行するために作られるホストマシン上の隔離された領域。
コンテナの実体は Linux の namespace という機能により他と分離されたただの 1 プロセス。そのプロセスをコンテナやイメージというものを用いて扱いやすくした技術が Docker

- **特徴** 1.コンテナはイメージをもとに作られる 2. 　 Docker の CLI や API を使って、生成や起動や停止を行うことができる。 3. 複数のコンテナは違いに独立していて影響できず、独自に動作する。 4. Docker Engine の上ならローカルマシンでも仮想マシンでもクラウド環境でも動かせる。

### イメージ

コンテナの実行に必要なパッケージで、ファイルやメタ情報を集めたもの。

- ベース -> Ubuntu
- インストールするもの -> PHP
- 環境変数
- 設定ファイルの配置内容 -> PHP の設定ファイル
- デフォルト命令

### Dockerfile

既存のイメージにレイヤーをさらに積み重ねるためのテキストファイル

## コンテナの基礎操作

- **コンテナの起動**: `docker container run [option] <image> [command]`
- **コンテナ一覧の確認**: `docker run [option] <image> [command]`
- **コンテナの停止**: `docker container stop [option] <container>`

### 最低限の`nginx`の起動

```
$ docker container run \
    --publish 8080:80  \
    nginx:1.21
```

### 起動中のコンテナを停止せずいきなり削除

```
$ docker container rm \
    --force           \
    <container>
```

### コンテナを起動

- コンテナを起動する - container run

| オプション        | 意味                               | 用途                                                   |
| ----------------- | ---------------------------------- | ------------------------------------------------------ |
| -i, --interactive | コンテナの標準入力に接続する       | コンテナを対話操作する                                 |
| -t, --tty         | 擬似ターミナルを割り当てる         | コンテナを対話操作する                                 |
| -d, --detach      | バックグラウンドで実行する         | ターミナルが固まるのを避ける                           |
| --rm              | 停止済コンテナを自動で削除する     | 起動時に停止済コンテナと一意な情報が衝突するのを避ける |
| --name            | コンテナに名前をつける             | コンテナを指定しやすくする                             |
| --platform        | イメージのアーキテクチャを明示する | M1 Mac で必要な場合がある                              |

### コンテナの対話操作

```
$ docker container run \
    --interactive      \
    --tty              \
    ubuntu:24.04
```

- **短縮バージョン**: `$ docker run -it ubuntu:20.04`

**bash 内**

```
root@46dae08e367a:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.10
DISTRIB_CODENAME=oracular
DISTRIB_DESCRIPTION="Ubuntu Oracular Oriole (development branch)"
root@46dae08e367a:/#
```

### コンテナ停止時に自動で削除

コンテナ起動時に`--rm`オプションを指定することで、停止時に同時に削除される。

```
$ docker container run \
    --rm               \
    --detach           \
    --publish 8080:80  \
    nginx:1.21
```

###　コンテナに名前をつける
コンテナを停止する際など名前で指定できる。

```
$ docker container run \
    --name web-server  \
    --rm               \
    --detach           \
    --publish 8080:80  \
    nginx:1.21
```

**結果**

```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
5b89dbe9f62e   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:8080->80/tcp   web-server
```

### コンテナ起動時の挙動

```
$ docker container run [option] <image> [command]
```

＊＊command\*\*: 任意の命令を実行させる。command なしの場合、デフォルトの命令が実行される。

#### **例１**: `bash` -> bash で起動(デフォルト命令では nginx の起動のみ)

##　コンテナの状態遷移

### コンテナとプロセスについて

### **例 1: Ubuntu コンテナをデフォルト命令(bash)で起動**

- `PID 1`: bash
- `PID 10`: ps

```
% docker container run \
--name ubuntu1 \
> --rm \
> --interactive \
> --tty \
> ubuntu:24.04
root@ae31ad1bffc7:/# ps
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
    9 pts/0    00:00:00 ps
```

\*\*結果＊＊
Ubuntu コンテナを起動し、デフォルト命令の `bash` を使っている状態で `ps` コマンドでコンテナ内のプロセス一覧を確認すると、PID の 1 は `bash` になっています。

### **例 2**: Nginx コンテナをデフォルト命令(nginx)で起動

- nginx を起動

```
% docker container run \
--name nginx1 \
--detach \
--rm \
nginx:1.21
docka3f678198babcbe37238ea46cf71a9a7dd4534495734e13807f6333030eef019
```

- コンテナに入る

```
% docker container exec \
--interactive \
--tty \
nginx1 \
bash
```

- コンテナ内に ps コマンドがないため、コンテナ内で実行しインストールを行う

```
apt update
apt install -y procps
```

**結果**
`PID` の 1 は `nginx` の起動コマンド
`ps`自身と`ps`を実行するために`container exec`で起動した bash のプロセスも存在する。

```
root@a3f678198bab:/# ps x
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 nginx: master process nginx -g daemon off;
   43 pts/0    Ss     0:00 bash
  397 pts/0    R+     0:00 ps x
```

### **例 3**: Nginx コンテナを指定命令(bash)で起動

```
docker container run \
> --name nginx2 \
> --rm \
> --interactive \
> --tty \
> nginx:1.21 \
> bash
```

# コンテナ内に ps コマンドがないためインストール

```
apt update
apt install -y procps
```

```
root@c844801718d3:/# ps
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
  355 pts/0    00:00:00 ps
```

**結果**
`PID` の 1 は `bash`

### 上記の例から分かること

- コンテナはある一つのコマンドを実行するために起動している。(PID 1)
- 複数のコンテナの`PID=1`は Linux の namespace 機能により衝突しない。

コンテナはメインプロセス(PID=1)を実行するために起動する



## コンテナに接続
- **`container exec`コマンド**:
コンテナ内で実行するLinuxコマンドを命令
`docker container exec [option] <container> command`

| オプション          | 意味                              | 用途                      |
|---------------------|-----------------------------------|---------------------------|
| -i, --interactive   | コンテナの標準入力に接続する       | コンテナを対話操作する     |
| -t, --tty           | 擬似ターミナルを割り当てる         | コンテナを対話操作する     |

### `container exec` の役に立つ場面
- コンテナの中にあるログを調べたい
- Dockerfile を書く前にbash でインストールコマンドを試し打ちしたい
- MySQL データベースサーバのクライアント mysql を直接操作した

## イメージの基礎

基本的にDockerHubで探す。
`Tags`のタブから都合の良いバージョンや構成内容を選び、記載してあるコマンドで取得・実行する。

### イメージの取得
`docker pull`で取得(`docker container run`で取得するため基本的にpull単体で使用はしない。)
```
docker image pull [option] <image>
```

### イメージの指定
```
docker container run [option] <image> [command]
```
- `container run`などのコマンドで、`<image>`を指定する場合、`IMAGE ID`や`REPOSITORY:TAG`形式などいくつかの方法で指定することができる。
※大体`REPOSITORY:TAG`を使用する。 例:`ubuntu 24.04`など

### 使用するタグの決め方
| スタンス                       | 構築フェーズ             | 保守フェーズ                              |
|--------------------------------|--------------------------|-------------------------------------------|
| 常に新しいものを使いたい       | 22.04 や 21.10 を指定     | 最新発表があったら手動で更新               |
| 常に LTS を使いたい            | latest を指定            | latest の実体が更新されるのに従う           |
| 常に今の LTS を使いたい        | 20.04 を指定             | ここから変更しない                        |

- **ホストマシンのイメージの確認**: `docker image ls`

対応アーキテクチャ、レイヤーはDocker Hubで確認。
`CMD["bash"]`: デフォルト命令は`bash`



## Dockerfileの基礎

### イメージのビルド
`docker image build [option] <path>`

| オプション         | 意味                       | 用途                          |
|--------------------|----------------------------|-------------------------------|
| -f, --file         | Dockerfile を指定する       | 複数の Dockerfile を使い分ける |
| -t, --tag          | ビルド結果にタグをつける   | 人間が把握しやすいようにする   |

### イメージのレイヤーを確認
```
docker image history [option] <image>
```

DockerHubにある公式イメージには軽量にするためレイヤーが最低限しか積み上がっていない。そのため、あらかじめ必要なセットアップを済ませたイメージを自分で作成しておくこと。
Dockerfileは既存のイメージ(レイヤーたち)に追加でレイヤーを乗せることができる。そのため、OS設定などをせず簡単にイメージを作成することができる。
例: Ubuntuコンテナ -> `vi`も`curl`も入っていない。

### Dockerfileの基本命令
| 命令 | 効果                               |
|------|------------------------------------|
| FROM | ベースイメージを指定する           |
| RUN  | 任意のコマンドを実行する           |
| COPY | ホストマシンのファイルをイメージに追加する |
| CMD  | デフォルト命令を指定する           |


### FROM: ベースイメージを指定
`FROM`はベースになるイメージを指定する命令。
これから `ubuntu:24.04`のレイヤーの上にレイヤーを乗せていく
```
FROM ubuntu:24.04
```

### RUN: 任意のコマンドを実行
`RUN` は Linux のコマンドを実行してその結果をレイヤーとする命令。
`RUN`によって「コンテナを起動するたびに`vi`をインストールする」という手間が省ける。

```
FROM ubuntu:24.04

RUN apt update
RUN apt install -y vim
```

RUN で apt を使うのか、それとも yum などほかのパッケージマネージャを使うのかは、ベースイメージによって決まる。
ubuntu:20.04 は当然 Ubuntu ですが、nginx:latest のようなイメージは一度 bash で起動して OS やパッケージマネージャが何かを調べておくのが基本です。


### COPY: ホストマシンのファイルをイメージに追加
`COPY`はホストマシンのファイルをイメージに追加する命令。
例: `ubuntu"24.04`イメージに行番号を表示する設定を記した　`.vimrc`を配置するには、まずホストマシンに`.vimrc`を作成する。

**.vimrc**
```
set number
```

**Dockerfile**
```
FROM ubuntu:24.04

RUN apt update
RUN apt install -y vim

COPY .vimrc /root/.vimrc
```

### CMD: デフォルト命令を指定
`CMD`はイメージのデフォルト命令を設定する命令。
例: `bash`の起動する汎用イメージではなく、特定のフォーマットで現在時刻を表示するイメージにする
```
FROM ubuntu:24.04

RUN apt update
RUN apt install -y vim

COPY ./vimrc /root/.vimrc

CMD date +"%Y/%m/%d %H:%M:%S ( UTC )"
```

### ディレクトリの確認
```
$ tree -a .

.
|-- .vimrc
`-- Dockerfile
```


### イメージのビルド
```
docker image build [option] <path>
```

### 実行コマンド
- `option`には`--tag`コマンドを指定して`my-ubuntu:date`というタグをつける。(`tag`をつけない場合buildを行うとランダム文字列の`IMAGE ID`でしか指定できなくなってしまうため。)
- `path`は`COPY`に使うファイルがあるディレクトリ`.`を指定。

```
$ docker image build     \
    --tag my-ubuntu:date \
    .
```

#### 以下のように出力されていれば成功
```
 => => writing image sha256:b2e8e19bd8760320b61eae81b79231ae0f80d9f1b3573b05e34931672ecc0879
```

### デフォルト命令でコンテナを起動し、意図した通りのイメージになっているか確認
```
% docker container run \
--name my-ubuntu1 \
--rm \
my-ubuntu:date

2024/08/03 02:41:17 ( UTC )
```
※`CMD`によるデフォルト命令の指定が意図通りになっていることを確認

### `RUN`と`COPY`の結果を確認するために指定命令`vi`でコンテナを起動

```
% docker container run \
> --name my-ubuntu2 \
> --rm \
> --interactive \
> --tty \
> my-ubuntu:date \
> vi
```
※vimの画面になり、各行に数字が表示されれば成功

### エラー対処
#### イメージが改ざんされていないか等を検証する機能が作動するためコンテナ起動できない。
- エラーメッセージ
`docker: you are not authorized to perform this operation: server returned 401.`というメッセージが表示された場合。

#### 解決策
以下のコマンドで`Docker Content Trust(DCT)`を無効化する。
`export DOCKER_CONTENT_TRUST=0`


### Dockerを複数扱う場合
