Introduction
============



# A propos des gestionaires de versions


> Un _gestionnaire de
version_ est un système qui enregistre l’évolution d’un fichier ou d’un ensemble de fichiers au cours
du temps de manière à ce qu’on puisse rappeler une version antérieure d’un fichier à tout moment.

Si vous êtes un dessinateur ou un développeur web, et que vous voulez conserver toutes les
versions d’une image ou d’une mise en page (ce que vous souhaiteriez assurément), un système de
gestion de version (VCS en anglais pour Version Control System) est un outil qu’il est très sage
d’utiliser. Il vous permet de ramener un fichier à un état précédent, de ramener le projet complet à
un état précédent, de visualiser les changements au cours du temps, de voir qui a modifié quelque
chose qui pourrait causer un problème, qui a introduit un problème et quand, et plus encore.
Utiliser un VCS signifie aussi généralement que si vous vous trompez ou que vous perdez des
fichiers, vous pouvez facilement revenir à un état stable.

# Les systèmes de gestion de version locaux


La méthode courante pour la gestion de version est généralement de recopier les fichiers dans un
autre répertoire (peut-être avec un nom incluant la date dans le meilleur des cas). Cette méthode
est la plus courante parce que c’est la plus simple, mais c’est aussi la moins fiable. Il est facile
d’oublier le répertoire dans lequel vous êtes et d’écrire accidentellement dans le mauvais fichier ou
d’écraser des fichiers que vous vouliez conserver.
Pour traiter ce problème, les programmeurs ont développé il y a longtemps des VCS locaux qui
utilisaient une base de données simple pour conserver les modifications d’un fichier.

