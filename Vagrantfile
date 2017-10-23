# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "minimal/xenial64"

  # for PostgreSQL
  config.vm.network :forwarded_port, guest:5432, host:5432
  # for Redis
  config.vm.network :forwarded_port, guest:6379, host:6379
  # for Redash
  config.vm.network :forwarded_port, guest:5000, host:5000

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.ssh.insert_key = false

  config.vm.provision "shell", inline: <<-SHELL
    update-locale LANG=C.UTF-8 LANGUAGE=
    sed -i.bak -e 's!http://\\(archive\\|security\\).ubuntu.com/!ftp://ftp.jaist.ac.jp/!g' /etc/apt/sources.list
    apt update

    # PostgreSQL
    apt install -y postgresql postgresql-contrib
    sudo -u postgres createuser -DRs vagrant
    sudo -u postgres psql -c "ALTER USER vagrant WITH PASSWORD 'db1234'"
    sudo -u postgres createdb -O vagrant vagrant
    echo "listen_addresses = '*'" >> /etc/postgresql/9.5/main/postgresql.conf
    echo "host all vagrant 0.0.0.0/0 md5" >> /etc/postgresql/9.5/main/pg_hba.conf
    systemctl restart postgresql

    # Redis
    apt install -y redis-server
    sed -i.bak -e 's!^bind !#bind!' /etc/redis/redis.conf
    systemctl restart redis

    # Docker
    apt install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    apt-key fingerprint 0EBFCD88
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    apt update
    apt install -y docker-ce
    adduser vagrant docker

    # docker-compose
    curl -fsSL https://github.com/docker/compose/releases/download/1.16.1/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose
    chmod 0755 /usr/local/bin/docker-compose

    # Redash with docker
    sudo -u vagrant ln -s /vagrant/redash-compose.yml docker-compose.yml
    sudo -u vagrant docker-compose pull
    sudo -u vagrant docker-compose run --rm server create_db
    sudo -u vagrant docker-compose up -d
  SHELL
end
