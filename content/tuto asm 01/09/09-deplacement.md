---
title: "09-Déplacement du joueur"
weight: 9
---

 Maintenant que les SPRITES sont affichés, on va commencer à faire bouger la fusée du joueur.

Pour cela on va un peu modifier la structure du code assembleur.
Il y surement d'autres méthodes, mais c'est celle présentée par le livre, donc je l'ai suivi.

Jusqu'à présent la ROM démarre et appelle tout de suite le sous-programme `INIT`. Pourquoi ? Ben, parce qu'on l'a spécifié dans l'entête du code, souvenez vous le `DW INIT` :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"test1.ROM"</span>
<span style="color:#66d9ef">cpu</span> <span style="color:#f8f8f2">z80</span>

<span style="color:#66d9ef">ORG</span> <span style="color:#ae81ff">4000h</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Include.asm"</span>

<span style="color:#66d9ef">db</span> <span style="color:#e6db74">"AB"</span>
<span style="color:#66d9ef">DW</span> <span style="color:#f8f8f2">INIT</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#a6e22e">DS</span> <span style="color:#ae81ff">6</span>
</pre>

Bon et bien maintenant on ne veut pas simplement faire une initialisation (accéder au SCREEN 1 et afficher les SPRITES), on veut installer une partie de code qui va devenir le programme principal, c'est par lui que la ROM va débuter. 

C'est encore lui qui va se charger d'initialiser l'affichage la première fois, et qui ensuite, va continuellement surveiller le clavier (via une boucle infinie) et appeler momentanément des sous-programmes de déplacement en fonction de la touche qu'il a détecté.

Bon, on va mieux visualiser par un schéma :

![orga](/tuto asm 01/09/images/1.png?classes=shadow)

Ainsi la ROM ne va plus appeler `INIT` mais un label que j'appelle `DEBUT` qui va se charger d'appeler le sous-programme (ou routine) `INIT`.
Ensuite `DEBUT` se termine et donne la main à un nouveau label (ou étiquette) qui s'appelle `BOUCLE` (c'est notre boucle sans fin) qui va tourner en boucle justement en appelant une routine de temporisation (qui ralenti l'affichage écran) et qui surveille le clavier.

Si pendant cette boucle infinie, une touche directionnelle est enfoncée alors on fait appel à la routine de déplacement de la fusée.

Tous les sous-programmes sont codés de façon à retourner au programme principal une fois leur boulot exécuté.

Voilà à quoi va ressembler notre programme principal:

![orga 1](/tuto asm 01/09/images/2.png?classes=shadow)

Dans l'article précédent nous avons déjà codé la routine `INIT:` (INITIALISATION), Il reste à coder le programme principal, les routines de temporisation et de déplacement.
Mais avant, modifions notre entête pour que la ROM lance `DEBUT` au démarrage et non plus `INIT` :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"test2.ROM"</span>
<span style="color:#66d9ef">cpu</span> <span style="color:#f8f8f2">z80</span>

<span style="color:#66d9ef">ORG</span> <span style="color:#ae81ff">4000h</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Include.asm"</span>

<span style="color:#66d9ef">db</span> <span style="color:#e6db74">"AB"</span>
<span style="color:#66d9ef">DW</span> <span style="color:#f8f8f2">DEBUT</span>            <span style="color:#75715e">; la ROM lance le sous-programme DEBUT au démarrage</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#a6e22e">DS</span> <span style="color:#ae81ff">6</span>
</pre>

#### PROGRAMME PRINCIPAL - DEBUT

Je vous livre le code par morceaux, ne vous occupez pas de savoir où le placer dans le fichier, je vous ferai une copie intégrale du code à la fin pour que vous voyez l'organisation.

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Boucle du jeu : Le programme démarre à DEBUT puis va à BOUCLE</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>


<span style="color:#f8f8f2">DEBUT:</span>              <span style="color:#75715e">; c'est ici que débute le programme !!!</span>
<span style="color:#a6e22e">call</span>  <span style="color:#f8f8f2">INIT</span>          <span style="color:#75715e">; on appelle le sous-prog INIT</span>
</pre>

Rien de plus simple pour commencer. Vous remarquez qu'il n'y a pas d'instruction `ret` (RETURN) à la fin de cette routine. Ainsi le code va continuer au label déclaré juste en dessous puisqu'il n'est pas arrêté ni re-routé.
Ca tombe bien puisque le prochain label que l'on va déclarer ensuite est `BOUCLE`

#### PROGRAMME PRINCIPAL - BOUCLE

<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; c'est la boucle principale du programme !!!</span>
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

C'est bien le coeur du programme et il va tourner en boucle (comme son label l'indique), d'ailleurs la dernière ligne de cette partie du code se termine par un `JP BOUCLE` : Saute au début du label `BOUCLE:`.

