---
layout: article
title: Emacs à tout faire
date: 2023-07-25
---

Quand je suis derrière un clavier, il est assez rare que je ne vienne
pas, à un instant donné, faire de la manipulation de fichiers
textes. Que ce soit pour coder, rédiger des articles, ou simplement
continuer à personnaliser ma machine, je ne peux pas vraiment me
passer d'un bon éditeur de texte. Par *bon éditeur de texte*,
j'entends un éditeur capable de répondre à chacun de ces besoins, sans
trop de difficultés. De même, je veux que l'éditeur puisse au maximum
se passer de l'utilisation de la souris (je n'aime pas devoir lever
mes mains du clavier pour autre chose que prendre une gorgée de café).

# Ma rencontre avec Emacs 🧡

Au moment où je cherchais un tel éditeur, deux choix m'ont paru
raisonnables : [vi(m)][vim] et [Emacs][]. Vu le titre de cet article,
vous comprenez aisément que je n'ai pas opté pour la première. Le
choix s'est fait à partir du choix des raccourcis claviers : je
préférais les commandes natives sur Emacs à celles de vim. 

[vim]: https://www.vim.org/
[Emacs]: https://www.gnu.org/software/emacs/

L'exploration des possibles pour Emacs commence avec le choix d'un
**thème de couleurs** : j'opte pour un thème foncé, [Dracula][], que
j'ai abandonné en 2021 pour un thème clair, [Leuven][]. Par la suite,
j'apprends qu'Emacs peut être utiliser dans de **nombreux contextes
différents**, autre que le code. Peu à peu, Emacs devient mon
**compagnon de route**, et je continue jour après jour à apprendre
comment le configurer et l'enrichir de nouvelles fonctionnalités.

[Dracula]: https://draculatheme.com/
[Leuven]: https://github.com/fniessen/emacs-leuven-theme

# Les contextes

Emacs est polyvalent. Une blague récurrente a longtemps été qu'il peut
tout faire, sauf le café ([quoique...][CoffeeMode]). Il est donc
illusoire d'en décrire ici chacune des fonctionnalités. Je vais en
fait décrire mon fichier de configuration principal (disponible [ici
!][init.el]) et expliquer quelques unes des entrées qui s'y trouve.

[CoffeeMode]: https://www.emacswiki.org/emacs/CoffeeMode
[init.el]: https://github.com/bdosse/.emacs.d

Avant de commencer, petite mise en garde : Emacs est hautement
personnalisable, et ma configuration est **une parmi une
multitude**. Je me suis inspiré (voire j'ai parfois fait un vilain
copier-coller) de pleins de sources disponibles sur le Web, parfois
dans la documentation même des *packages* que j'utilise.

## Prémisses

Je suis sous un OS GNU/Linux, et la configuration a été testée sous
Fedora 36, Debian 12, et ArchLinux. Savoir si elle fonctionnera sous
un OS Windows : osef-tier.

Pour gérer les paquets présents dans mon installation, j'utilise
[use-package][]. Vu que je n'ai pas beaucoup de paquets, la différence
(en termes de **performances**) n'est pas notable sur mon ordinateur
de bureau. Mon **netbook**, qui lui est bien moins puissant,
**bénéficie** de cette configuration (donc regarde un peu la doc' de
use-package, ça peut être utile 😉).

[use-package]: https://github.com/jwiegley/use-package

## Les mails

Cette partie n'est configurée que pour envoyer des mails depuis
Emacs. La récupération et la lecture des mails sont pilotés par
[offlineimap][] et [notmuch][], et je ferai un article pour en
discuter plus tard. Pour la partie qui nous intéresse, on trouve les
lignes suivantes :

[offlineimap]: https://github.com/OfflineIMAP/offlineimap3
[notmuch]: https://notmuchmail.org/

```elisp
(require 'smtpmail)
(autoload 'notmuch "notmuch" "notmuch mail" t)
```

Ces lignes chargent respectivement [smtpmail][] et notmuch.

[smtpmail]: https://www.emacswiki.org/emacs/SendingMail

```elisp
(setq mail-user-agent 'message-user-agent)
```

