# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "gitlab.local"

  config.vm.define :gitlab do |cfg|

  if Vagrant.has_plugin?("vagrant-vbguest")
    cfg.vbguest.auto_update = false
  end

   cfg.vm.network "forwarded_port", guest: 80, host: 8080
   cfg.vm.network "forwarded_port", guest: 22, host: 8022

   cfg.vm.network "private_network", ip: "192.168.33.44"
   cfg.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [".git/"]
  end

  config.vm.define :runner do |cfg|
    cfg.vm.box = "ubuntu/focal64"  # Ubuntu 이미지 선택
    cfg.vm.hostname = "runner.local"

    # Docker 실행자를 사용하는 경우 docker 컨테이너를 생성합니다.
    cfg.vm.provider "docker" do |docker|
      docker.image = "ubuntu:latest"  # GitLab Runner를 실행할 Docker 이미지
      docker.name = "gitlab-runner-container"  # 컨테이너 이름 설정 (원하는 대로 변경 가능)
      
      # 컨테이너에 포트 포워딩 설정 (runner와 GitLab 서버 사이의 통신을 위해 필요)
      docker.ports = ["8081:8081", "22:22"]  # 원하는 포트로 변경 가능
      
      # Docker 컨테이너에서 사용할 환경 변수 설정 (GitLab Runner에 대한 설정)
      docker.env = [
        "CI_SERVER_URL=http://localhost:8080/",
        "RUNNER_TOKEN=GR134894131qYZyiGvTsM-Z9xgahr",
        "RUNNER_EXECUTOR=docker"
      ]
    end
  end

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
    vb.name = "gitlab.local"
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y curl openssh-server ca-certificates

    debconf-set-selections <<< "postfix postfix/mailname string $HOSTNAME"
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    DEBIAN_FRONTEND=noninteractive sudo apt-get install -y postfix

    if [ ! -e /vagrant/gitlab-ce.deb ]; then
        wget --content-disposition -O /vagrant/gitlab-ce.deb
 https://packages.gitlab.com/gitlab/gitlab
ce/packages/ubuntu/jammy/gitlab-ce_15.9.3
ce.0_amd64.deb/download.deb
    fi
    sudo dpkg -i /vagrant/gitlab-ce.deb

    sudo gitlab-ctl reconfigure
  SHELL
end
