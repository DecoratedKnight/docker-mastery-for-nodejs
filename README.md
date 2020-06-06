## Section 2: Docker Compose Basics
composeのバージョンについて
v2 ... シングルノードの開発 / テスト
v3 ... 複数ノードのオーケストレーション
→ Swarm / k8s などを使わないなら v2 でも大丈夫(でも別に3にしないメリットないとも個人的には思う)

docker-compose exec のあとはサービス名を指定する
ex ) docker-compose exec web sh
docker container run の時のように -it を付けなくていい

イメージをビルドし直すときは --build をつける
Dockerfileを更新したからといって勝手に再ビルドされる訳ではない

## Section 3: Node Dockerfile Best Practice Basics
### Dockerfile best practice
CMDではnpmではなくnodeで起動した方がいいらしい
- npm から nodeが起動する、というワンステップを踏むから
- 実際に何をしているかDockerfileでわかりやすいから
- でも nodeは initかPID 1 ではうまく動かない

https://text.superbrothers.dev/200328-how-to-avoid-pid-1-problem-in-kubernetes/
でもここ見るとnpm経由した方が良さそうな楽な気も

https://www.creationline.com/lab/29422
BretFisherの記事

https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md
nodejs公式

https://www.m3tech.blog/entry/2018/11/26/160000
やっぱりnpm経由はダメみたい

mkdirしないで WORKDIR をすればいい
ディレクトリがなければ勝手に作る、所有者もいい感じに
と思ったけどrootじゃないユーザーのときは自分で作って所有権変える必要出てくるかも

### BaseImageについて
バージョンを固定する。latestは使わない
alpine推しでslimに否定的だけど、上の記事では逆
→ 意見が変わったのかな

### node User
nodeユーザーを使おう
npm i -g やパッケージ周りはrootの時にいれる
npm i （ローカル）はnodeになってから

Dockerfile
USER node ... ユーザー切り替え
RUN mkdir app && chown -R node:node . ... パーミッション変更

ユーザーを切り替えていてもWORKDIRやCOPYはrootで走る
USERが影響するのは RUN ENTRYPOINT CMD の3つ

rootユーザーでコンテナに入りたかったりするときは -u root オプションをつけるとできる

## Section 4: Controlling The Node Process In Containers
プロセス管理(nodemon, forever, pm2)は本番稼働に必要ない
Dockerがアプリの起動などをするから
ただし開発用途で使うことはある（watchなどで）

### アプリ正常終了のために
PID1問題は大きく二つ
・ゾンビプロセス ... これはnodeではあまり問題にならないらしい
・正常終了
→ SIGINT / SIGTERMに反応しないといけない（SIGKILLはコンテナまで届かないので関係ない）
→ npmは上記に反応しない
→ nodeならハンドリングするコードがかける

方法
- --init オプションをコンテナ起動時に使う
- tini をイメージに追加する
- SIGINTをハンドリングするコードを書く

## Section 5: Advanced Dockerfiles with Multi-stage and BuildKit
マルチステージビルドでどのステージをビルドするかは --target オプションを使う

## Section 6: Node Apps in Cloud Native Docker
12factorをよく知る
設定は環境変数で渡す
ログは標準出力に出す

bindmountするときコンテナ側のディレクトリは作らなくても大丈夫
DockerfileのVOLUME指定もいらない

VOLUMEはなんか永続化したいのがわかりきってる時に使えばいいっぽい、container run -v の時にボリューム指定しなくても作られるので

## Section 7: Compose for Awesome Local Development
Docker for Mac ではマウント遅い問題がある
マウントオプションにcached or delegated を指定すると改善することができる
https://docs.docker.com/docker-for-mac/osxfs-caching/
https://www.to-mega-therion.net/docker/docker-mounted-volume-slow

node_modulesをホスト側からコンテナにコピーしないこと
→ OS依存のバイナリとかがある（node-gyp）
→ 重くなる
.dockerignoreを必ず書こう

上記の理由で、node_modulesをbind-mountできない（host側がmac / winの場合）
→ 1 ) ホスト側で npm installをしないで、composeでやる
→ 2 ) image側のmoduleを移動して、ホスト側から隠す

1 ) 
コンテナ側の空ではないディレクトリをbind-mountした場合、ホスト側の情報で上書きされる
したがってホスト側にnode_modulesがない状態で、ディレクトリ丸ごとbind-mountしたら、
Dockerfileでimage側で先にnpm iするように書いてあったとしてもnode_modulesがない状態のコンテナが立ち上がる
→ docker-compose run express npm i として、サービス側でインストールすればいい。それからdocker-compose up
→ bind-mount済なので、host側にnode_modulesが出てくるはず（中身はコンテナ側で実行されたもの）

2 ) node_modulesは親の階層を辿って探しに行くことを利用して、npm iを一つ上の階層で実行する
そしてその他のソースコードは一階層深いところに置く
また、ホスト側にnode_modulesがあったりしたらbind-mountされてしまうので、それを無名volumeと結び付けておくと
コンテナ側では空になるので、共存できる
当然 .dockerignoreで node_modulesは対象に入れておく（COPYした時にコンテナ側に渡らないように）
こちらの方が柔軟性はあるが、ライブラリが対応してないとできない

docker-compose.yml のcommand で、コンテナ起動時のコマンドを上書きできる
開発時にnodemonなどの起動を仕込めばwatchできる

docker-composeのdepends_onは起動の順番を指定するのであって、readyかどうかはわからない
v2だと conditionをつけて ヘルスチェックをできるが、v3だとそれがない
(なのでローカル開発用途ならv2を推しているっぽい)
https://docs.docker.com/compose/compose-file/compose-file-v2/#depends_on
https://docs.docker.com/compose/compose-file/#depends_on
https://docs.docker.com/compose/startup-order/

マイクロサービス開発時の問題
・エンドポイントめっちゃあってそれぞれの名前解決が必要
→ Nginx/HAProxy/Traefik などのプロキシサーバーで x.localhost にルーティングしちゃう (chrome)
→ vcap.me xip.io のようなワイルドカードドメインを使う
→ dnsmasq を使う
→ hostsを自分で編集する

・CORS
→ * ヘッダーつけたプロキシ

・ローカルのHTTPS
https://letsencrypt.org/docs/certificates-for-localhost/

・リモートデバッグのやり方を別途勉強したい