Voici ce que fait la boucle :

- Appelle une routine `DELAI`.
Verifie si la touche gauche est enfoncée, si tel est le cas, elle appelle la routine de déplacement du sprite vers la gauche `FUSGAU`. Cette routine lui rendra la main à la fin.
- Appelle la routine `DELAI`.
Verifie si la touche droite est enfoncée, si tel est le cas, elle appelle la routine de déplacement du sprite vers la droite `FUSDRO`. Cette routine lui rendra la main à la fin.
Appelle la routine `DELAI`.
- Retourne au début de la boucle.

0n voit déjà que l'on aura 3 routines à écrire par la suite :

-Routine **DELAI**  
- Routine **FUSGAU**  
- Routine **FUSDRO**  

Mais revenons à notre boucle pour l'instant. On voit qu'après avoir appelé la routine `DELAI`, on vérifie l'enfoncement de la touche gauche, comment ça marche ?

En fait, on se base sur une matrice de clavier et une instruction du BIOS qui surveille cette matrice :

![orga 1](/tuto asm 01/09/images/3.png?classes=shadow)
<center>*MATRICE CLAVIER*</center>

Et l'instruction SNSMAT du BIOS c'est le `CALL 321` (ou call 141 en hexa) :

>SNSMAT  
Address  : #0141  
Function : Returns the value of the specified line from the keyboard matrix  
Input    : A  - for the specified line  
Output   : A  - for data (the bit corresponding to the pressed key will be 0)  
Registers: AF

Cette instruction BIOS nous dit : Donnez-moi dans A la ligne qui vous intéresse, et je vous charge dans A les infos des bit0 à 7 de cette ligne.

Nous on veut la touche gauche, elle correspond à la ligne 8, donc si on **charge A avec 8** et qu'on appelle la routine, on récupère toutes les infos de cette ligne dans **A**.

**La touche gauche c'est le bit 4** (colonnes de la matrice), l'instruction BIOS nous dit que si la touche est enfoncée alors le bit correspondant égale 0.

<pre style="line-height:125%;margin:0"><span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche gauche = ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; (bios) SNSMAT retourne dans la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">4</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; bit teste la colonne 4 (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSGAU</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSGAU (déplacer à droite)</span>
</pre>


C'est exactement ce que l'on fait avec `BIT 4,A` : on teste A avec bit 4 et la ligne suivante on appelle la routine `FUSGAU` si A est à zéro `call z,FUSGAU`.

Je vous laisse chercher pour le curseur à droite et l'appel à la routine correspondante.

#### DELAI
Cette routine permet de ralentir le déplacement de la fusée entre autre sinon cela irait trop vite et on ne verrait pas notre fusée se déplacer de façon douce.

Ce n'est rien d'autre qu'une boucle qui va compter de 255 à 0 sans rien faire d'autre avant de rendre la main au programme principal.

Cette routine a 2 labels:

- Le label `DELAI`, qui est initialisation du compteur à 255 passe le relai au label `DEL` qui va faire le travail de comptage.  
- Le label `DEL` va décrémenter A d'une unité `DEC A`, si A n'égale pas zéro (NZ=Non Zéro) alors on revient au début de `DEL`  
- Si A=0 alors on ne fait plus de `JUMP` (puisque le résultat n'est pas vérifié) mais on fait un retour au programme qui l'a appelé (programme principal) via un `RET` (return).

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; DELAI</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">DELAI:</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">255</span>         <span style="color:#75715e">; c'est la valeur de départ, plus elles est haute, plus c'est lent</span>
                    <span style="color:#75715e">; a ne peut contenir qu'une valeur entre 0 et 255</span>
<span style="color:#f8f8f2">DEL:</span>
<span style="color:#a6e22e">dec</span>   <span style="color:#f8f8f2">a</span>             <span style="color:#75715e">; c'est la boucle qui va décrémenter de 1 jusqu'à a=0</span>
<span style="color:#a6e22e">jp</span>    <span style="color:#f8f8f2">nz,DEL</span>        <span style="color:#75715e">; si a n'est pas égale à 0 on repart au début de la boucle</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; si a=0 on retourne au programme principal</span>
</pre>

Vous pourrez vous amuser à changer la valeur 255 par 1 pour voir le résultat.


#### FUSGAU

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
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
</pre>

Quand on arrive dans cette routine c'est que la touche gauche a été enfoncée. 

Donc on va devoir déplacer notre fusée vers la gauche d'un pixel.
Pour cela il nous faut la coordonnée horizontale de notre fusée.

Rien de plus simple, quand on a chargé les attributs des sprites dans la TAS, on a commencée à les enregistrer à l'adresse 6912 (début de la TAS) et en plus le 1er sprite pour lequel on a chargé les infos (4 données: position y, position x, numéro de sprite, couleur) c'est notre fusée. Donc les données de notre sprite sont aux adresses suivantes :

- Adresse 6912 : position y  
- Adresse 6913 : position x  
- Adresse 6914 : numéro du sprite  
- Adresse 6915 : couleur du sprite  

Et ainsi de suite pour les sprites suivants (vous devriez donc connaître les coordonées du sprite ennemi...en tout cas leurs adresses).

Dans notre cas, la fusée, on sait que la position x est à l'adresse 6913.

Il suffit donc de l'appeler, de faire des calculs sur la valeur x et de la recharger dans la TAS pour que notre sprite prenne en compte la nouvelle position.

Pour lire la valeur d'une adresse de la VRAM on fait un `CALL 74` (CALL 4A) qui chargera la valeur dans l'accumulateur A :

>RDVRM  
Address  : #004A  
Function : Reads the content of VRAM  
Input    : HL - address read  
Output   : A  - value which was read  
Registers: AF

On soustrait 1 unité, et oui pour aller à gauche on revient vers la coordonnée 0 de l'écran.
On retourne au programme principale sans rien faire si A=0 (RET Z) car cela veut dire que l'on est déjà à l'extrémité gauche de l'écran.
Sinon on continue dans le code, on a déjà décrémenté A, donc il nous reste plus qu'à écrire à l'adresse chargée dans l'accumulateur HL la nouvelle valeur via un `CALL 77` (CALL 4D) :

>WRTVRM  
Address  : #004D  
Function : Writes data in VRAM  
Input    : HL - address write  
           A  - value write  
Registers: AF

Et on retourne au programme principal (RET).


#### FUSDRO
C'est pareil que pour la routine précédente. Seul change le test de l'extrémité de l'écran à droite qui est fixé à 240 (240+ 16 pixels de notre sprite =256.....un sprite de 8 mais agrandi x 2).

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
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
</pre>

Comme pour l'instant on pratique des calculs qui nous permettent de comparer à 0 ou non 0 , pour vérifier si x est déjà à 240, il suffit de luis soustraire 240 et de comparer à 0, sinon on lui rajoute les 240 pour le ramener à l'état dans lequel on la récupéré et on lui rajoute 1 pour qu'il se déplace à droite.
Ce code devrait s'abstenir de plus de commentaires désormais.


Je vous livre le code en entier pour que vous voyez l'ordre des routines insérées bien qu'il n'y ait pas grande importance (sauf pour celles où on n'a pas mis de `RET` pour qu'elles enchaînent sur le prochain label en dessous, ex :`DEBUT` et `BOUCLE`) puisqu'une fois la ROM démarrée, elle va appeler la routine 'DEBUT' et les sauts vont s'enchaîner comme on l'a codé. Compilez, jouez vous devriez pouvoir déplacer votre fusée :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"test2.ROM"</span>
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
<span style="color:#a6e22e">LD</span>   <span style="color:#f8f8f2">A,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; 1 pour screen 1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],A</span>      <span style="color:#75715e">; (bios)SCRMOD adresse hexa $FCAF :mode courant de l'ecran</span>
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
<span style="color:#75715e">; Boucle du jeu : Le programme démarre à DEBUT puis va à BOUCLE</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>


<span style="color:#f8f8f2">DEBUT:</span>              <span style="color:#75715e">; c'est ici que débute le programme !!!</span>
<span style="color:#a6e22e">call</span>  <span style="color:#f8f8f2">INIT</span>          <span style="color:#75715e">; on appelle le sous-prog INIT</span>
<span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; c'est la boucle principale du programme !!!</span>
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
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">170</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">100</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">15</span>   <span style="color:#75715e">; Attributs du sprite 0 (position y, x, n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">12</span>       <span style="color:#75715e">; Attributs du sprite 1 (position y, x, n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">200</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">2</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span>      <span style="color:#75715e">; Attributs du sprite 2 (position y, x, n°sprite,couleur)</span>



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
<span style="color:#66d9ef">ORG</span> <span style="color:#f8f8f2">RAMSTART</span>
</pre>



