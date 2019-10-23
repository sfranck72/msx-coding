---
title: "02-structure"
weight: 10
---

Je vais présenter uniquement ce qui est utile pour débuter le programme assembleur.

Le MSX c'est un processeur Z80 (le coeur).
Il discute avec 3 organes principaux :

- Le VDP : Un processeur qui gère l'affichage écran (16ko de mémoire).

- Le PSG : Un processeur qui gère le son.

- Le PPI  : Un processeur qui gère le clavier notamment.

Revenons au Z80, il dispose de 3 sortes de mémoires :

- La ROM.

- La RAM.

- Les registres internes.

![Stucture du MSX](/tuto asm 01/02/images/strucmsx.png?classes=shadow)

Les mémoires sont des cases. Pour pouvoir les différencier, on leur a donné des numéros que l'on appelle Adresse (comme les numéros des maisons dans une rue). 

- Ces adresses vont de 0 à 65535.

- Chaque adresse peut contenir un nombre entre 0 et 255.

**ROM** : Appelée mémoire morte. On peut simplement lire cette mémoire. On ne peut pas la modifier (données constructeur). Elle contient de petits programmes (routines) bien pratiques que l'on utilisera en assembleur pour lancer des actions ou faire des calculs.

**RAM** : Appelée mémoire vive. Elle est modifiable et c'est celle que nous utiliserons pour programmer. Par contre elle s'efface dès que l'on éteint le MSX.
C'est pour cela que l'on sauvegarde nos programmes.

**Registres internes** : Ce sont des sortes de mémoires. Ils n'ont pas d'adresse mais des lettres. En assembleur, c'est avec eux (une vingtaine environ) que l'on va jouer.
On va rappeler des adresses, lire ou écrire dedans et faire des calculs. Et tout ça, c'est avec l'aide de ces registres que l'on va le faire.

![Registres internes du Z80](/tuto asm 01/02/images/Regz80.jpg?classes=shadow)


Il existe une page très simple et explicative du fonctionnement de ces registres du Z80 : [Wiki Z80](http://www.google.com/url?q=http%3A%2F%2Ffr.wikibooks.org%2Fwiki%2FProgrammation_Assembleur_Z80&sa=D&sntz=1&usg=AFQjCNHwD_VHYzPYk2D7E879Q2yaX32nQA) 

Il est important que l'on parle du VDP pour finir car c'est avec lui que l'on va discuter fréquemment pour afficher nos sprites et les faire bouger.
Je passe volontairement sur les autres processeurs (PSG et PPI), on y reviendra au besoin.

**VDP**
Ce processeur permet d'afficher des images, des caractères ou des sprites à l'écran. 
Il contient 16ko de mémoire, appelée **VRAM** (dans laquelle on peut lire et écrire) et **9 registres internes** (R0 à R7+ 1 registre d'état) qui ne sont pas les mêmes que ceux du Z80.
La VRAM est divisée en 5 tables : 3 pour gérer l'affichage des caractères et 2 pour les sprites. Ces tables ne sont rien d'autre qu'un groupe d'adresses pouvant stocker des nombres.

Les 3 tables des caractères:

-**TGP** (Table Génératrice des Patrons): C'est là que l'on stocke les formes des caractères. Quand on tape la lettre A, le processeur Z80 demande au VDP de choisir la forme correspondant à la lettre A dans cette table.

-**TNP** (Table des Noms des Patrons): C'est là que l'on va indiquer où afficher le caractère à l'écran.

-**TC** (Table des Couleurs): C'est dans cette table que l'on va choisir la couleur du caractère et du fond de ce caractère.

Le principe de fonctionnement de ces 3 tables est le suivant : on demande au processeur VDP d'afficher un (ou des) caractère(s) en définissant ce que l'on veut afficher, où on veut l'afficher et dans quelle couleur. Et comme on connait maintenant les tables de caractères, on pourra modifier la forme de ces derniers à volonté (par exemple pour dessiner un décor de jeu).

Les 2 tables des sprites:

-**TGS** (Table Génératrice des Sprites): Stocke les dessins des sprites que l'on va définir.

-**TAS** (Table des Attributs des Sprites): stocke la position, le numéro et la couleur de chaque sprite.

![Registres internes du Z80](/tuto asm 01/02/images/vdp.png?classes=shadow)

Chacune de ces 5 tables débute par une adresse que l'on doit connaitre pour pouvoir y intégrer une liste d'informations, qui seront stockées à partir de là, vers les adresses suivantes en fonction du nombre d'informations (données) à stocker.

Pour définir automatiquement ces adresses de début en suivant les données constructeur, le VDP utilise 4 modes de fonctionnement :

- SCREEN 0 : Le mode texte

- SCREEN 1 : Le mode graphique 1

- SCREEN 2 : Le mode graphique 2

- SCREEN 3 : Le mode multicolore

Le programme assembleur que je veux expliquer utilise le SCREEN 1 donc je ne parlerai que de celui-là pour l'instant.

Le SCREEN 1 sur MSX1 c'est 32 colonnes sur 24 lignes de texte et la possibilité d'utiliser les sprites.

![Registres internes du Z80](/tuto asm 01/02/images/adrscreen1.png?classes=shadow)
