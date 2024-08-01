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


## Dockerとは
- **Docker Engine**:
コンテナ型仮想化ソフトウェアのことで、これによりアプリケーションをソフトウェアとして扱うことができる。
DockerのコンテナはLinuxのカーネルと機能を使用しているため、Docker EngineはLinux上でしか動かない。
- **Docker CLI**: Dockerに命令をするコマンド
- **Docker Desktop**: Docker EngineやLinuxカーネルが含まれているため、Linux以外のOSからでもDocker Engineを動かすことができる。
- **Docker Compose**: Docker CLIをまとめて実行してくれる便利なツール。
- **Docker Hub**: DockerのイメージレジストリであるSaasサービス。
- **ECS/GKE**: コンテナ管理サービス。Docker Engineの入ったLinuxのことで、ローカル開発に使ったコンテナをそのままデプロイできる場所のこと。
- **ECR/GCR**: 非公開のイメージレジストリ。プライベートなDocker Hubのことで、「商用サービスのイメージをDocker Hubで公開したくないけど、レジストリに登録しないとデプロイできない」場合に使用される。
- **Kubernetes**: 多数のコンテナを管理するオーケストレーションソフトウェア。ロードバランサーの作成や、アクセス集中時のスケーリングなどが簡単に実現できる

## Dockerを理解
### コンテナ
特定のコマンドを実行するために作られるホストマシン上の隔離された領域。
コンテナの実体はLinuxのnamespaceという機能により他と分離されたただの1プロセス。そのプロセスをコンテナやイメージというものを用いて扱いやすくした技術がDocker

　- **特徴**
 1.コンテナはイメージをもとに作られる
 2. 　DockerのCLIやAPIを使って、生成や起動や停止を行うことができる。
 3. 複数のコンテナは違いに独立していて影響できず、独自に動作する。
 4. Docker Engineの上ならローカルマシンでも仮想マシンでもクラウド環境でも動かせる。

 ### イメージ
 コンテナの実行に必要なパッケージで、ファイルやメタ情報を集めたもの。
 - ベース -> Ubuntu
 - インストールするもの -> PHP
 - 環境変数
 - 設定ファイルの配置内容 -> PHPの設定ファイル
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

| オプション                 | 意味                           | 用途                                            |
|----------------------------|--------------------------------|-------------------------------------------------|
| -i, --interactive          | コンテナの標準入力に接続する   | コンテナを対話操作する                          |
| -t, --tty                  | 擬似ターミナルを割り当てる     | コンテナを対話操作する                          |
| -d, --detach               | バックグラウンドで実行する     | ターミナルが固まるのを避ける                    |
| --rm                       | 停止済コンテナを自動で削除する | 起動時に停止済コンテナと一意な情報が衝突するのを避ける |
| --name                     | コンテナに名前をつける         | コンテナを指定しやすくする                      |
| --platform                 | イメージのアーキテクチャを明示する | M1 Mac で必要な場合がある                        |

### コンテナの対話操作
```
$ docker container run \
    --interactive      \
    --tty              \
    ubuntu:24.04
```
- **短縮バージョン**: `$ docker run -it ubuntu:20.04`

**bash内**
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
＊＊command**: 任意の命令を実行させる。commandなしの場合、デフォルトの命令が実行される。

#### **例１**: `bash` -> bashで起動(デフォルト命令ではnginxの起動のみ)



##　コンテナの状態遷移
### コンテナとプロセスについて

### **例1: Ubuntuコンテナをデフォルト命令(bash)で起動**
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

**結果＊＊
Ubuntu コンテナを起動し、デフォルト命令の `bash` を使っている状態で `ps` コマンドでコンテナ内のプロセス一覧を確認すると、PID の 1 は `bash` になっています。

### **例2**: Nginxコンテナをデフォルト命令(nginx)で起動

- nginxを起動
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

- コンテナ内にpsコマンドがないため、コンテナ内で実行しインストールを行う
```
apt update
apt install -y procps
```

**結果**
`PID` の 1 は `nginx` の起動コマンド
`ps`自身と`ps`を実行するために`container exec`で起動したbashのプロセスも存在する。

```
root@a3f678198bab:/# ps x
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 nginx: master process nginx -g daemon off;
   43 pts/0    Ss     0:00 bash
  397 pts/0    R+     0:00 ps x
```

### **例3**: Nginxコンテナを指定命令(bash)で起動
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
- 複数のコンテナの`PID=1`はLinuxのnamespace機能により衝突しない。

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

