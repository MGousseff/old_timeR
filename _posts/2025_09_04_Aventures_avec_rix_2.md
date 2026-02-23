---
title: "RIX : une aventure vers la reproductibilité avec R (2)"
date: 2025-09-04
author: MGousseff
---

#R/reproducibility/rix 
# `{rix}` : créer des environnements spécifiques par projet R
Le paquet [`{rix}`](https://github.com/ropensci/rix) permet de créer des environnements de programmation reproductibles `Nix` pour les utilisateurs de R. 

Nix permet de créer des environnement semi-étanches en définissant les versions exactes des paquets désirés et les dépendances qu'ils exigent. Le tout est spécifié à l'aide d'un langage fonctionnel qui est réputé difficile à apprendre. L'utilisateur décrit les paquets et les dépendances dans un fichier `nix.default` enregistré dans le répertoire où il veut créer son environnement, puis construit l'environnement avec la commande `nix-build` et charge cet environnement pour y travailler avec ces paquets avec la commande `nix-shell`.

Rix permet de générer les fichiers `default.nix` à l'aide d'une syntaxe intuitive pour les utilisateurs de R. 

Nix ne produit pas des machines virtuelles complètes ni des containers étanches, mais des environnements qui interagissent fortement avec le système d'exploitation où on les crée. Ainsi, si un paquet n'existe pas dans un environnement Nix donné (on parle de nix shell) Nix va remonter dans le système d'exploitation et utiliser le paquet correspondant s'il s'y trouve. On verra que cela a des conséquences sur l'installation de R et `{rix}` . 

L'installation de `Nix`, de `{rix}` et leur configuration sous Ubuntu n'est pas évidente, et j'ai rencontré pas mal de difficultés en suivant les vignettes proposées [ici](https://docs.ropensci.org/rix/articles/a-getting-started.html) . 

Cet article décrit toutes les étapes que j'ai suivies et souligne quelques différences avec les vignettes du paquet. 

# Installation et configuration de Nix

## Installation de Nix
`Determinates` est une entreprise qui contribue au développement de `Nix` et propose des services et des fonctionnalités supplémentaires. Bruno Rodrigues suggère d'installer Nix avec l'installeur fourni par Determinates. 

```bash
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install

```
curl est une instruction qui télécharge le contenu de l'url fournie, et l'instruction après le pipe | lance l'installation du script `install`

## Cachix
Je n'ai pas approfondi ce qu'était `cachix`, mais c'est en gros un cache distant qui permet d'obtenir des paquets `Nix` déjà compilés, ce qui accélère fortement la construction des environnements `Nix`. 

On l'installe ainsi avec cette commande :
` nix-env -iA cachix -f https://cachix.org/api/v1/install `

Pour pouvoir l'utiliser, il a fallu que je m'ajoute comme `trusted user` avec la commande : 
```bash
echo "trusted-users = root <yourusername>" | sudo tee -a /etc/nix/nix.conf && sudo pkill nix-daemon
```
où il faut remplacer` <yourusername>` par votre identifiant d'utilisateur de session Ubuntu. 

Puis activer l'utilisation de cachix pour les paquets R : 
```bash
cachix use rstats-on-nix
```


# Installation d'une version de R via Nix et non via apt

Le créateur de `{rix}`, (Bruno Rodrigues, que j'appellerai BR dans cet article) recommande de ne pas installer R directement sur Ubuntu, afin d'éviter d'avoir des répertoires où se trouvent des paquets R et de risquer de ne pas savoir si on utilise un paquet défini dans un environnement Nix ou un paquet directement installé sur Ubuntu dont on n'aurait donc pas maîtrisé la version. 
Mais il faut une version de R pour utiliser `{rix}` et installer une version de R ! 
BR a donc mis en ligne sur le dépôt GitHub de `{rix}` un fichier `nix.default` et propose d'utiliser la commande suivante pour le télécharger et entrer dans un environnement Nix de départ, contenant une version de R, et à partir duquel on pourra invoquer `{rix}` pour créer notre fichier `default.nix` qui contiendra les versions de R et des paquets qu'on désire utiliser, avec les dépendances qui leur correspondent. 

Pour charger cet environnement de départ, il faut lancer la commande suivante depuis le répertoire dans votre répertoire projet : 

```bash
nix-shell -p R rPackages.rix
```

Le prompt dans votre terminal devrait alors être précédé de `[nix-shell:~/your_working_directory]$`

On se trouve alors dans un shell Nix, à l'intérieur d'un environnement tel qu'il a été défini par BR dans le script, et qui contient donc une version de R et une installation du paquet `{rix}`.

On peut alors, depuis cet environnement temporaire, lancer R et utiliser `{rix}` pour construire un fichier default.nix pour construire un premier environnement qui nous convienne. 

# Construction d'un premier environnement Nix

Depuis le shell Nix temporaire, lancer R et dans R lancer les commandes suivantes : 
```R
library(rix)

rix(
  r_ver = "4.4.2",
  r_pkgs = c("rix", "dplyr", "ggplot2"),
  # git_pkgs = list(
  #  package_name = "rix",
	# repo_url = "https://github.com/ropensci/rix",
    # commit = "76d1bdd03d78589d399b4b9d473ecde616920a82"
  #),
  system_pkgs = NULL,
  ide = "none",
  project_path = ".",
  overwrite = TRUE
)
```

Par rapport à la vignette de BR, j'ai commenté ce qui concerne les packages GitHub, voir section suivante. 
Cet appel à la fonction `rix()` va  créer un fichier `default.nix`  dans le repertoire défini par l'argument `project_path`, ici, "." définit le répertoire d'où on a lancé le nix shell. 
Ce fichier définira, dans la syntaxe propre à Nix (et qu'on veut éviter d'apprendre soi-même) qu'on veut un environnement avec la version 4.4.2 de R, les paquets dplyr et ggplot2 compatibles avec cette version de R. 

On quitte ensuite R,  `q()`, puis le shell nix temporaire, `exit`.

On peut alors lancer la commande `nix-build`, et une fois que l'environnement est construit, on y accède avec `nix-shell` et on peut y lancer R et vérifier avec` R.Version()` et `installed.packages()` que l'environnement Nix contient bien ce qu'on a défini. 

# Gestion de GitHub et des accès à GitHub
Dans la section précédente, les instructions relatives aux paquets disponibles sur un dépôt `git` ont été commenté. 
En effet, on ne peut bénéficier que d'un nombre d'accès limités à des dépôt GitHub sans être identifié, et on reçoit alors un avertissement lors de la construction de l'environnement `Nix`. 

Il convient alors de configurer un _Personnal Access Token, PAT,_ c'est à dire un jeton d'identification personnelle. 

Si vous n'avez pas associé de PAT à votre compte GitHub, [cette page](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) vous indique comment le faire. Il faut ensuite sauvegarder cet PAT dans un coffre fort de mot de passe. 

Je propose d'utiliser le paquet `credentials` pour faciliter l'accès de R à ce PAT et de saisir cette occasion pour s'entraîner à la mécanique d'ajout de paquets avec `{rix}`. En effet il faut impérativement éviter d'utiliser `install.packages` dans un environnement `Nix`,  puisque le paquet serait alors installé directement dans le système d'exploitation et non dans l'environnement Nix, et on retrouverait le risque d'enfer des dépendances. 