Un des systèmes les plus populaires était [RCS](https://www.gnu.org/software/rcs/), qui est encore distribué avec de nombreux systèmes
d’exploitation aujourd’hui.

Cet outil fonctionne en conservant des ensembles de patchs

## Les patchs

> A _patch_ is a compact representation of the differences between two files, intended for use with line-oriented text files. It describes how to turn one file into another, and is asymmetric: the patch from file1 to file2 is not the same as the patch for the other direction.


The terms “patch” and “diff” are often used interchangeably, although there is a distinction, at least historically. A diff only need show the differences between two files, and can be quite minimal in doing so. A patch is an extension of a diff, augmented with further information such as context lines and filenames, which allow it to be applied more widely. These days, the Unix diff program can produce patches of various kinds.

```diff
diff --git a/foo.c b/foo.c
index 30cfd169..8de130c2 100644
--- a/foo.c
+++ b/foo.c
@@ -1,5 +1,5 @@
 #include <string.h>

 int check (char *string) {
-    return !strcmp(string, "ok");
+    return (string != NULL) && !strcmp(string, "ok");
 }
```

diff --git a/foo.c b/foo.c →  shows the unstaged changes between the working tree and the index. There are in fact no directories named a and b in the repository; they are just convention.

index 30cfd169..8de130c2 100644 →This is an extended header line, one of several possible forms, though there is only one in this patch. This line gives information from the Git index regarding this file: 30cfd169 and 8de130c2 are the blob IDs of the A and B versions of the file contents being compared, and 100644 are the “mode bits,” indicating that this is a regular file: not executable and not a symbolic link.


le reste → This is a difference section, or “hunk,” of which there is just one in this diff. The line beginning with `@@` indicates by line number and length the positions of this hunk in the A and B versions; here, the hunk starts at line 1 and extends for 5 lines in both versions. The subsequent lines beginning with a space are context: they appear as shown in both versions of the file. The lines beginning with minus and plus signs have the meanings just mentioned: this patch replaces a single line, fixing a common C bug whereby the program would crash if this function were passed a null pointer as its `string` argument.

A single patch file can contain the differences for any number of files, and `git diff` produces diffs for all altered files in the repository in a single patch (unlike the usual Unix `diff` command, which requires extra options to recursively process whole directory trees) d’une version à l’autre dans un format spécial sur disque ; il peut alors restituer l’état de n’importe quel fichier à n’importe quel instant en ajoutant toutes les
différences.

# Les systèmes de gestion de version centralisés

Le problème majeur que les gens rencontrent est qu’ils ont besoin de collaborer avec des
développeurs sur d’autres ordinateurs. Pour traiter ce problème, les systèmes de gestion de version
centralisés (CVCS en anglais pour Centralized Version Control Systems) furent développés. Ces
systèmes tels que CVS, Subversion, et Perforce, mettent en place un serveur central qui contient
tous les fichiers sous gestion de version, et des clients qui peuvent extraire les fichiers de ce dépôt
central. Pendant de nombreuses années, cela a été le standard pour la gestion de version.

Ce schéma (Figure 2. Gestion de version centralisée.) offre de nombreux avantages par rapport à la gestion de version locale. Par exemple,
chacun sait jusqu’à un certain point ce que tous les autres sont en train de faire sur le projet. Les
administrateurs ont un contrôle fin des permissions et il est beaucoup plus facile d’administrer un
CVCS que de gérer des bases de données locales.
Cependant ce système a aussi de nombreux défauts. Le plus visible est le point unique de panne
que le serveur centralisé représente. Si ce serveur est en panne pendant une heure, alors durant
cette heure, aucun client ne peut collaborer ou enregistrer les modifications issues de son travail. Si
le disque dur du serveur central se corrompt, et s’il n’y a pas eu de sauvegarde, vous perdez
absolument tout de l’historique d’un projet en dehors des sauvegardes locales que les gens auraient
pu réaliser sur leurs machines locales. Les systèmes de gestion de version locaux souffrent du
même problème — dès qu’on a tout l’historique d’un projet sauvegardé à un endroit unique, on
prend le risque de tout perdre.

# Les systèmes de gestion de version distribués


C’est à ce moment que les systèmes de gestion de version distribués entrent en jeu (DVCS en anglais
pour Distributed Version Control Systems). Dans un DVCS (tel que Git, Mercurial ou Darcs), les
clients n’extraient plus seulement la dernière version d’un fichier, mais ils dupliquent
complètement le dépôt. Ainsi, si le serveur disparaît et si les systèmes collaboraient via ce serveur,
n’importe quel dépôt d’un des clients peut être copié sur le serveur pour le restaurer. Chaque
extraction devient une sauvegarde complète de toutes les données.

De plus, un grand nombre de ces systèmes gère particulièrement bien le fait d’avoir plusieurs
dépôts avec lesquels travailler, vous permettant de collaborer avec différents groupes de personnes
de manières différentes simultanément dans le même projet. Cela permet la mise en place de
différentes chaînes de traitement qui ne sont pas réalisables avec les systèmes centralisés, tels que
les modèles hiérarchiques

# Comment Git apparait dans tout cela ?

Le noyau Linux est un projet libre de grande envergure. Pour la plus grande partie de sa vie
(1991–2002), les modifications étaient transmises sous forme de patchs et d’archives de fichiers. En
2002, le projet du noyau Linux commença à utiliser un DVCS propriétaire appelé BitKeeper.

La décision prise en 2002 d'utiliser BitKeeper pour le développement du noyau Linux a été très controversée. Certains, notamment Richard Stallman, fondateur de GNU, ont exprimé leur scepticisme envers l'utilisation d'un outil propriétaire pour un projet faisant figure de porte-drapeau du logiciel libre.

Tandis que le coordonnateur Linus Torvalds et quelques-uns des principaux développeurs adoptèrent BitKeeper, de nombreux développeurs-clés refusèrent d'en faire de même, en citant la licence de BitMover et en arguant du fait que le projet remettait une partie de son devenir à un développement propriétaire.

Pour couper court aux craintes exprimées, BitMover a ajouté des passerelles permettant une interopérabilité partielle entre les serveurs BitKeeper de Linux (administrés par BitMover) et les développeurs utilisant CVS ou Subversion. Mais même après cet ajout, des flamewars occasionnelles se produisaient sur la Linux Kernel Mailing List, impliquant régulièrement des développeurs-clés du noyau et Larry McVoy, le PDG de BitMover, qui est lui aussi un développeur du noyau Linux. mwouhahaha l'anbiance.

## Bitkeeper en gros

### Features

- **Simple:**	An easy to use command line interface.
- **Scalable:**	Nested Repositories are submodules done right! Version control	collections of repositories.
- **Flexible:**	Hybrid mode for binary files that uses a cloud of server for	binaries instead of bloating the source repositories.
- **Accurate:**	Tracking of file operations like creates, deletes, and renames.
- **Safe:**	All file accesses validate checksums for integrity. All file	writes include redundancy for error correction.
- **Dependable:**	Highly accurate auto-merge that uses the whole history to	resolve conflicts. Most other systems use variations of	`diff3`.
- **Discernable:**	Source annotations instantly available.
- **Fast:**	High performance and scales to very large repositories.
- **Free:**	Licensed under the [Apache Version 2 license](http://www.apache.org/licenses/LICENSE-2.0)


Si on regarde de plus près dans son utilisation: ça ressemble pas mal à git (cf les images)

## Drama

En 2005, les relations entre la communauté développant le noyau Linux et la société en charge du
développement de BitKeeper furent rompues, et le statut de gratuité de l’outil fut révoqué. Cela
poussa la communauté du développement de Linux (et plus particulièrement Linus Torvalds, le
créateur de Linux) à développer son propre outil en se basant sur les leçons apprises lors de
l’utilisation de BitKeeper. Certains des objectifs du nouveau système étaient les suivants :
• vitesse ;
• conception simple ;
• support pour les développements non linéaires (milliers de branches parallèles) ;
• complètement distribué ;
• capacité à gérer efficacement des projets d’envergure tels que le noyau Linux (vitesse et
compacité des données).