Pour aller plus loin
====================


# Différence entre switch / checkout

Trop simple, regardons dans le code (un peu): (mdr cette partie ne sert à rien en tant que note)


- On essaye de trouver le main de la CLI
- Ok pas mal, ensuite essayons de trouver les commandes, on cherche donc une liste de structure de données associés a des actions et des options
- Pas mal,
- Ok on est content, on cherche donc celles qui nous interessent  ...

Les bouts de code 


```c
int cmd_switch(int argc,
	       const char **argv,
	       const char *prefix,
	       struct repository *repo UNUSED)
{
	struct checkout_opts opts = CHECKOUT_OPTS_INIT;
	struct option *options = NULL;
	struct option switch_options[] = {
		OPT_STRING('c', "create", &opts.new_branch, N_("branch"),
			   N_("create and switch to a new branch")),
		OPT_STRING('C', "force-create", &opts.new_branch_force, N_("branch"),
			   N_("create/reset and switch to a branch")),
		OPT_BOOL(0, "guess", &opts.dwim_new_local_branch,
			 N_("second guess 'git switch <no-such-branch>'")),
		OPT_BOOL(0, "discard-changes", &opts.discard_changes,
			 N_("throw away local modifications")),
		OPT_END()
	};

	opts.dwim_new_local_branch = 1;
	opts.accept_ref = 1;
	opts.accept_pathspec = 0;
	opts.switch_branch_doing_nothing_is_ok = 0; -> Il faudra mettre en lumière ces deux commandes
	opts.only_merge_on_switching_branches = 1;
	opts.implicit_detach = 0;
	opts.can_switch_when_in_progress = 0;
	opts.orphan_from_empty_tree = 1;
	opts.overlay_mode = -1;

	options = parse_options_dup(switch_options);
	options = add_common_options(&opts, options);
	options = add_common_switch_branch_options(&opts, options);

	cb_option = 'c';

	return checkout_main(argc, argv, prefix, &opts, options,
			     switch_branch_usage);
}
```


```c
int cmd_checkout(int argc,
		 const char **argv,
		 const char *prefix,
		 struct repository *repo UNUSED)
{
	struct checkout_opts opts = CHECKOUT_OPTS_INIT;
	struct option *options;
	struct option checkout_options[] = {
		OPT_STRING('b', NULL, &opts.new_branch, N_("branch"),
			   N_("create and checkout a new branch")),
		OPT_STRING('B', NULL, &opts.new_branch_force, N_("branch"),
			   N_("create/reset and checkout a branch")),
		OPT_BOOL('l', NULL, &opts.new_branch_log, N_("create reflog for new branch")),
		OPT_BOOL(0, "guess", &opts.dwim_new_local_branch,
			 N_("second guess 'git checkout <no-such-branch>' (default)")),
		OPT_BOOL(0, "overlay", &opts.overlay_mode, N_("use overlay mode (default)")),
		OPT_END()
	};

	opts.dwim_new_local_branch = 1;
	opts.switch_branch_doing_nothing_is_ok = 1;
	opts.only_merge_on_switching_branches = 0;
	opts.accept_ref = 1;
	opts.accept_pathspec = 1;
	opts.implicit_detach = 1;
	opts.can_switch_when_in_progress = 1;
	opts.orphan_from_empty_tree = 0;
	opts.empty_pathspec_ok = 1;
	opts.overlay_mode = -1;
	opts.checkout_index = -2;    /* default on */
	opts.checkout_worktree = -2; /* default on */

	if (argc == 3 && !strcmp(argv[1], "-b")) { -> J'ai trois arguments et le second est -b
		/*
		 * User ran 'git checkout -b <branch>' and expects
		 * the same behavior as 'git switch -c <branch>'.
		 */
		opts.switch_branch_doing_nothing_is_ok = 0;
		opts.only_merge_on_switching_branches = 1;
	}

	options = parse_options_dup(checkout_options);
	options = add_common_options(&opts, options);
	options = add_common_switch_branch_options(&opts, options);
	options = add_checkout_path_options(&opts, options);

	return checkout_main(argc, argv, prefix, &opts, options,
			     checkout_usage); -> On Appelle la même fonction en bas
}
```

