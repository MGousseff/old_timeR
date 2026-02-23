---
title: "RIX : une aventure vers la reproductibilité avec R (1)"
date: 2025-09-03
author : MGousseff
---
#R/reproducibility/rix 

Ce post pose le contexte de mes tentatives avec `rix`, il ne présente pas d'intérêt technique immédiat, pour suivre un tout pas à pas d'installation de `rix` sur `ubuntu`, se référer au post suivant. 


# Contexte : pourquoi j'ai voulu tester rix, pourquoi j'ai abandonné rix, pourquoi j'ai retesté rix

## R et les interfaces intégrées de développement (ou comment se rendre compte qu'on fait partie d'une minorité)

### À l'ancienne ![[Rlogo.svg | 49]]

J'utilise R depuis la fin des années 90. Je n'ai pas appris R comme un langage de développement, mais comme un logiciel de traitement de données en ligne de commande. 
On apprenait alors à l'utiliser sur des terminaux monochrome reliés à un serveur famélique, en ligne de commande. Puis on a écrit des scripts dans des fichiers .R qu'on appelait avec  `source("script1.R")`. 
Je ne suis pas vieux au point d'avoir connu les cartes perforées, mais c'était quand même à l'ancienne. 

### Les IDES et l'ambivalence de RStudio ![[RStudio_Logo.png | 59]]

Les premiers environnement de développement intégrés sont arrivés assez rapidement, je pense que j'ai dû commencer avec Rkward, mais l'arrivée de RStudio a changé pas mal de choses. 

J'ai alors vécu un changement de vie radical qui m'a éloigné de la recherche et même du traitement de données pendant près de quinze ans. 

Quand je suis revenu, tout avait changé et utiliser Rstudio était devenu une évidence. Non seulement c'était le seul IDE vraiment dédié à R, mais en plus, un nouveau paradigme avait émergé. `ggplot2`, `tidyr`, `dplyr` la syntaxe changeait, pour le meilleur et pour le pire. 

De mon côté, quand je suis revenu vers l'analyse de données, les attentes métier avaient changé. On attendait avant d'un statisticien appliqué qu'il récupère les données, qu'il les traite et fournisse les graphiques et analyses qui permettraient d'écrire un article. On attendait maintenant d'un data-scientist qu'il produise une bibliothèque de code, un paquet R, que les utilisateurs puissent s'approprier, si possible avec une interface shiny. 

Et là, les limites ou les spécificités de RStudio et du tidyverse ont commencé à apparaître : 
- une installation spécifique
- l'évaluation non-standard au sein de boucles au sein de fonctions au sein de paquets et les problèmes que ça pose
- les fonctionnalités qu'on trouve dans d'autres IDE et pas dans Rstudio

Et c'est là qu'on réalise qu'on fait partie d'une communauté minoritaire et qu'on utilise un langage un peu bâtard. IntelliJ ou Eclipse ont été créées pour Java, VSCode pour C# (si je ne m'abuse), puis PyCharm pour Python...
Pour R, il y a Rstudio ou des plugins pour ces autres IDE. 

Je travaille maintenant avec une équipe qui utilise IntelliJ, j'ai donc testé le plugin R d'intelliJ. 

Il m'est devenu difficile de continuer avec Rstudio. Le refactoring est vraiment plus facile avec intelliJ, l'intégration avec git/GitHub me semble plus agréable. 

### Le jour où la terre s'arrêta (enfin où le plugin R d'intelliJ s'arrêta) ![[RpluginJB.svg | 49 ]]

J'ai essayé de passer de "script kiddie" à développeur en R, mais c'est une autre histoire. Toujours est-il que le  plugin R d'intelliJ est ce qui m'a permis de mieux organiser mes projets, et de refactorer le code plus facilement. 

Mais lors d'une montée de version de R, tout s'est arrêté (à la version  R-4.3, je crois). Le rwrapper avait changé d'une façon qui ne posait pas de problème à RStudio mais qui rendait le plugin R d'intelliJ inutilisable. 

Le retour sur Rstudio a été pénible. Difficile de distinguer ce qui tient à la difficulté à changer ses habitudes des limitations réelles de RStudio, mais  travailler était devenu très inconfortable.

## Rig, renv, et leurs limites ![[RIGlogo.png | 49]]

Afin de continuer à utiliser IntelliJ et son plugin R, j'ai donc cherché à désinstaller la dernière version de R et à explorer de quelles manières je pouvais installer des versions antérieures. 

Rig permet d'installer plusieurs versions de R. Pour chaque version, il permet d'avoir un répertoire spécifique pour les paquets R qu'on installe, cela aurait pu être la solution ultime. 

