---
title: "07-un shoot-em-up"
weight: 7
---

## Organigramme

Maintenant que l'on a vu comment utiliser les tables relatives aux caractères, on va mettre en pratique la 2eme partie des tables du VDP : **L'affichage des sprites**.

Avant de commencer, je vais vous parler du programme de shoot-em-up basique qui fait l'objet de ce blog.  
Car on va commencer à écrire ce programme. En effet, il a l'avantage d'être très simple et d'aborder les grands principes d'un jeu, c'est avec lui que l'on abordera la mise en pratique de l'affichage des sprites dans un premier temps.

Mais ce programme nous permettra d'aller plus loin et d'aborder des sujets complémentaires comme :  

- le déplacement du sprite-joueur,  
- le déplacement du sprite-ennemi,  
- la gestion du tir pour le joueur, 
- la gestion de la collision,  
- la gestion d'un temporisation basique,  
- la construction d'une boucle principale qui gère tous cela.

Avant de rentrer dans le code, vous devez avoir une vue d'ensemble (organigramme) du projet et comment il est organisé, sinon vous ne comprendrez pas le découpage du programme.

Un programme en assembleur est la plupart du temps décomposé en une succession de petits programmes que l'on peut appeler blocs, routines, etc...
Ils exécutent un calcul ou une action (voire les deux) d'un domaine bien précis.
Par exemple, on peut faire un bloc qui s'occupe de la mise en place des sprites, un autre pour la gestion des touches directionnelles du clavier, un autre pour la gestion de la collision des sprites, etc...

L'avantage, c'est que l'on visualise mieux des blocs qui gèrent un seul sujet, on y comprend mieux le code, on saura mieux repérer les éventuelles erreurs, on pourra améliorer le programme en modifiant bloc par bloc sans tout casser.  

Ainsi, on peut tout découper et faire en sorte que notre programme principal ne soit qu'une succession d'appel de ces blocs dans un ordre qui nous convienne.
Ce programme principal sera une boucle assez simple et lisible mais qui pourra faire beaucoup de chose par de simples appels à ces sous-programmes.

Voici un découpage possible par blocs :

![Organigramme](/tuto asm 01/07/images/1.png?classes=shadow)

On va faire démarrer la cartouche .ROM pour qu'elle appelle le programme principal, qui se comporte comme la boucle infinie que nous avons codé pour l'affichage d'un caractère.....bon, je vous le remontre :  
<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">BOUCLE:</span>
<span style="color:#a6e22e">JP</span>     <span style="color:#f8f8f2">BOUCLE</span>
</pre>

On voit d'ailleurs qu'un bloc (ou sous-programme) débute par un nom suivit de 2 points `:`

Ca s'appelle un **label**, je vous ai déjà dit qu'en assembleur on n'a pas les numéros de ligne pour appeler un code, on utilise des déclarations de labels que l'on pourra appeler via un *`CALL` sans les 2 points ':'
Dans notre exemple ci-dessus, si je voulais appeler mon programme principal, je taperai `CALL BOUCLE`.

`JP` veut dire **Jump** (saute), donc le bloc se comporte de cette façon:

- `BOUCLE:`     Je m'appelle BOUCLE  
- `JP BOUCLE`  Saute au label BOUCLE

Si on ne faisait pas cette boucle infinie, tous le jeux serait installé puis il s'arrêterait, nous ce que l'on veut c'est que le jeux déplace constamment l'ennemi, le joueur, surveille le clavier directionnel en permanence.

Par contre, la plupart des autres sous-programmes ne vont pas se finir par un `JP` de son propre label.  
Ils sont appelés par le programme principal pour réaliser un calcul ou une action  bien précise et rendent la main.  
Par exemple, si le programme principal surveille le clavier directionnel et qu'en fonction de la flèche pressée, il fait appel à un sous-programme qui déplace le joueur d'un petit espace et rend la main au programme principal, car ce dernier à besoin de controler si une autre touche est pressée.  
Donc ce sous-programme se terminera par un `RET` pour **return** (retour) et non pas par un `JP`.

Vous allez mieux comprendre maintenant l'organigramme de ces blocs que j'ai volontairement vulgarisé (on verra dans la pratique qu'il est plus subtil) et comment ils peuvent intéragir :  
![Organigramme2](/tuto asm 01/07/images/2.png?classes=shadow)

Je vous promets, dès le prochain article on attaque le code directement ;-)

