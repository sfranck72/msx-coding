---
title: "10-Déplacement de l'envahisseur"
weight: 10
---



On le sait tous, un extra-terrestre c'est bête à manger du foin, donc on va lui créer une intelligence artificielle (IA) limitée...

en fait, on va faire déplacer l'ennemi de façon autonome mais simple et de plus, l'exemple du livre ne va pas plus loin, donc....on va dire que c'est moi qui suis limité ;-)

On va utiliser une des variables que l'on a déclaré précédemment et qui s'appelle `inddir` (comme 'indicateur de direction'), je vous rappelle ce que l'on avait fait pour ces variables :

<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">indcol:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41000</span>       <span style="color:#75715e">; défini l'adresse de la variable indcol (registre des collisions)</span>
<span style="color:#f8f8f2">indbal:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41001</span>       <span style="color:#75715e">; défini l'adresse de la variable indbal</span>
<span style="color:#f8f8f2">inddir:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41002</span>       <span style="color:#75715e">; défini l'adresse de la variable inddir</span>
</pre>


On les a d'abord déclaré comme associées à une adresse spécifique de la mémoire RAM dans le pied de page de notre code , pour `inddir` c'était une adresse très lointaine dans la mémoire qui nous permet d'espérer qu'elle ne se fera pas écraser par du code, ici l'adresse 41002.

De ce fait on pourra stocker n'importe quelle valeur dedans et la modifier au besoin.

Cet indicateur, on va le manipuler pour le faire passer d'une valeur 0 à 1 (et inversement).

Dans l'un des articles précédent on a déclaré `inddir` à l'adresse 41002 mais on a stocké une valeur dedans qui était 1, un petit rappel de la manière dont on a procédé dans le code :
<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Initialise les 3 variables </span>
<span style="color:#75715e">; indcol: indicateur de collision adresse 41000</span>
<span style="color:#75715e">; indbal: indicateur de balle tirée adresse 41001</span>
<span style="color:#75715e">; inddir: indicateur de direction de l'ennemi adresse 41002</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indcol],a</span>       <span style="color:#75715e">; indcol = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indbal],a</span>       <span style="color:#75715e">; indbal = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[inddir],a</span>       <span style="color:#75715e">; inddir = 1</span>
</pre>

La stratégie va être que dans la boucle principale du programme, on va appeler une fois par boucle la routine de déplacement de l'ennemi.

Avant de le déplacer, on va aller consulter la valeur de cet indicateur :

-  s'il est à 1 on appellera une routine pour le déplacer de droite à gauche, 
- s'il est à 0 on appellera une routine qui le déplace de gauche à droite.

Mais ce n'est pas tout, avant d'incrémenter le déplacement dans les sous-routine on vérifiera que la valeur de la coordonnée (ici, **X**) n'a pas atteint une extrémité de l'écran.

Si c'est le cas, on va aller changer la valeur de l'indicateur à l'adresse 41002 pour qu'il se déplace dans l'autre sens à la prochaine boucle et ce, jusqu'à l'extrémité opposée. un petit schéma :
![schema](/tuto asm 01/10/images/1.png?classes=shadow)

{{% notice note %}}
Pourquoi x=240 à droite alors que la résolution va jusqu'à 256 ?
{{% /notice %}}

En fait les coordonnées des sprites correspondent au premier pixel en haut à gauche de chaque sprite.

Nos sprites étant agrandis à une taille x2, ils font 16 pixels sur 16 pixels, donc quand notre sprite est à la coordonnée x=240, entre 240 et 256 ce sont les 16 pixels de l'envahisseur qui s'affichent.

>Si on décidait d'aller au-delà le sprite sortirait de l'écran car il a besoin de 16 pixels à droite pour apparaître.

---

Je vais d'abord vous décrire l'algorithme choisi pour le déplacement (qui commence par la routine `DEPENV`) et ensuite on rentrera dans le code :
![algo](/tuto asm 01/10/images/2.png?classes=shadow)

On a donc 4 routines à écrire :

- **DEPENV** (comme 'déplacement envahisseur')  
- **ENDRO1** (envahisseur droite)  
- **ENGAU** (envahisseur à gauche)  
- **ENGAU1** (sous routine envahisseur à gauche)


Dans la boucle principale du programme on appellera la routine `DEPENV` qui  dispatchera le travail entre  les sous-routines en fonction de l'état de l'indicateur `inddir` et de l'état de la valeur de la coordonnée X du sprite envahisseur.