Mais une bonne partie des paquets R  appellent en fait des bibliothèques de code externes. Dans mon cas, le package `sf` repose sur des bibliothèques de manipulations de données spatiales, GDAL, GEOS, PROJ, et j'ai rencontré des problèmes de dépendances : bienvenue dans le **"dependency hell".** 

J'ai tout désinstallé, réinstallé RStudio et  lorsque j'ai cru que j'avais stabilisé mon installation pour une formation "calcul haute performance avec R", je suis tombé sur un problème de définition de dépendance, et j'ai raté une demi-journée de TD où je devais apprendre l'utilisation de `future` pour la parallélisation. 

## Rix, oui, mais non. 

Aux rencontres R 2024, j'ai suivi un atelier animé par Bruno Rodriguez, qui travaille au ministère de l'enseignement supérieur du Luxembourg, et qui a produit le package RIX. 

### Nix, oui, mais dur
B. Rodriguez  expliquait ce qu'était Nix : un gestionnaire de paquets sous forme de langage fonctionnel, qui permettait de construire des environnements en spécifiant très précisément les paquets qu'on voulait y utiliser, leurs versions, leurs dépendances. 

### rix, Nix pour R, mais facile ?
Il présentait ensuite son paquet, rix, et comment quelques commandes intuitives pour des R-istes permettaient de générer le fichier `default.nix `qui permettait de construire ensuite l'environnement nix (le nix shell) d'où on pourrait lancer R avec des paquets libérés de tout enfer des dépendances. 

Sauf que Bruno y travaillait depuis des mois, et que tout ce qui lui paraissait évident me demandait pas mal de temps. 

#### Nix, Determinates, Lyx etc.
Nix est un projet libre porté par une communauté enthousiaste. Mais une communauté de programmeurs, pas rebutés par les lignes de code, les installations exotiques et les réglages fin du système d'exploitation. 

Dès l'installation, on n'est pas sur un simple `sudo apt install nix` mais sur une commande `curl` pour aller télécharger des scripts sur les serveurs d'une entreprise, en l'occurrence, Determinates, avant de les lancer. Et les liens entre Determinates et la communauté Nix" libre" sont houleux, avec des débats upstream-downstream fork ou pas qui n'arrangent rien. 

### Récursivité
Pour utiliser rix, il faut avoir R. Mais Bruno recommande de n'utiliser que des versions de R installées avec rix. Donc trouver un script à charger pour lancer un environnement NIX temporaire dans lequel lancer R, dans lequel lancer `{rix}` dans lequel générer un nouveau fichier `default.nix` pour construire un nouveau nix shell !

### Chemins et canaux 
Je suppose que j'avais déjà une version de nix sur mon ordinateur, mais des conflits interminables ont failli me décourager dès l'installation de Nix. 

Après avoir fini par installer `{rix}`, réussi à m'en servir pour générer un fichier `default.nix` `nix-shell` continuait à me renvoyer des erreurs liés à des problèmes de chemins (PATH).

Après avoir démasqué un nix-daemon caché, tout désinstallé, tout réinstallé, comme ça ne fonctionnait toujours pas, j'ai appris qu'il fallait que je définisse des nix-channels et que celui qui était dédié à Ubuntu s'appelait :"unstable". 

C'est à ce stade que j'ai considéré que l'investissement était trop important et j'ai abandonné`{rix}`. 
J'ai cédé, j'ai réinstallé RStudio et Positron, j'ai réinstallé tous mes paquets à la version en cours, et j'ai travaillé en mode dégradé pendant des mois. 

Mais quand l'équipe d'intelliJ a réparé le plugin R, j'ai réalisé que ma productivité était bien meilleure sur intelliJ avec R qu'avec les IDE de posit. 

Il me fallait donc impérativement : 
- sécuriser une installation avec une version compatible avec intelliJ
- être capable de faire cohabiter des installations différentes au cas où un paquet dont je ne peux me passer deviendrait obsolète
- soigner mon orgueil blessé par la première tentative d'installation de`{rix}`. 

J'ai donc décidé de réessayer d'installer`{rix}`. Entre temps, Bruno Rodrigues a continué à travailler sur le paquet, et j'ai sans doute un peu progressé, je comprends peut-être un peu mieux le fonctionnement d'Ubuntu.  Et cette fois-ci, après pas mal de tâtonnements et de galères que j'espère éviter à d'autres R-istes sous Ubuntu, je crois y être arrivé.  Le prochain post décrira cette installation pas-à-pas en soulignant les choses que je ne comprends pas dans les vignettes du paquet `{rix}`. 