# Ouverture

Parler de git reset & restore comme ouverture pour cette partie

# Bisect

Un historique, c’est une liste triée. Le critère de tri ? Le temps ! 
Les commits partent du plus ancien vers le plus récent, même s’ils 
peuvent bifurquer puis se rejoindre au fil du graphe des branches et des
 fusions.

Lorsqu’on cherche quelque chose au sein d’une liste triée, il serait 
dommage de simplement commencer au début pour avancer vers la fin… Vous 
avez probablement déjà joué au jeu du « plus petit, plus grand » : vous 
devez trouver un nombre entre 1 et 100, par exemple. Dans un tel cas, je
 m’inquièterais pour une personne qui commencerait à 1, ou vers 100, 
pour ensuite piocher au hasard. Instinctivement, la plupart des gens 
commencent au milieu, à 50, et si on leur répond « plus petit », 
piochent ensuite au milieu du sous-ensemble ainsi défini, donc à 25, 
etc.

Ce type d’algorithme porte un nom : c’est une [dichotomie](http://fr.wikipedia.org/wiki/Dichotomie) (parfois appelée *recherche dichotomique*). Il permet de trouver ce qu’on cherche au maximum en *[log2(n)]* coups, soit pour un ensemble *[1, 100]*, au maximum en 7 coups. C’est encore bien plus impressionnant lorsqu’on étend l’intervalle de façon significative : pour *[1, 1 000 000 000]*, il ne faudrait au pire que 27 coups ! Un sérieux gain de temps…

Il est possible d’appliquer ce principe à la recherche du premier 
commit qui, dans un historique de commits (c’est-à-dire une liste 
temporellement ordonnée de commits), a introduit un bug.

En anglais, cet algorithme se nomme *binary search*, mais c’est son exploitation mathématique, nommée *bisection*, qui a donné son nom à la commande `git bisect`.


La commande `git bisect` exploite toute une série de sous-commandes.

1. On démarre avec un `git bisect start`. Il est possible de préciser d’entrée de jeu un commit foireux (généralement le `HEAD` et un autre qui est bon), sinon on les indiquera ensuite :
2. Un `git bisect bad` identifie le premier commit problématique connu (si on n’ajoute rien, c’est le `HEAD`, ce qui est normalement le cas)
3. Un `git bisect good` identifie un commit qui n’avait pas
le problème (le plus proche possible de nous, mais au pire on ira le
chercher loin pour ne pas galérer)
4. À partir de là, la dichotomie commence : Git fait un `checkout` au milieu (ou à peu près) de l’intervalle, nous dit où on en est, et demande le verdict : suivant le cas, on répondra par un `git bisect bad` ou `git bisect good` (plus rarement, `git bisect skip`).
5. Au bout d’un moment, si on n’a pas répondu n’importe quoi ni laissé
trop de cas indéterminés, Git nous indiquera le premier commit fautif.
6. On pourra alors abandonner le *bisecting* avec un `git bisect reset`.


# Git log

Après avoir créé plusieurs commits ou si vous avez cloné un dépôt ayant un historique de commits,
vous souhaitez probablement revoir le fil des évènements. Pour ce faire, la commande git log est
l’outil le plus basique et le plus puissant.


```bash
$ git log
	commit ca82a6dff817ec66f44342007202690a93763949
	Author: Scott Chacon schacon@gee-mail.com
	Date: Mon Mar 17 21:52:11 2008 -0700
			changed the version number
	commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	Author: Scott Chacon schacon@gee-mail.com
	Date: Sat Mar 15 16:40:33 2008 -0700
			removed unnecessary test
	commit a11bef06a3f659402fe7563abf99ad00de2209e6
	Author: Scott Chacon schacon@gee-mail.com
	Date: Sat Mar 15 10:31:28 2008 -0700
			first commit
```

Par défaut, git log invoqué sans argument énumère en ordre chronologique inversé les commits
réalisés. Cela signifie que les commits les plus récents apparaissent en premier. Comme vous le
remarquez, cette commande indique chaque commit avec sa somme de contrôle SHA-1, le nom et
l’e-mail de l’auteur, la date et le message du commit.
git log dispose d’un très grand nombre d’options permettant de paramétrer exactement ce que l’on
cherche à voir. Nous allons détailler quelques-unes des plus utilisées

Une des options les plus utiles est **-p**, qui montre les différences introduites entre chaque validation.
Vous pouvez aussi utiliser **-2** qui limite la sortie de la commande aux deux entrées les plus
récentes :

```bash
$ git log -p -2
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date: Mon Mar 17 21:52:11 2008 -0700
changed the version number
diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
spec = Gem::Specification.new do |s|
s.platform = Gem::Platform::RUBY
s.name = "simplegit"
- s.version = "0.1.0"
+ s.version = "0.1.1"
s.author = "Scott Chacon"
s.email = "schacon@gee-mail.com"
s.summary = "A simple gem for using Git in Ruby code."
commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date: Sat Mar 15 16:40:33 2008 -0700
removed unnecessary test
diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index a0a60ae..47c6340 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -18,8 +18,3 @@ class SimpleGit
end
end
-
-if $0 == __FILE__
- git = SimpleGit.new
- puts git.show
-end
\ No newline at end of file
```

Cette option affiche la même information mais avec un diff suivant directement chaque entrée.
C’est très utile pour des revues de code ou pour naviguer rapidement à travers l’historique des
modifications qu’un collaborateur a apportées. Vous pouvez aussi utiliser une liste d’options de
résumé avec git log. Par exemple, si vous souhaitez visualiser des statistiques résumées pour
chaque commit, vous pouvez utiliser l’option --stat

```bash
$ git log --stat
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date: Mon Mar 17 21:52:11 2008 -0700
changed the version number
Rakefile | 2 +-
1 file changed, 1 insertion(+), 1 deletion(-)
commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date: Sat Mar 15 16:40:33 2008 -0700
removed unnecessary test
lib/simplegit.rb | 5 -----
1 file changed, 5 deletions(-)
commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date: Sat Mar 15 10:31:28 2008 -0700
first commit
README | 6 ++++++
Rakefile | 23 +++++++++++++++++++++++
lib/simplegit.rb | 25 +++++++++++++++++++++++++
3 files changed, 54 insertions(+
```

Comme vous pouvez le voir, l’option --stat affiche sous chaque entrée de validation une liste des
fichiers modifiés, combien de fichiers ont été changés et combien de lignes ont été ajoutées ou
retirées dans ces fichiers. Elle ajoute un résumé des informations en fin de sortie.
Une autre option utile est --pretty. Cette option modifie le journal vers un format différent.
Quelques options incluses sont disponibles. L’option oneline affiche chaque commit sur une seule
ligne, ce qui peut s’avérer utile lors de la revue d’un long journal. En complément, les options short
(court), full (complet) et fuller (plus complet) montrent le résultat à peu de choses près dans le
même format mais avec plus ou moins d’informations.

```bash
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 changed the version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 first commi
```

L’option la plus intéressante est format qui permet de décrire précisément le format de sortie. C’est
spécialement utile pour générer des sorties dans un format facile à analyser par une machine —
lorsqu’on spécifie intégralement et explicitement le format, on s’assure qu’il ne changera pas au gré

```bash
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit
```

Table 1. Options utiles pour git log --pretty=format
Option Description du formatage
%H Somme de contrôle du commit
%h Somme de contrôle abrégée du commit
%T Somme de contrôle de l’arborescence
%t Somme de contrôle abrégée de l’arborescence
%P Sommes de contrôle des parents
%p Sommes de contrôle abrégées des parents
%an Nom de l’auteur
%ae E-mail de l’auteur
%ad Date de l’auteur (au format de l’option -date=)
%ar Date relative de l’auteur
%cn Nom du validateur
%ce E-mail du validateur
%cd Date du validateur
%cr Date relative du validateur
%s Sujet

Les options oneline et format sont encore plus utiles avec une autre option log appelée --graph.
Cette option ajoute un joli graphe en caractères ASCII pour décrire l’historique des branches et
fusions :

```bash
$ git log --pretty=format:"%h %s" --graph
* 2d3acf9 ignore errors from SIGCHLD on trap
* 5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
|\
| * 420eac9 Added a method for getting the current branch.
* | 30e367c timeout code and tests
* | 5a09431 add timeout protection to grit
* | e1193f8 support for heads with slashes in them
|/
* d6016bc require time for xmlschema
* 11d191e Merge branch 'defunkt' into local
```

En complément des options de formatage de sortie, git log est pourvu de certaines options de
limitation utiles — des options qui permettent de restreindre la liste à un sous-ensemble de
commits. Vous avez déjà vu une de ces options — l’option -2 qui ne montre que les deux derniers
commits. En fait, on peut utiliser -<n>, où n correspond au nombre de commits que l’on cherche à
visualiser en partant des plus récents. En vérité, il est peu probable que vous utilisiez cette option,
parce que Git injecte par défaut sa sortie dans un outil de pagination qui permet de la visualiser
45
page à page.
Cependant, les options de limitation portant sur le temps, telles que --since (depuis) et --until
(jusqu’à) sont très utiles. Par exemple, la commande suivante affiche la liste des commits des deux
dernières semaines :

```bash

$ git log --since=2.weeks
```

Cette commande fonctionne avec de nombreux formats — vous pouvez indiquer une date
spécifique (2008-01-05) ou une date relative au présent telle que "2 years 1 day 3 minutes ago".
Vous pouvez aussi restreindre la liste aux commits vérifiant certains critères de recherche. L’option
--author permet de filtrer sur un auteur spécifique, et l’option --grep permet de chercher des mots
clés dans les messages de validation.

Note : Vous pouvez spécifier à la fois des instances --author et --grep, ce qui limitera la
sortie aux commits correspondant à au moins un des critères ; cependant l’ajout de
l’option --all-match limite la sortie aux seuls commits qui correspondent à la fois à
tous les critères des motifs --grep.

Un autre filtre vraiment utile est l’option -S qui prend une chaîne de caractères et ne retourne que
les commits qui introduisent des modifications qui ajoutent ou retirent du texte comportant cette
chaîne. Par exemple, si vous voulez trouver la dernière validation qui a ajouté ou retiré une
référence à une fonction spécifique, vous pouvez lancer :

```bash
$ git log -S nom_de_fonction
```

La dernière option vraiment utile à git log est la spécification d’un chemin. Si un répertoire ou un
nom de fichier est spécifié, le journal est limité aux commits qui ont introduit des modifications aux
fichiers concernés. C’est toujours la dernière option de la commande, souvent précédée de deux
tirets (--) pour séparer les chemins des options précédentes.

```bash
$ git log -- chemin/vers/le/fichier
```

Le tableau Options pour limiter la sortie de git log récapitule les options que nous venons de voir
ainsi que quelques autres pour référence.

Table 3. Options pour limiter la sortie de git log
Option Description
-(n) N’affiche que les n derniers commits
--since, --after Limite l’affichage aux commits réalisés après la date spécifiée
--until, --before Limite l’affichage aux commits réalisés avant la date spécifiée
--author Ne montre que les commits dont le champ auteur correspond à la
chaîne passée en argument
46
Option Description
--committer Ne montre que les commits dont le champ validateur correspond
à la chaîne passée en argument
--grep Ne montre que les commits dont le message de validation
contient la chaîne de caractères
-S Ne montre que les commits dont les ajouts ou retraits contient la
chaîne de caractères

Par exemple, si vous souhaitez visualiser quels commits modifiant les fichiers de test dans
l’historique du code source de Git ont été validés par Junio C Hamano et n’étaient pas des fusions
durant le mois d’octobre 2008, vous pouvez lancer ce qui suit :

```bash

$ git log --pretty="%h - %s" --author='Junio C Hamano' --since="2008-10-01" \
--before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an unborn
branch
```