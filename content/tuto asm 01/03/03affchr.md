---
title: "03-Afficher un caractère"
weight: 15
---

# (La théorie)

Un caractère affiché par le VDP c'est 3 tables auxquelles on devra donner des instructions:

- **TNP (position à l'écran)**

- **TGP (la forme du caractère)**

- **TC (la couleur du caractère et du fond de celui-ci)**

---

#### TNP

>En mode SCREEN 1, on a vu que la TNP débute à l'adresse 6144.

On sait aussi qu'en SCREEN 1 on a **32 colonnes pour 24 lignes** (pour le texte).
Cela donne **768 positions possibles à l'écran (32 x 24 = 768)**.

Ces positions vont de gauche à droite (comme le fonctionnement d'une machine à écrire), de la position 0 à 31 pour la première ligne, puis passe à la ligne 2 et poursuit la numérotation jusqu'à la dernière position N° 768 en bas à droite.

Voici l'écran telle qu'il est en SCREEN 1 : 
![Stucture du MSX](/tuto asm 01/03/images/TNP.png?classes=shadow)
<center> *TNP Screen 1* </center>


Toutes les positions sont des adresses mémoires qui se suivent.
Sachant que la TNP  débute à l'adresse 6144, on sait maintenant que cette adresse correspond à la position 0.
Si on veut afficher le caractère à la position 5, on va simplement faire 6144+5 et on aura l'adresse où afficher le caractère: 6149

---

#### TGP
On a vu que c'était la table qui abrite les formes des caractères.
**Cette table contient 256 caractères numérotés** sous la norme standard internationale ASCII.

![Table des caractères](/tuto asm 01/03/images/ascii.png?classes=shadow)  <center> *Table des caractères* </center>


Un exemple avec le tableau ci-dessus :  
Vous voulez le code ASCII de **la lettre 'A'**, vous prenez le N° de ligne et de colonne (respectivement), ici 4 et 1, vous l'assemblez et **cela donne du hexadécimal : 41**

En assemblage on utilise beaucoup l'hexadécimal pour des raisons que je n'expliquerai pas ici.  
Mais il est facile de traduire de l'hexadécimal vers le décimal (et inversement), avec une calculatrice.

Celle de Windows le fait très bien, il suffit de choisir dans le menu : *Affichage/Programmeur*.

Vous cochez :
![Stucture du MSX](/tuto asm 01/03/images/calc.jpg?classes=shadow)

Vous rentrez le nombre **41**, il ne vous reste plus qu'à cocher

- [x] Déc

et vous obtenez du décimal, dans notre cas 65, qui est le n° ASCII de la lettre **'A'** que reconnait la TGP, ouf !!!

Juste pour information, l'adresse de la lettre **'A'** dans la TGP n'est pas **65**.
Chaque caractère est défini dans une grille de 8 x 8 (comme un sprite simple).

Donc chaque caractère prend 8 adresses mémoires successives dans la TGP.

La TGP débute en  0 (SCREEN 1), notre **'A'** est le 65ème, donc son adresse de début de forme est :
**Adresse de début de la TGP + 65\*8 = 0 + 520 = 520**

Si on voulait modifier la forme du **'A'** il faudrait que l'on modifie les valeurs des adresses 520 à 527 (8 octets), mais on va pas le faire dans notre exemple. 

![Charactère A](/tuto asm 01/03/images/char.jpg?classes=shadow)

Avec l'adresse et le code du caractère, on est déjà capable de l'afficher, la couleur est optionnelle.
On va quand même voir comment la TC fonctionne.

---

#### TC
Pour modifier cette table, il nous faut 2 informations :

- [x] L'adresse du caractère dans cette table.
- [x] Le calcul de sa couleur.

Dans notre exemple, le **'A' c'est le numéro 65**, on le divise par 8 et on ajoute ce résultat au début de l'adresse de la table (8192 en SCREEN1).
**(65:8) + 8192 = 8200** (on ne retient que la part entière).

> **8200 c'est l'adresse du 'A' dans la TC.**


Pour la couleur (16 couleurs disponibles en SCREEN 1), le calcul est :  
**Couleur du caractère x 16 + couleur de fond du caractère**.
Pour un caractère Magenta (couleur 13) et Fond Blanc (couleur 15) cela donne:  
    **13 x 16 + 15 = 223**

Donc si on veut modifier une couleur, on passe par cette table et on modifie la valeur de l'adresse 8200 en lui mettant 223.
![Charactère A](/tuto asm 01/03/images/affchr.jpg?classes=shadow)
<center>*Affiche le caractère 'A' à l'écran*</center>

Voilà, on va mettre tout cela en application en utilisant enfin un assembleur pour MSX.  
L'article suivant va uniquement parler de la mise en place des outils pour programmer en assembleur.  
C'est le préalable à la mise en pratique de ce que nous avons vu jusqu'ici.