Passons au code que l'on rajoute en dessous de la routine `DELAI` (mais on pourrait le mettre ailleurs vu qu'on utilise des labels pour les appeler) :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de l'envahisseur / Gestion de la direction de l'envahisseur</span>
<span style="color:#75715e">; indir=1 : de droite à gauche / indir=0 : de gauche à droite</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">DEPENV:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[inddir]</span>   <span style="color:#75715e">; on charge en a la valeur de l'indicateur de direction</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on décremente a pour vérifier si indir=1</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,ENGAU</span>      <span style="color:#75715e">; si a=0 c'est que inddir valait 1,on va au sous-prog ENGAU (à gauche)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>      <span style="color:#75715e">; sinon on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">sub</span>    <span style="color:#ae81ff">240</span>          <span style="color:#75715e">; on elève 240 à a</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">nz,ENDRO1</span>    <span style="color:#75715e">; si le résultat n'égale pas 0 alors on va en ENDRO1 (à droite)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>          <span style="color:#75715e">; sinon on est au bord droit de l'écran et on change l'indicateur</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">[inddir],a</span>   <span style="color:#75715e">; on charge l'indicateur avec la nouvelle direction (1)</span>
<span style="color:#a6e22e">ret</span>
</pre>

La 1ere chose que fait la routine `DEPENV`, c'est de vérifier la valeur de l'indicateur `inddir`, s'il vaut 1 c'est que l'envahisseur doit se déplacer vers la gauche (0 vers la droite), il suffit de faire un calcul sur la valeur de l'inddir.

Ici on lui soustrait 1 :

