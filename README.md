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