Indique le mode de composition des mails. Ici, il s'agit de
[MessageMode][] (une partie de [Gnus][]), qui peut gérer les pièces
jointes, contrairement à [MailMode][]. Il n'est pas nécessaire
d'utiliser Gnus pour utiliser *MessageMode*, donc si tu préfères
notmuch, [mutt][], ou même [cmail][] (wtf!?), libre à toi !

[MessageMode]: https://www.gnu.org/software/emacs/manual/html_node/message/index.html
[Gnus]: https://www.gnu.org/software/emacs/manual/html_node/emacs/Gnus.html
[MailMode]: https://www.emacswiki.org/emacs/MailMode
[mutt]: http://www.mutt.org/
[cmail]: http://cmail.osdn.jp/

```elisp
(setq user-mail-address "bdosse01@netc.eu"
      user-full-name "Benjamin Dosse")
```

Je configure mon identité ici, ainsi que l'adresse mail utilisé pour
les réponses. Cette configuration est suffisante si on ne compte pas
répondre à partir de plusieurs adresses mails différentes. Sinon, il
faudra bricoler un peu.

```elisp
(setq smtpmail-smtp-server "mail.mailo.com"
      smtpmail-smtp-user "bdosse01@netc.eu"
      send-mail-function 'smtpmail-send-it
      message-send-mail-function 'smtpmail-send-it
      smtpmail-stream-type 'ssl
      smtpmail-smtp-service 465)
```

Il s'agit de la configuration du service SMTP. Ces paramètres sont
disponibles chez le fournisseur (ici, [Mailo][]). Les fonctions
`send-mail-function` et `message-send-mail-function` sont
respectivement utilisées par *MailMode* et *MessageMode*. Je garde les
deux, just in case 😅

[Mailo]: https://www.mailo.com/

```elisp
(setq smtpmail-debug-info t)
(setq message-default-mail-headers "Cc: \nBcc: \n")
(setq message-kill-buffer-on-exit t)
```

La première ligne rend les logs plus verbeux, utile lorsque quelque
chose ne va pas dans la configuration. Avec la deuxième ligne, on
impose la présence des champs **Cc:** et **Bcc:**, absent par défaut.
La dernière ligne indique que lorsque le message est envoyé, on doit
immédiatement fermé le buffer qui a servi à la rédaction.

### Identification sur le serveur SMTP

Ce n'est pas une bonne idée de stocker le mot de passe du compte SMTP
en clair dans un fichier de configuration, en particulier s'il est
partagé comme celui que je mets à disposition. Pour éviter cet
écueuil, on peut stocker les identifiants dans un fichier chiffré, qui
sera lu et déchiffré par Emacs via GPG. 

```elisp
(require 'epa)
(epa-file-enable)
(setq epg-gpg-program "gpg")
(setq auth-sources '((:source "~/.authinfo.gpg")))
```

On utilise donc le module `epa`, et on lui indique qu'il doit chiffrer
et déchiffrer automatiquement les fichiers dont l'extension est
`.gpg`. Si on dispose de plusieurs versions de `gpg`, on indique avec
`epg-gpg-program` l'emplacement de la version que l'on souhaite
utiliser. Enfin, la dernière ligne indique l'emplacement du fichier
contenant des identifiants.

## YASnippet

Rédiger un document en LaTeX peut prendre un certain
temps... Heureusement, on peut raccourcir ce temps en utilisant
[YASnippet][] ! Ce petit ~~bijou/miracle~~ utilitaire permet de gérer
des *templates* : on tape une abréviation, on appuie sur la touche
d'appel, et... ✨ TADAAAM ✨ ! L'abréviation est étendue !

[YASnippet]: http://joaotavora.github.io/yasnippet/

Un de ses avantages, c'est qu'il est possible de définir des résultats
différents selon les *MajorMode*s en cours d'utilisation. C'est ce qui
fait que je peux avoir l'abréviation (appelé *clé* par YASnippet) `CC`
étendue en `\mathbb{C}` lorsque j'édite un fichier LaTeX, alors
qu'elle est étendue en