- si le résultat est égale à 0 c'est que **inddir=1** quand on l'a appelé, dans ce cas on appelle la sous-routine `ENGAU` qui gère ce type de déplacement.  
- sinon (**inddir=0** car 0-1 n'égale pas 0) on récupère le X du sprite envahisseur à l'adresse correspondante.

la TAS abrite ces informations, elle débute à 6912 et il faut 4 adresses pour chaque sprites, donc pour l'envahisseur c'est 6917 qui abrite la valeur du X....  
...bon allez, je vous fais un petit rappel de ce qu'on a rentré dans la TAS pour ces SPRITES :

![schema](/tuto asm 01/10/images/3.png?classes=shadow)

Donc on récupère la valeur de la coordonnée X du sprite, comme on compare avec zéro ou pas zéro, pour vérifier si X=240, il suffit de faire un calcul dessus, si tel n'est  pas le cas alors la sous-routine `ENDRO1` fera le boulot de déplacement vers la droite.

Si par contre on est déjà à l'extrème droite de l'écran, on ne fait que changer la valeur de inddir pour qu'à la prochaine boucle l'envahisseur se déplace vers la gauche.


Je vous donne les autres sous-routines, elles sont maintenant simples à comprendre :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de l'envahisseur vers la droite</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">ENDRO1:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>      <span style="color:#75715e">; on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on incrémente a pour déplacer de 1 à droite (x+1)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; on charge la position x de la TAS avec le x+1</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; on retourne au programme principal</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Changement de direction (0) si l'envahisseur est arrivé au bord gauche x=0</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">ENGAU:</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>       <span style="color:#75715e">; on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>  <span style="color:#ae81ff">74</span>            <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">SUB</span>   <span style="color:#ae81ff">1</span>             <span style="color:#75715e">; on enlève 1 à a (x-1)</span>
<span style="color:#a6e22e">jp</span>    <span style="color:#f8f8f2">nz,ENGAU1</span>     <span style="color:#75715e">; si maintenant n'est pas égale à 0 on va en ENGAU1</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">0</span>           <span style="color:#75715e">; sinon on est au bord gauche de l'écran et on change l'indicateur</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">[inddir],a</span>    <span style="color:#75715e">; on charge l'indicateur avec la nouvelle direction (0)</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de l'envahisseur vers la gauche</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">ENGAU1:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>      <span style="color:#75715e">; on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on décrémente a pour déplacer de 1 à gauche (x-1)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; on charge la position x de la TAS avec le x+1</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; on retourne au programme principal</span>
</pre>

Ce code est à ajouter en dessous de la routine `DEPENV`.

Elles sont simple à comprendre, surtout avec le schéma de l'algorithme que je vous ai fait plus haut.
La seule subtilité c'est les routines qui ont pour rôle de calculer le prochain déplacement effectif de l'envahisseur (`ENGAU1` et `ENDRO1`) finissent par un `CALL 77` qui est **l'instruction qui modifie vraiment la coordonnée X du SPRITE dans la TAS**, sans ce CALL 77 les autres calculs que l'on a fait étaient sans impact sur la TAS. Ils étaient un peut comme quand l'on fait des calculs sur le coin d'un cahier de brouillon.

Il ne nous reste plus qu'à rajouter une instruction dans la boucle principale qui appellera le `DEPENV` et c'est `CALL DEPENV` (c'est la ligne 8 du code ci-dessous) :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Boucle du jeu : Le programme démarre à DEBUT puis va à BOUCLE</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>

<span style="color:#f8f8f2">DEBUT:</span>              <span style="color:#75715e">; c'est ici que débute le programme !!!</span>
<span style="color:#a6e22e">call</span>  <span style="color:#f8f8f2">INIT</span>          <span style="color:#75715e">; on appelle le sous-prog INIT</span>
<span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; c'est la boucle principale du programme !!!</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPENV</span>       <span style="color:#75715e">; on appelle le déplacement de l'envahisseur</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on appelle le sous-prog DELAI afin de ralentir l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche gauche = ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; (bios) SNSMAT retourne dans la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">4</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; bit teste la colonne 4 (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSGAU</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSGAU (déplacer à droite)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; la touche droite est aussi sur la ligne 8 de la matrice</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; on appelle le bios pour qu'il donne la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">7</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; la colonne 7 est testée (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSDRO</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSDRO (déplacer àgauche)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">BOUCLE</span>       <span style="color:#75715e">; on redémarre la boucle principale</span>
</pre>

Et pour finir le code entier du programme mis à jour qu'il ne vous reste plus qu'à compiler et lancer dans l'émulateur blueMSX :
<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"test3.ROM"</span>
<span style="color:#66d9ef">cpu</span> <span style="color:#f8f8f2">z80</span>

<span style="color:#66d9ef">ORG</span> <span style="color:#ae81ff">4000h</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Include.asm"</span>

<span style="color:#66d9ef">db</span> <span style="color:#e6db74">"AB"</span>
<span style="color:#66d9ef">DW</span> <span style="color:#f8f8f2">DEBUT</span>            <span style="color:#75715e">; la ROM lance le sous-programme DEBUT au démarrage</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#a6e22e">DS</span> <span style="color:#ae81ff">6</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Iinitialisation, appelé par le début du programme : DEBUT</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">INIT:</span>
<span style="color:#a6e22e">Ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; 1 pour screen 1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],a</span>      <span style="color:#75715e">; (bios)SCRMOD adresse hexa $FCAF :mode courant de l'ecran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">95</span>             <span style="color:#75715e">; (bios)CHGMOD : change le mode l'écran</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Charge la TGS avec la forme des 3 sprites</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,donnee</span>      <span style="color:#75715e">; charge les données TGS via le label: donnee</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">14336</span>       <span style="color:#75715e">; adresse du début de la TGS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">24</span>          <span style="color:#75715e">; longueur du bloc = 3 sprites de 8 octets = 24</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">92</span>             <span style="color:#75715e">; LIDRVM transfert de bloc de la RAM vers la VRAM</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Charge la TAS avec les attributs des 3 sprites  (registre 5)</span>
<span style="color:#75715e">; Registre 5 =&gt; valeur x 128 = Début de l'adresse de la TAS</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">c,</span><span style="color:#ae81ff">5</span>            <span style="color:#75715e">; registre 5 gère l'adresse de début de la TAS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">b,</span><span style="color:#ae81ff">54</span>           <span style="color:#75715e">; valeur 54 = positionne début de la TAS à l'adresse 6912</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">71</span>             <span style="color:#75715e">; WRTVDP Ecrire dans le VDP (VRAM)</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,donne1</span>      <span style="color:#75715e">; charge les données TAS au label: donne1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">6912</span>        <span style="color:#75715e">; adresse du début de la TAS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">12</span>          <span style="color:#75715e">; longueur du bloc = 4 données pour chaque sprite (3 sprites)</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">92</span>             <span style="color:#75715e">; LIDRVM transfert de bloc de la RAM vers la VRAM</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Initialise les 3 variables </span>
<span style="color:#75715e">; indcol: indicateur de collision adresse 41000</span>
<span style="color:#75715e">; indbal: indicateur de balle tirée adresse 41001</span>
<span style="color:#75715e">; inddir: indicateur de direction de l'ennemi adresse 41002</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indcol],a</span>       <span style="color:#75715e">; indcol = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indbal],a</span>       <span style="color:#75715e">; indbal = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[inddir],a</span>       <span style="color:#75715e">; inddir = 1</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Taille des sprites agrandis (registre 1)</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">c,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; registre 1 gère la taille des sprites</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">b,</span><span style="color:#ae81ff">225</span>          <span style="color:#75715e">; 225 c'est la valeur pour agrandir un sprite 8 x 8</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">71</span>             <span style="color:#75715e">; WRTVDP Ecrire dans un registre du VDP (VRAM)</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Positionne l'adresse de début de la TNP (registre 2)</span>
<span style="color:#75715e">; Registre 2 =&gt; valeur x 1024 = Début de l'adresse de la TNP</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">c,</span><span style="color:#ae81ff">2</span>            <span style="color:#75715e">; registre 2</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">b,</span><span style="color:#ae81ff">6</span>            <span style="color:#75715e">; valeur 6 = positionne début de la TNP à l'adresse 6144</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">71</span>             <span style="color:#75715e">; WRTVDP Ecrire dans un registre du VDP (VRAM)</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; fin de la boucle INIT</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de la fusée à gauche</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">FUSGAU:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6913</span>      <span style="color:#75715e">; récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction charge la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on teste A en le décrementant d'une unité x-1</span>
<span style="color:#a6e22e">ret</span>    <span style="color:#f8f8f2">z</span>            <span style="color:#75715e">; si le résultat de l'opération x=0 on ne fait rien</span>
                    <span style="color:#75715e">; et on quitte la sous-routine</span>
                    <span style="color:#75715e">; car cela veut dire que la fusée est à la limite de l'écran à gauche</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; sinon on charge la position x de la TAS avec la nouvelle valeur x-1</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; on retourne au programme principal</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de la fusée droite</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">FUSDRO:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6913</span>      <span style="color:#75715e">; récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction charge la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">SUB</span>    <span style="color:#ae81ff">240</span>          <span style="color:#75715e">; on teste A en lui enlevant 240</span>
<span style="color:#a6e22e">ret</span>    <span style="color:#f8f8f2">z</span>            <span style="color:#75715e">; si le résultat de l'opération=0 on ne fait rien </span>
                    <span style="color:#75715e">; et on quitte la sous-routine</span>
                    <span style="color:#75715e">; car cela veut dire que la fusée est à la limite de l'écran à droite</span>
<span style="color:#a6e22e">add</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">241</span>        <span style="color:#75715e">; rajoute les 240 pour retrouver le x initial et incrémente de 1(donc 241)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; sinon on charge la position x de la TAS avec la nouvelle valeur x+1</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; on retourne au programme principal</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; DELAI</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">DELAI:</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">255</span>         <span style="color:#75715e">; c'est la valeur de départ, plus elles est haute, plus c'est lent</span>
                    <span style="color:#75715e">; a ne peut contenir qu'une valeur entre 0 et 255</span>
<span style="color:#f8f8f2">DEL:</span>
<span style="color:#a6e22e">dec</span>   <span style="color:#f8f8f2">a</span>             <span style="color:#75715e">; c'est la boucle qui va décrémenter de 1 jusqu'à a=0</span>
<span style="color:#a6e22e">jp</span>    <span style="color:#f8f8f2">nz,DEL</span>        <span style="color:#75715e">; si a n'est pas égale à 0 on repart au début de la boucle</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; si a=0 on retourne au programme principal</span>


<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de l'envahisseur / Gestion de la direction de l'envahisseur</span>
<span style="color:#75715e">; indir=1 : de droite à gauche / indir=0 : de gauche à droite</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">DEPENV:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[inddir]</span>   <span style="color:#75715e">; on charge en a la valeur de l'indicateur de direction</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on décremente a pour vérifier si indir=1</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,ENGAU</span>      <span style="color:#75715e">; si a=0 c'est que inddir valait 1,on va au sous-prog ENGAU (à gauche)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>      <span style="color:#75715e">; sinon on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">sub</span>    <span style="color:#ae81ff">240</span>          <span style="color:#75715e">; on elève 240 à a</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">nz,ENDRO1</span>    <span style="color:#75715e">; si le résultat n'égale pas 0 alors on va en ENDRO1 (à droite)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>          <span style="color:#75715e">; sinon on est au bord droit de l'écran et on change l'indicateur</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">[inddir],a</span>   <span style="color:#75715e">; on charge l'indicateur avec la nouvelle direction (1)</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de l'envahisseur vers la droite</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">ENDRO1:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>      <span style="color:#75715e">; on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on incrémente a pour déplacer de 1 à droite (x+1)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; on charge la position x de la TAS avec le x+1</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; on retourne au programme principal</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Changement de direction (0) si l'envahisseur est arrivé au bord gauche x=0</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">ENGAU:</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>       <span style="color:#75715e">; on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>  <span style="color:#ae81ff">74</span>            <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">SUB</span>   <span style="color:#ae81ff">1</span>             <span style="color:#75715e">; on enlève 1 à a (x-1)</span>
<span style="color:#a6e22e">jp</span>    <span style="color:#f8f8f2">nz,ENGAU1</span>     <span style="color:#75715e">; si maintenant n'est pas égale à 0 on va en ENGAU1</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">0</span>           <span style="color:#75715e">; sinon on est au bord gauche de l'écran et on change l'indicateur</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">[inddir],a</span>    <span style="color:#75715e">; on charge l'indicateur avec la nouvelle direction (0)</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Déplacement de l'envahisseur vers la gauche</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">ENGAU1:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6917</span>      <span style="color:#75715e">; on récupère adresse de la position x du sprite dans la TAS</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; la fonction récupère la valeur de l'adresse dans A donc x=A</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on décrémente a pour déplacer de 1 à gauche (x-1)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; on charge la position x de la TAS avec le x+1</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; on retourne au programme principal</span>


<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Boucle du jeu : Le programme démarre à DEBUT puis va à BOUCLE</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>

<span style="color:#f8f8f2">DEBUT:</span>              <span style="color:#75715e">; c'est ici que débute le programme !!!</span>
<span style="color:#a6e22e">call</span>  <span style="color:#f8f8f2">INIT</span>          <span style="color:#75715e">; on appelle le sous-prog INIT</span>
<span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; c'est la boucle principale du programme !!!</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPENV</span>       <span style="color:#75715e">; on appelle le déplacement de l'envahisseur</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on appelle le sous-prog DELAI afin de ralentir l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche gauche = ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; (bios) SNSMAT retourne dans la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">4</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; bit teste la colonne 4 (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSGAU</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSGAU (déplacer à droite)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; la touche droite est aussi sur la ligne 8 de la matrice</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; on appelle le bios pour qu'il donne la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">7</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; la colonne 7 est testée (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSDRO</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSDRO (déplacer àgauche)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">BOUCLE</span>       <span style="color:#75715e">; on redémarre la boucle principale</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Les données</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">donnee:</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span>  <span style="color:#75715e">; sprite 1 pour la TGS</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span>  <span style="color:#75715e">; sprite 1   "     "</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">60</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">153</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span> <span style="color:#75715e">; sprite 2   "     "</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">102</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">60</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">66</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">36</span>   <span style="color:#75715e">; sprite 2   "     "</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span>    <span style="color:#75715e">; sprite 3   "     "</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span>    <span style="color:#75715e">; sprite 3   "     "</span>
<span style="color:#f8f8f2">donne1:</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">170</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">100</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">15</span>   <span style="color:#75715e">; Fusée joueur sprite 0 (position y, x,n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">12</span>       <span style="color:#75715e">; Envahisseur  sprite 1 (position y, x,n°sprite,couleur)  </span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">200</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">2</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span>      <span style="color:#75715e">; Laser fusée  sprite 2 (position y, x,n°sprite,couleur)</span>



<span style="color:#75715e">;**************************************************************************************************</span>
<span style="color:#75715e">; Standard Libraries</span>
<span style="color:#75715e">;**************************************************************************************************</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Lib.asm"</span> <span style="color:#75715e">; intègre la librairie MSX</span>

<span style="color:#f8f8f2">END:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#66d9ef">$</span>
<span style="color:#f8f8f2">indcol:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41000</span>       <span style="color:#75715e">; défini l'adresse de la variable indcol (registre des collisions)</span>
<span style="color:#f8f8f2">indbal:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41001</span>       <span style="color:#75715e">; défini l'adresse de la variable indbal</span>
<span style="color:#f8f8f2">inddir:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41002</span>       <span style="color:#75715e">; défini l'adresse de la variable inddir</span>

<span style="color:#75715e">;**************************************************************************************************</span>
<span style="color:#75715e">; RAM Definitions</span>
<span style="color:#75715e">;**************************************************************************************************</span>

<span style="color:#75715e">;-------------------</span>
<span style="color:#75715e">; Définit que le programme démarrera au début de la RAM disponible</span>
<span style="color:#75715e">;-------------------</span>
<span style="color:#66d9ef">ORG</span> <span style="color:#f8f8f2">RAMSTART<br /></span> </pre>

