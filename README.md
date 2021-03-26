# lamp-vagransible

## 概要

このリポジトリはVagrant + AnsibleによってLAMPの開発環境を構築するためのリポジトリです。

## ミドルウェア

* PHP: 7.4
* MySQL: 5.7
* Apache: 2.4
* Node.js: 14.15.1

## ディレクトリ

* `ansible` → Ansible関連ファイル
* `www` → プログラムを配置するディレクトリ

## 環境構築

### 事前インストールが必要なツール

* [Vagrant](https://www.vagrantup.com/)
* [VirtualBox](https://www.virtualbox.org/)

### 手順

1. `Vagrantfile` を作成

    `cp Vagrantfile.example Vagrantfile`

2. vagrant起動

    `vagrant up`

    ※初回起動時はvagrantの起動と一緒にプロビジョニング実行

3. vagrantに接続

    `vagrant ssh`

    ※手順4〜はvagrantに接続した状態で実行

4. 各種ミドルウェアがインストールされているか確認

    以下のコマンドで各ミドルウェアのバージョンを確認

    `php -v`  
    `composer --version`  
    `mysql --version`  
    `apachectl -V`  
    `node -v`

5. プログラムを配置

    `www` ディレクトリ以下にプログラムを配置すると、apacheのドキュメントルートに同期されるため、  

    `localhost` +  `www` ディレクトリからの相対パスでアクセスが可能です。

6. 動作確認

    6-1. hostsファイルにApacheに設定したVirtualHostのドメインを追記

      `127.0.0.1 dev.sample-app.com`

      ※VirtualHostの設定は `ansible/group_vars/local/_web/apache.yml` を参照

    6-2. ブラウザからアクセス

      * ドキュメントルート: [http://localhost](http://localhost)
      * VirtualHostのドメイン: [http://dev.sample-app.com](http://dev.sample-app.com)

      ※ポートフォワーディングの衝突回避(後述)でホストマシン側がポート80以外にマッピングされている場合はポート番号をURLに追加してアクセス

## TIPS

### vagrantで動作しているMySQLへの接続

vagrant内のMySQLが動作している `3306` ポートをポートフォワーディングするよう設定しているため、外部のMySQLクライアントツールなどから接続が可能です。  
接続する際は `ansible/group_vars/local/_db/mysql.yml` に設定されているデータベース情報を使用してください。  
デフォルトは以下の設定となっています。

* Host: `127.0.0.1`
* Port: `3306`
  * 後述のポートフォワーディングの `auto_correct` オプションにより変わる可能性あり
* Database: `app`
* Users
  * DB操作ユーザ
    * User: `db_user`
    * Password: `db_pass`
  * rootユーザ
    * User: `root`
    * Password: `root_pass`

※vagrantへのssh経由で接続設定しておくと `auto_correct` の動作に関わらず `3306` ポートで接続可能

### ポートフォワーディングの衝突回避

`Vagrantfile` のポートフォワーディング設定で `auto_correct` オプションを有効にしているため、ホストマシン側で使用するポート( `80` , `3306` )が既に使用されている場合に別のポート番号が自動的に割り当てられます。  
実際に割り当てられたポートは `vagrant up` 時に出力された値を確認するか、 起動後に `vagrant port` を実行することで確認できます。

参考: [Vagrant - Networking > Forwarded Ports > Option Reference > auto_correct](https://www.vagrantup.com/docs/networking/forwarded_ports#auto_correct)

### プロビジョニング実行

`vagrant up` 時にプロビジョニングが行われるのは初回だけのため、環境構築後に再度プロビジョニングを行う場合は以下のコマンドをホストマシン上で実行してください。

`vagrant provision`

※起動と同時にプロビジョニングを行う場合は `vagrant up --provision`

参考: [Vagrant - Commands (CLI) > provision](https://www.vagrantup.com/docs/cli/provision)

### `Vagrantfile` 配置ディレクトリ以外からvagrantへの接続

以下の対応をしておくと `Vagrantfile` が配置されているディレクトリ以外でもvagrantへの接続が可能になります。

`vagrant ssh-config --host <ホスト名> >> ~/.ssh/config` でssh設定ファイルに情報を追記  
`ssh <ホスト名>` で接続

※ `~/.ssh/config` が存在しない場合は `touch ~/.ssh/config` などで作成してから実行

参考: [Vagrant - Commands (CLI) > ssh-config](https://www.vagrantup.com/docs/cli/ssh_config)

### アプリケーションの高速化

#### vagrantのファイル同期について

vagrantの起動にVirtualBoxを利用している場合、ホストマシン - vagrant間のファイル同期にはVirtualBoxの共有フォルダ機能が利用されます。  
VirtualBoxの共有フォルダ機能を使用している場合はvagrant内でのディスクアクセスが遅く、アプリケーションの動作速度が多少遅くなります。  
そこで、vagrantの共有フォルダ設定で指定可能なRSyncやNFSを利用することで、ディスクアクセス速度によるパフォーマンス低下の影響を軽減することができます。

#### RSync, NFSを利用するための設定変更

`Vagrantfile` の `config.vm.synced_folder` でアプリケーションのソースコードのディレクトリ共有設定を行っている箇所を変更します。  
変更箇所については `Vagrantfile` 内のコメントアウトに記述しているため、そちらを参照してください。

#### RSyncを利用する際の注意

RSyncを利用する場合、ホストマシン - vagrant間でディレクトリがマウントされて同期されるわけではなく、ホストマシン → vagrantへ単方向でファイルが転送されます。  
そのため、vagrant内で変更されたファイルはホストマシン側へ反映されず、Laravelのログファイルなどはvagrant内で確認する必要があります。  
`vagrant up` , `vagrant reload` , `vagrant rsync` 実行時にRSyncが動作してファイル転送が実行されます。  
`vagrant rsync-auto` を実行しておくと、ホストマシン側の転送対象ファイルを監視して変更を検知した場合に自動で転送してくれるため、開発作業中はこちらのコマンドの実行を推奨します。

##### 参考

* [Vagrant - Synced Folders > RSync](https://www.vagrantup.com/docs/synced-folders/rsync)
* [Vagrant - Commands (CLI) > rsync](https://www.vagrantup.com/docs/cli/rsync)
* [Vagrant - Commands (CLI) > rsync-auto](https://www.vagrantup.com/docs/cli/rsync-auto)

#### NFSを利用する際の注意

Windowsでは標準ではNFSを使用できず、 [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd)というプラグインをインストールする必要があるようです。  
また、NFSを使用するとvagrant起動時にホストマシンのパスワード入力が要求されますが、その場合は以下の記事の対応を行うと入力する必要がなくなります。  

[【設定方法】VagrantのでNFSマウントを利用する際に求められる管理者パスワード入力を省略する設定](https://t-cr.jp/memo/1ae163834b866d3b8)

##### 参考

* [Vagrant - Synced Folders > NFS](https://www.vagrantup.com/docs/synced-folders/nfs)
