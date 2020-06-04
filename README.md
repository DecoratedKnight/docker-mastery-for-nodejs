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
- initかPID 1 ではうまく動かないから

https://text.superbrothers.dev/200328-how-to-avoid-pid-1-problem-in-kubernetes/
でもここ見るとnpm経由した方が良さそうな楽な気も

https://www.creationline.com/lab/29422
BretFisherの記事

https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md
nodejs公式

mkdirしないで WORKDIR をすればいい
ディレクトリがなければ勝手に作る、所有者もいい感じに

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
