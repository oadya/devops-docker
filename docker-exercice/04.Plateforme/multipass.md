Dans cet exercice vous allez utiliser [Multipass](https://multipass.run) pour créer une machine virtuelle et installer Docker sur celle-ci.

## Quelques mots sur Multipass

Multipass est un utilitaire développé par Canonical, il permet de lancer des machines virtuelles Ubuntu facilement.

En fonction de l'OS, Multipass peut utiliser différents hyperviseurs:
- Hyper-V
- HyperKit
- KVM
- VirtualBox

En s'intégrant de manière native à ces hyperviseurs, il permet de démarrer des machines virtuelles très rapidement.

## Installation

Vous trouverez sur le site [https://multipass.run](https://multipass.run) la procédure d'installation de Multipass en fonction de votre OS.

![Multipass](./images/local/multipass.png)

## Commandes disponibles

La liste des commandes disponibles pour la gestion du cycle de vie des VMs peut être obtenue avec la commande suivante:

```
$ multipass
Usage: multipass [options] <command>
Create, control and connect to Ubuntu instances.

This is a command line utility for multipass, a
service that manages Ubuntu instances.

Options:
  -h, --help     Display this help
  -v, --verbose  Increase logging verbosity. Repeat the 'v' in the short option
                 for more detail. Maximum verbosity is obtained with 4 (or more)
                 v's, i.e. -vvvv.

Available commands:
  delete    Delete instances
  exec      Run a command on an instance
  find      Display available images to create instances from
  get       Get a configuration setting
  help      Display help about a command
  info      Display information about instances
  launch    Create and start an Ubuntu instance
  list      List all available instances
  mount     Mount a local directory in the instance
  networks  List available network interfaces
  purge     Purge all deleted instances permanently
  recover   Recover deleted instances
  restart   Restart instances
  set       Set a configuration setting
  shell     Open a shell on a running instance
  start     Start instances
  stop      Stop running instances
  suspend   Suspend running instances
  transfer  Transfer files between the host and instances
  umount    Unmount a directory from an instance
  version   Show version details
```

Nous allons voir quelques unes de ces commandes sur des exemples.

## Creation d'une machine virtuelle

La commande suivante permet de lancer une nouvelle machine virtuelle nommé *docker*

```
$ multipass launch -n docker
Launched: docker
```

Pour obtenir des informations sur cette VM, il suffit d'utiliser la commande suivante:

```
$ multipass info docker
Name:           docker
State:          Running
IPv4:           192.168.64.93
Release:        Ubuntu 20.04.2 LTS
Image hash:     04aabbbd5964 (Ubuntu 20.04 LTS)
Load:           0.08 0.16 0.08
Disk usage:     1.3G out of 4.7G
Memory usage:   138.9M out of 981.3M
```

Il est également possible de lister les VM existantes:

```
$ multipass list
Name                    State             IPv4             Image
docker                  Running           192.168.64.93    Ubuntu 20.04 LTS
```

## Installation de Docker

A l'aide de la commande suivante, lancez un shell dans la VM créée précédemment:

```
$ multipass shell docker
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-72-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Apr 30 20:01:34 CEST 2021

  System load:  0.02              Processes:               105
  Usage of /:   26.9% of 4.67GB   Users logged in:         0
  Memory usage: 19%               IPv4 address for enp0s2: 192.168.64.93
  Swap usage:   0%


1 update can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@docker:~$
```

On obtient alors un shell avec l'utilisateur *ubuntu* qui est notamment dans le groupe *sudo*.


Depuis ce shell, nous pouvons installer Docker avec la commande suivante:

```
ubuntu@docker:~$ curl -sSL https://get.docker.com | sh
```

Ajouter ensuite l'utilisateur *ubuntu* dans le groupe *docker* créé lors de l'installation. Cela permettra à cet utilisateur de communiquer avec le daemon Docker sans passer par l'utilisation de *sudo*.

```
ubuntu@docker:~$ sudo usermod -aG docker ubuntu
```

Note: il est nécessaire de sortir du shell et d'en lancer un nouveau pour que cela soit actif.

Vérifiez ensuite la version de Docker qui a été installé:

```
ubuntu@docker:~$ docker version
```

Vous devriez obtenir un résultat proche de celui ci-dessous

```
Client: Docker Engine - Community
 Version:           20.10.6
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        370c289
 Built:             Fri Apr  9 22:47:17 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.6
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       8728dd2
  Built:            Fri Apr  9 22:45:28 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.4
  GitCommit:        05f951a3781f4f2c1911b05e61c160e9c30eaa8e
 runc:
  Version:          1.0.0-rc93
  GitCommit:        12644e614e25b05da6fd08a38ffa0cfe1903fdec
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Multipass est un utilitaire très pratique pour manipuler des VMs en local, je vous conseille de le manipuler afin de voir les différentes fonctionnalités qu'il propose.

## Cleanup

La VM créée précedemment peut être supprimer facilement en utilisant la commande suivante:

```
$ multipass delete -p docker
```