```python
class ClassName(Base):
    """ClassName --- Short Desc.
	
	Long Desc.
	"""
	
	def __init__(self, arg, *args, **kwargs):
	    pass
```

lorsque je travaille sur un fichier Python. Il est possible de
configurer finement le comportement de YASnippet vis à vis de
l'extension grâce à un système de *condition*. Ici, je ne vais décrire
que ce que j'ai fait pour m'éviter d'appuyer sur la touche `<TAB>` à
chaque commande LaTeX (et il y en a Tout. Le. Temps.)[^footnote1], et
la structure d'un snippet.

[^footnote1]: La capture d'écran qui illustre le jeu de couleur Leuven
    montre quelques commandes LaTeX que j'ai pris soin de
    raccourcir. Détrompe-toi, Naute, sans ses raccourcis (qui sont
    loins d'être optimaux, l'écriture serait encore plus longue *sans*
    YASnippet.

### Extension automatique

La fonction 

```elisp
(defun bdj/yas-try-auto-expand ()
    (when yas-minor-mode
      (let ((yas-buffer-local-condition ''(require-snippet-condition . auto)))
	(yas-expand))))
```

instruit à Emacs d'appeler la fonction `yas-expand` (celle qui est
chargée d'étendre les abréviations) à chaque fois qu'il rencontre le
**préfixe** d'une abréviation qui contient la *condition* `auto`. Il
faudra alors éviter de créer des snippets dont les clés ont un préfixe
commun de longueur non-nulle **et** possédant cette condition, au
risque de ne voir **que** le snippet avec le plus court préfixe être
étendu. 

Pour rendre cette fonction utile, *i.e.* faire en sorte qu'Emacs
appelle bien cette fonction lorsque le *MinorMode* `yas-minor-mode`
est actif, on ajoute cette ligne :

```elisp
(add-hook 'post-command-hook #'bdj/yas-try-auto-expand)
```

Avec use-package, on obtient :

```elisp
(use-package yasnippet
  :ensure t
  :init (yas-global-mode 1)
  :config
  (defun bdj/yas-try-auto-expand ()
    (when yas-minor-mode
      (let ((yas-buffer-local-condition ''(require-snippet-condition . auto)))
	(yas-expand))))
  (add-hook 'post-command-hook #'bdj/yas-try-auto-expand))
```

### Structure d'un snippet

Pour créer un nouveau snippet, il suffit de quelques informations de
base. On peut créer des templates assez long et complexe, c'est pour
cette raison que j'apprécie YASnippet !

Si on appelle la fonction `yas-new-snippet` (par défaut : `C-c &
C-n`), on nous présente ce buffer :

```
# -*- mode: snippet -*-
# name: 
# key: 
# --
```

L'entrée `name` est le nom du snippet (original n'est-ce pas ?), et
est utile en cas de recherche, mieux vaut donc lui donner un nom le
moins équivoque possible. L'entrée `key` est l'abréviation retenue par
YASnippet pour exécuter l'extension. On peut aussi utiliser des
variables, sous la forme `$n` (où `n` est un entier positif) ou
`${n:text}` où `text` est la valeur par défaut de la variable `$n`. La
variable `$0` indique l'emplacement final du curseur (après plusieurs
appuis sur `<TAB>`). Un exemple :

```
# -*- mode: snippet -*-
# name: Python property
# key: pty
# --
@property
def ${1:x}(self):
    $2
    return _${1:x}
	
@${1:x}.setter
def ${1:x}(self, ${3:value}):
    $0
```

qui permet de définir une propriété et un méthode getter simple en
Python. Il suffira ainsi d'écrire `pty` puis de presser `<TAB>` pour
développer ce template. Pour parcourir les champs, il suffira
d'appuyer sur `<TAB>` également.

Pour reprendre l'exemple du snippet LaTeX, j'ai écrit :

```
# -*- mode: snippet -*-
# name: Math Bold C
# key: CC
# condition : 'auto
# --
$\C$$0
```

Remarque ici que j'ai mis la condition `'auto` qui permet d'étendre le
snippet automatiquement !

## Correction orthographique

Il existe un moyen (assez rudimentaire) de faire de la correction
orthographique dans les fichiers textes : [Flyspell][]

