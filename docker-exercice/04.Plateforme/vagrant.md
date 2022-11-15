Dans cet exercice vous allez utiliser [Vagrant](https://vagrantup.com) pour créer une machine virtuelle et installer Docker sur celle-ci.

## Quelques mots sur Vagrant

[Vagrant](https://vagrantup.com) est un excellent outil de [Hashicorp](https://hashicorp.com), il est disponible pour Windows, Linux et Mac et permet de lancer des machines virtuelles Linux sur VirtualBox, Hyper-V, VMWare, ...

Vagrant repose sur la définition d'un fichier *Vagrantfile* dans lequel sont précisés les caractéristiques de la VM (ou des VMs) à provisionner et configurer. 

L'installation de Vagrant est très simple, il suffit de récupérer le binaire depuis le site officiel et de mettre celui-ci dans votre PATH. En fonction de votre environnement, il pourrait être nécessaire d'installer l'hyperviseur [VirtualBox](https://virtualbox.org) qui sera utilisé par Vagrant.

## Création d'une VM

Dans un nouveau dossier, créez le fichier nommé *Vagrantfile* avec le contenu suivant:

```
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.network "private_network", ip: "192.168.33.11"
  config.vm.provision "shell", inline: <<-SHELL
    curl -sSLf https://get.docker.com | sh
    sudo usermod -aG docker vagrant
  SHELL
end
```

Ce fichier précise qu'une VM basée sur l'image *hashicorp/bionic64* (une déclinaison de Ubuntu) sera créée et que l'adresse IP local *192.168.33.11* lui sera attribuée. Une fois cette VM créée, Docker sera installé dans celle-ci.

Utilisez la commande suivante afin de créer cette VM:

```
$ vagrant up
```

Lancez ensuite un shell dans cette VM:

```
$ vagrant ssh
```

et vérifiez que Docker a bien été installé, en utilisant la sous commande *info*:

```
vagrant@vagrant:~$ docker info
```

Vous devriez obtenir un résultat similaire à celui ci-dessous:

```
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.5.1-docker)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.6
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc io.containerd.runc.v2 io.containerd.runtime.v1.linux
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 05f951a3781f4f2c1911b05e61c160e9c30eaa8e
 runc version: 12644e614e25b05da6fd08a38ffa0cfe1903fdec
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.15.0-58-generic
 Operating System: Ubuntu 18.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 985.5MiB
 Name: vagrant
 ID: EVC2:HHMI:A7C5:HY7I:TKAB:QIYD:3TH6:SRYZ:337P:ZL5P:FJVX:SDCQ
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: No swap limit support
```

## Cleanup

La VM créée précédemment peut être supprimer en utilisant la commande suivante:

```
$ vagrant destroy -f
```