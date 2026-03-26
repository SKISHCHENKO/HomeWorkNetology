# Домашнее задание к занятию 2. «Применение принципов IaaC в работе с виртуальными машинами»


## Задача 1
Установите на личный Linux-компьютер или учебную локальную ВМ с Linux следующие сервисы(желательно ОС ubuntu 20.04):

VirtualBox,  
Vagrant, рекомендуем версию 2.3.4  
Packer версии 1.9.х + плагин от Яндекс Облако по инструкции  
уandex cloud cli Так же инициализируйте профиль с помощью yc init .  
Примечание: Облачная ВМ с Linux в данной задаче не подойдёт из-за ограничений облачного провайдера. У вас просто не установится virtualbox.  

## Решение 1

1. VirtualBox  
sudo apt update  
sudo apt install -y virtualbox  
VBoxManage --version  

2. Vagrant (версия 2.3.4)  
wget https://releases.hashicorp.com/vagrant/2.3.4/vagrant_2.3.4-1_amd64.deb  
sudo apt install ./vagrant_2.3.4-1_amd64.deb  
--version  

3. Packer 1.9.x  
wget https://releases.hashicorp.com/packer/1.9.4/packer_1.9.4_linux_amd64.zip  
unzip packer_1.9.4_linux_amd64.zip  
sudo mv packer /usr/local/bin/  
packer version  

4. yc CLI (Yandex Cloud)
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh 
yc init  


## Задача 2

Убедитесь, что у вас есть ssh ключ в ОС или создайте его с помощью команды ssh-keygen -t ed25519  
Создайте виртуальную машину Virtualbox с помощью Vagrant и Vagrantfile в директории src.  
Зайдите внутрь ВМ и убедитесь, что Docker установлен с помощью команды:  
docker version && docker compose version  
Если Vagrant выдаёт ошибку (блокировка трафика):  
URL: ["https://vagrantcloud.com/bento/ubuntu-20.04"]       
Error: The requested URL returned error: 404:  
Выполните следующие действия:  

Используйте зеркало файл-образ "bento/ubuntu-24.04".  
Важно:  

Если ваша хостовая рабочая станция - это windows ОС, то у вас могут возникнуть проблемы со вложенной виртуализацией. Ознакомиться со cпособами решения можно по ссылке.  

Если вы устанавливали hyper-v или docker desktop, то все равно может возникать ошибка:  
Stderr: VBoxManage: error: AMD-V VT-X is not available (VERR_SVM_NO_SVM)  
Попробуйте в этом случае выполнить в Windows от администратора команду bcdedit /set hypervisorlaunchtype off и перезагрузиться.  

Если ваша рабочая станция в меру различных факторов не может запустить вложенную виртуализацию - допускается неполное выполнение(до ошибки запуска ВМ)  

## Решение 2

1. SSH ключ
ssh-keygen -t ed25519  
~/.ssh/id_ed25519  
~/.ssh/id_ed25519.pub  

2. Структура проекта  
mkdir src  
cd src  

1. Vagrantfile  
Vagrant.configure("2") do |config|  
  config.vm.box = "bento/ubuntu-24.04"  
  config.vm.provider "virtualbox" do |vb|  
    vb.memory = 2048  
    vb.cpus = 2  
  end  

  config.vm.network "private_network", type: "dhcp"  

  config.vm.provision "shell", inline: <<-SHELL  
    apt-get update  
    apt-get install -y ca-certificates curl gnupg lsb-release  

    mkdir -p /etc/apt/keyrings  
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \  
      gpg --dearmor -o /etc/apt/keyrings/docker.gpg  

    echo \  
      "deb [arch=$(dpkg --print-architecture) \  
      signed-by=/etc/apt/keyrings/docker.gpg] \  
      https://download.docker.com/linux/ubuntu \  
      $(lsb_release -cs) stable" \  
      > /etc/apt/sources.list.d/docker.list  

    apt-get update  
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin  

    usermod -aG docker vagrant  
  SHELL  
end  

4. Запуск  
vagrant up  


## Задача 3

Отредактируйте файл mydebian.json.pkr.hcl или mydebian.jsonl в директории src (packer умеет и в json, и в hcl форматы):  
добавьте в скрипт установку docker. Возьмите скрипт установки для debian из документации к docker,  
дополнительно установите в данном образе htop и tmux.(не забудьте про ключ автоматического подтверждения установки для apt)  
Найдите свой образ в web консоли yandex_cloud  
Необязательное задание(*): найдите в документации yandex cloud как найти свой образ с помощью утилиты командной строки "yc cli".  
Создайте новую ВМ (минимальные параметры) в облаке, используя данный образ.  
Подключитесь по ssh и убедитесь в наличии установленного docker.  
Удалите ВМ и образ.  
ВНИМАНИЕ! Никогда не выкладываете oauth token от облака в git-репозиторий! Утечка секретного токена может привести к финансовым потерям. После выполнения задания обязательно удалите секретные   данные из файла mydebian.json и mydebian.json.pkr.hcl. (замените содержимое токена на "ххххх")  
В качестве ответа на задание загрузите результирующий файл в ваш ЛК.  

## Решение 3

packer {  
  required_plugins {  
    yandex = {  
      source  = "github.com/yandex-cloud/yandex"  
      version = ">= 1.0.0"  
    }  
  }  
}  

source "yandex" "debian" {  
  token               = "xxxxx"  
  cloud_id            = "xxxxx"  
  folder_id           = "xxxxx"  
  zone                = "ru-central1-a"  
  image_family        = "debian-11"  
  ssh_username        = "debian"  
  use_ipv4_nat        = true  
}  

build {  
  sources = ["source.yandex.debian"]  

  provisioner "shell" {  
    inline = [  
      "apt-get update",  
      "apt-get install -y ca-certificates curl gnupg lsb-release htop tmux",  

      # Docker install  
      "mkdir -p /etc/apt/keyrings",  
      "curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg",  

      "echo \"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable\" > /etc/apt/sources.list.d/docker.list",  

      "apt-get update",  
      "apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin"  
    ]  
  }  
}  


packer init .  
packer build mydebian.pkr.hcl  
yc compute image list  

yc compute instance create \  
  --name test-vm \  
  --zone ru-central1-a \  
  --create-boot-disk image-id=<***> \  
  --public-ip \  
  --ssh-key ~/.ssh/id_ed25519.pub  

Vagrant — поднимает ВМ локально  
Packer — делает golden image  
Docker — runtime  
yc CLI — управление облаком  
Всё это = Infrastructure as Code  