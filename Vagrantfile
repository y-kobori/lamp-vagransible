Vagrant.configure("2") do |config|
  # OS
  config.vm.box = "ubuntu/bionic64"

  # メモリ (デフォルト: 1024MB | 変更する場合のみ指定)
  # config.vm.provider "virtualbox" do |vm|
  #   vm.memory = 1024
  # end

  # ポートフォワーディング
  config.vm.network "forwarded_port", guest: 3306, host: 3306, auto_correct: true
  config.vm.network "forwarded_port", guest: 80, host: 80, auto_correct: true

  # vagrantに割り振るIPアドレス
  # 自動割り当て設定
  config.vm.network "private_network", type: "dhcp"
  # 固定IPアドレス設定
  # config.vm.network "private_network", ip: "192.168.33.10"

  # ディレクトリ共有設定
  # デフォルトの共有ディレクトリの無効化
  config.vm.synced_folder ".", "/vagrant", disabled: true
  # ansible用ファイル群
  config.vm.synced_folder "./ansible", "/home/vagrant/ansible", create: true

  # アプリケーションのソースコード

  # =====================================================================================
  # VirtualBox provider
  # https://www.vagrantup.com/docs/synced-folders/virtualbox
  # (RSync/NFSプロバイダのブロックをコメントアウト + このブロックをコメントアウト解除して使用)
  # =====================================================================================
  config.vm.synced_folder "./www", "/var/www/html", create: true, mount_options: ["dmode=777,fmode=777"]

  # =====================================================================================
  # RSync provider
  # https://www.vagrantup.com/docs/synced-folders/rsync
  # (VirtualBox/NFSプロバイダのブロックをコメントアウト + このブロックをコメントアウト解除して使用)
  # =====================================================================================
  # config.vm.synced_folder "./www", "/var/www/html", create: true, mount_options: ["dmode=777,fmode=777"],
  #   type: "rsync",
  #   rsync__exclude: [
  #     ".git/",
  #     "node_modules/",
  #     "vendor/"
  #   ]

  # =====================================================================================
  # NFS provider
  # https://www.vagrantup.com/docs/synced-folders/nfs
  # (VirtualBox/RSyncプロバイダのブロックをコメントアウト + このブロックをコメントアウト解除して使用)
  # =====================================================================================
  # config.vm.synced_folder "./www", "/var/www/html", create: true,` mount_options: ["rw","vers=3","tcp"],
  #   type: "nfs",
  #   linux__nfs_options: ["rw","no_subtree_check","all_squash","async"]`

  # ansibleによるプロビジョニング
  config.vm.provision "ansible_local" do |ansible|
    ansible.provisioning_path = "/home/vagrant/ansible"
    ansible.playbook = "playbooks/site.yml"
    ansible.inventory_path = "/home/vagrant/ansible/local"
    ansible.limit = "all"
    ansible.galaxy_role_file = "requirements.yml"
    ansible.galaxy_roles_path = "/etc/ansible/roles"
    ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
  end
end
