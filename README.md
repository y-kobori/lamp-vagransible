# lamp_vagransible

## About
__vagransible = vagrant + ansible__

vagrant + ansibleによりLAMP環境を構築するためのリポジトリです。

## Get started
### ansibleのroleをインストール
`ansible-galaxy install -r requirements.yml -p roles`

### vagrantを起動
`vagrant up`

### 起動中のvagrant環境に向けてプロビジョニングを実行
`vagrant provision`

## License
MIT
