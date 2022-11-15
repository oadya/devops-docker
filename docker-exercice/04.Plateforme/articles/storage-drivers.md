Le storage driver est utilisé par le daemon Docker pour contrôler la façon dont les images et les containers sont sauvegardés et utilisés sur la machine hôte.

Nous allons voir dans le prochain chapitre qu'une image est constituée de plusieurs layers read-only. Lorsqu'un container est créé, c'est le travail du storage driver de créer un système de fichier à partir de l'ensemble de ces layers et d'ajouter une layer read-write par dessus. Cette dernière layer est l'endroit ou les modifications effectuées dans un container (fichiers ajoutés / modifiés / supprimés) seront écrites.

![Layers](./images/layers.jpg)

Lorsqu'une modification est effectuée dans le container, une opération de Copy-On-Write est effectuée. Son implémentation dépend du storage driver qui est utilisé. Les storage drivers disponibles sont:
* aufs
* overlay
* overlay2
* btrfs
* zfs
* devicemapper

Nous allons voir que le choix d'un driver dépend de différents éléments.


## La distribution Linux utilisée (Ubuntu, Debian, CentOS, Fedora, ...)

Distribution Linux | Storage drivers recommandé
------------------ | --------------------------
Ubuntu | aufs, devicemapper, overlay2 , overlay, zfs, vfs
Debian | aufs, devicemapper, overlay2, overlay, vfs
CentOS | devicemapper, vfs
Fedora | devicemapper, overlay2 (Fedora 26+, experimental), overlay (experimental), vfs

## Le système de fichiers sur lequel /var/lib/docker est installé

Storage driver | Systèmes de fichiers supportés
-------------- | ------------------------------
overlay, overlay2 | ext4, xfs
aufs              | ext4, xfs
devicemapper      | direct-lvm 
btrfs             | btrfs
zfs               | zfs

## Le workload de l'application

Ceci inclu notamment:
* la taille des fichiers modifiés
* la fréquence de modification
* ...

Type de workload | Storage driver recommandé
---------------- | -------------------------
écriture intensive     | block level => devicemapper, btrfs, zfs
écriture non intensive | file level => aufs, overlay, overlay2
haute densité          | zfs

Le choix optimal du storage driver n'est pas une chose simple car il dépend de différents paramètres. Un kernel récent avec overlay2 est un bon choix par défaut. Il est cependant souvent recommandé d'utiliser un volume pour la gestion des données de façon à assurer la persistance en dehors du cycle de vie d'un container.

## Connaitre le driver utilisé

La commande docker info permet de connaitre le storage driver utilisé ainsi que le système de fichiers sous-jacents.

```
$ docker info | grep Storage -A1
Storage Driver: overlay2
 Backing Filesystem: extfs
```

Pour changer le storage driver, il suffit de le spécifier en ligne de commande avec l'option --storage-driver (approche non recommandé) ou bien d'ajouter la clé storage-driver dans le fichier de configuration */etc/docker/daemon.json* comme le montre l'exemple suivant. 

```
{
  ...
  "storage-driver": "devicemapper"
}
```

> L'utilisation de certains storage drivers nécessite des configurations supplémentaires au niveau du kernel.

Il suffit ensuite redémarrer le daemon Docker (attention la modification du storage driver n'est pas possible sur Docker Desktop macOS ou Windows).
