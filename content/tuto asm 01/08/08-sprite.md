---
title: "08-Afficher un sprite"
weight: 8
---

## La pratique

On va maintenant taper notre 1er code du shoot-em-up.  

On va enlever la partie du code correspondant à l'affichage du caractère (qui était notre exemple précédent), et donc repartir du fichier sources vierge `exemple.asm`.

>Quelques informations sur le début du code : 

- Le programme est en SCREEN 1
3 SPRITES vont être créés (la fusée du joueur, l'ennemi/envahisseur, le tir)  
- On charge les formes des SPRITES dans la table TGS  
- On charge les attributs des SPRITES dans la table TAS   
- On va positionner 3 indicateurs qui nous serviront à déplacer les SPRITES plus tard.  
- On spécifie la taille des SPRITES

Voici l'entête, j'ai juste modifié le nom du fichier qui sera généré via `FNAME` :
<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"test1.ROM"</span>
<span style="color:#66d9ef">cpu</span> <span style="color:#f8f8f2">z80</span>

<span style="color:#66d9ef">ORG</span> <span style="color:#ae81ff">4000h</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Include.asm"</span>

<span style="color:#66d9ef">db</span> <span style="color:#e6db74">"AB"</span>
<span style="color:#66d9ef">DW</span> <span style="color:#f8f8f2">INIT</span>             <span style="color:#75715e">; Fait démarrer la ROM au label INIT</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#a6e22e">DS</span> <span style="color:#ae81ff">6</span>
</pre>

---

#### SCREEN 1  
Passons au début du code proprement dit et par le label (appelé aussi "Etiquette") 'INIT:' (c'est par lui que la .ROM démarrera) :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Démarrage du programme à INIT</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">INIT:</span>
<span style="color:#a6e22e">LD</span>   <span style="color:#f8f8f2">A,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; 1 pour screen 1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],A</span>      <span style="color:#75715e">; (bios)SCRMOD adresse $FCAF :mode courant de l'ecran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">95</span>             <span style="color:#75715e">; (bios) CHGMOD : change le mode l'écran</span>
</pre>

Pas de modification par rapport aux précédent exemple, on se place en SCREEN 1.

---

#### CHARGEMENT DE LA TGS

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Charge la TGS avec la forme des 3 sprites</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,donnee</span>      <span style="color:#75715e">; va chercher les données TGS au label: donnee</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">14336</span>       <span style="color:#75715e">; adresse du début de la TGS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">24</span>          <span style="color:#75715e">; longueur du bloc = 3 sprites de 8 octets = 24</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">92</span>             <span style="color:#75715e">; LIDRVM transfert de bloc de la RAM vers la VRAM</span>
</pre>

Je commence par la fin, on va donner un ordre de transfert de données (les formes des SPRITES) qui seront écrites dans notre code (donc dans la RAM) vers la table TGS (donc la VRAM), on ne va pas pouvoir utiliser un ordre du style "écrire dans la VRAM" car cela est limité à une adresse mémoire et une donnée.  
Pour de gros **blocs de données entre RAM et VRAM**, il existe une routine BIOS qui fait le boulôt : LIDRVM ou en langage assembleur `CALL 92` (en décimal) ou `CALL $5C` (en hexadécimal).
Cette routine, pour être déclenchée, a besoin de 3 paramètres en entrée :

- **HL**, pour les données des formes de sprites  
- **DE**,  l'adresse de la TGS à partir de laquelle on veut écrire (une donnée = une case mémoire)  
- **BC**, la longueurde bloc total à transférer (en octets)

Voici d'ailleurs une définition de cette routine :

>LDIRVM  
Address  : #005C  
Function : Block transfer to VRAM from memory  
Input    : BC - blocklength  
           DE - Start address of VRAM  
           HL - Start address of memory  
Registers: All

Vu que l'on a **3 SPRITES** à charger (8 pixels sur 8 pixels donc 8 octets) on a 3 x  8 = **24 données à envoyer en VRAM**. On ne va pas les taper juste après la virgule du HL il ne saurait pas le traiter.
On va plutôt lui dire où il peut les trouver et les "digérer à son rythme", pour cela on va créer un nouveau label `donnee` qui abritera ces 24 données.

Les données en général sont stockées plutôt à la fin du programme codé et chaque ligne doit commencer par un `DB`.

Ce label `donnee` je l'ai placé juste avant le pied de page, voici comment il se présente :
<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Les données</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">donnee:</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span>  <span style="color:#75715e">; sprite 0 pour la TGS</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span>  <span style="color:#75715e">; sprite 0</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">60</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">153</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span> <span style="color:#75715e">; sprite 1</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">102</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">60</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">66</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">36</span>   <span style="color:#75715e">; sprite 1</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span>    <span style="color:#75715e">; sprite 2</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span>    <span style="color:#75715e">; sprite 2</span>
</pre>


Pour le **SPRITE 0**, les 2 premières lignes donnent 8 données qui forment un SPRITE joueur (la fusée) : 
![sprite 0](/tuto asm 01/08/images/1.png?classes=shadow)

Pour le **SPRITE 1**, les 2 lignes suivantes qui donnent la forme suivante pour l'ennemi :
![sprite 1](/tuto asm 01/08/images/2.png?classes=shadow)

Pour le **SPRITE 2**, les 2 dernières lignes qui donnent la forme suivante pour le tir :

![sprite 2](/tuto asm 01/08/images/3.png?classes=shadow)

Du coup, tout devient simple:

- On dit à `HL` de se charger (via l'instruction 'LD') en allant chercher les données que l'on a mis en fin de code dont le label est `donnee`.  
- On charge en `DE` le début de **l'adresse de la TGS** (je vous ai dit dans un des premiers articles qu'elle débutait à **14336 en SCREEN 1**).  
- On charge en `BC` **la longueur du bloc** 'donnee'  
- On exécute `CALL 92` qui fera le boulot vu qu'on lui a donné les entrants.

---

#### CHARGEMENT DE LA TAS
<pre style="line-height:125%;margin:0"><span style="color:#75715e">;-------------------</span>
<span style="color:#75715e">; Charge la TAS avec les attributs des 3 sprites  (registre 5)</span>
<span style="color:#75715e">; Registre 5 =&gt; valeur x 128 = Début de l'adresse de la TAS</span>
<span style="color:#75715e">;-------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">c,</span><span style="color:#ae81ff">5</span>            <span style="color:#75715e">; registre 5 gère l'adresse de début de la TAS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">b,</span><span style="color:#ae81ff">54</span>           <span style="color:#75715e">; valeur 54 = positionne début de la TAS à l'adresse 6912</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">71</span>             <span style="color:#75715e">; WRTVDP Ecrire dans le VDP (VRAM)</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,donne1</span>      <span style="color:#75715e">; va chercher les données TAS au label: donne1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">6912</span>        <span style="color:#75715e">; adresse du début de la TAS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">12</span>          <span style="color:#75715e">; longueur du bloc = 4 données pour chaque sprite (3 sprites)</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">92</span>             <span style="color:#75715e">; LIDRVM transfert de bloc de la RAM vers la VRAM</span>
</pre>

J'y ai intégré un petit exercice de manière à progresser au fur et à mesure du code :

Je vous ai dit qu'en **SCREEN 1 la table des attributs de SPRITE (TAS) débutait à l'adresse 6912**.
Sur MSX on pourrait changer cette adresse pour diverses raisons (et d'ailleurs, ces adresses de tables sont succeptibles de changer en fonction des modes SCREEN que l'on utilise).

Admettons que l'on ne sait pas si la TAS est bien positionnée pour débuter à 6912.
Je vous ai expliqué précédemment qu'**en VRAM**, il existait des tables et des registres, et bien **chaque table est gérée via un de ces registres**. 

>**Pour notre TAS c'est le registre 5.**

Toute valeur que l'on écrira dans ce registre sera automatiquement multipliée par 128 et définira l'adresse de début de la table TAS.
Donc pour obtenir 6912, on le divise par 128 et cela donne 54.

>**Finalement, pour définir la TAS à 6912, il suffit d'écrire dans le registre 5 la valeur 54.**

C'est l'objet des 3 premières lignes du code ci-dessus jusqu'au `CALL 71` (ou 47 en hexa) qui est la routine **BIOS** qui existe pour écrire dans les **registres du VDP** :

>WRTVDP  
Address  : #0047  
Function : write data in the VDP-register  
Input    : B  - data to write  
           C  - number of the register  
Registers: AF, BC

Fin de l'exercice.

Passons au chargement de la TAS : d'abord la procédure va ressembler au PUTSPRITE du BASIC MSX donc cela devrait vous paraître simple et ensuite on va procéder au codage assembleur de la même manière que pour la TGS.

On va donner 4 infos pour chaque SPRITE créé :

- **La position verticale du SPRITE à l'écran**  
- **La position horizontale du SPRITE à l'écran**  
- **Le numéro du SPRITE**  
- **La couleur du SPRITE**

Mais avant de décrire le bout de code, il faut préciser une particularité importante des SPRITES, ils ne se localisent pas comme les caractères (32 colonnes sur 24 lignes) mais sur des coordonnées plus fines.

En SCREEN 1 c'est 256 pixels sur 192 pixels :  
Du coup, on va pas trop s'attarder sur la méthode car c'est la même que pour la TGS et le commentaire associé au code est explicite.
On va plutôt détailler le label `donne1` associé (que j'ai placé au niveau du code, après le label `donnee` et avant le pied de page).

![screen](/tuto asm 01/08/images/4.png?classes=shadow)  <center>*Résolution graphique SCREEN 1*</center>

.

<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">donne1:</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">170</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">100</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">15</span>   <span style="color:#75715e">; Attributs du sprite 0 (position y, x, n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">12</span>       <span style="color:#75715e">; Attributs du sprite 1 (position y, x, n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">200</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">2</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span>      <span style="color:#75715e">; Attributs du sprite 2 (position y, x, n°sprite,couleur)</span>
</pre>

Je vous avais dit qu'il y a 4 informations à rentrer par SPRITE dans la TAS.

Du coup, pour le SPRITE de la 1ere ligne, il aura au 1er affichage à l'écran la coordonnée vertical=170  horizontal=100 pour le SPRITE numéro 0 et qui aura la couleur 15 (blanc), vous avez reconnu le sprite joueur/fusée.

Je vous laisse décrypter les 2 autres lignes pour les 2 autres SPRITES.

Juste pour finir sur ce code, on voit que l'on a 12 données à rentrer, cela tombe bien c'est ce que l'on charge en `BC` avant de lancer le `CALL 92` qui devient familier pour vous maintenant.

---

#### 3 INDICATEURS DE DEPLACEMENT
Au début de l'article j'ai listé le fait que l'on allait positionner 3 indicateurs nécessaire au déplacement des sprites.
On ne va pas les utiliser de suite mais un minimum d'information quand même :

- **indcol** : un indicateur de collision
- **indbal** : un indicateur de balle tirée  
- **inddir**  : un indicateur de direction pour l'ennemi

Pour que le programme nous autorise à manipuler des valeurs avec ces termes, on doit d'abord les déclarer comme suit :

<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">indcol:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41000</span>       <span style="color:#75715e">; défini l'adresse de la variable indcol</span>
<span style="color:#f8f8f2">indbal:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41001</span>       <span style="color:#75715e">; défini l'adresse de la variable indbal</span>
<span style="color:#f8f8f2">inddir:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41002</span>       <span style="color:#75715e">; défini l'adresse de la variable inddir</span>
</pre>

Ainsi on dit au programme qu'à chaque fois que l'on utilisera `indcol` cela voudra dire en fait que l'on sollicite l'adresse 41000 de la RAM (un choix arbitraire d'adresse libre en RAM).
On pourrait traduire `EQU` par 'équivaut'.

Idem pour les 2 autres déclarations. ce code je l'ai placé dans le bas de page, mais vous pourriez le mettre dans l'entête, cela n'a pas grande importance.

Ensuite, je vais assigner des valeurs à ces adresses, pour cela je place le code suivant à la suite du code sur le chargement de la TAS puisque cela fait partie de l'initialisation du jeu :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Initialise les 3 variables </span>
<span style="color:#75715e">; indcol: indicateur de collision 41000 (adresse)</span>
<span style="color:#75715e">; indbal: indicateur de balle tirée  41001 (adresse)</span>
<span style="color:#75715e">; inddir: indicateur de direction de l'ennemi 41002 (adresse)</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indcol],a</span>       <span style="color:#75715e">; indcol = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indbal],a</span>       <span style="color:#75715e">; indbal = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[inddir],a</span>       <span style="color:#75715e">; inddir = 1</span>
</pre>

Rien d'exceptionnel, je charge ces pseudo adresses (pour cela, je les passe entre crochets) avec un A qui vaut 1 puisque dans la 1ere ligne j'ai assigné 1 à l'accumulateur **A**.

---

#### TAILLE DES SPRITES

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Taille des sprites agrandis (registre 1)</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">c,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; registre 1 gère la talle des sprites</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">b,</span><span style="color:#ae81ff">225</span>          <span style="color:#75715e">; 225 c'est la valeur pour agrandir un sprite 8 x 8</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">71</span>             <span style="color:#75715e">; WRTVDP Ecrire dans le VDP (VRAM)</span>
</pre>

Sur MSX on peut agir sur la taille des SPRITES, pour cela on utilise le registre 1  de la VRAM qui accepte 4 valeurs possibles :

>SPRITES taille 8 x 8 normal      = 224  
SPRITES taille 8 x 8 agrandis    = 225   
SPRITES taille 16 x 16 normal   = 226  
SPRITES taille 16 x 16 agrandis = 227

Vu que l'on a déclaré dans la TGS des SPRITES  de taille 8 x 8, on peut utiliser la valeur 224 ou 225 (vous pouvez vous amuser à changer la valeur pour voir la différence), dans notre jeux on va utiliser 225.

Le code devrait donc vous parler maintenant : on charge 'C' avec le numéro du registre 1 et 'B' avec la valeur à écrire dans ce registre, ce sont les 2 entrants nécessaires au `CALL 71` qui permet d'exécuter un ordre d'écriture dans un registre VRAM.

C'était le dernier bout de code du `label INIT:`  
Si on veut voir le résultat il suffit de rajouter une boucle infinie, pour que le programme enchaine dessus après avoir exécuté toutes les actions du `INIT:`, comme cela par exemple :

<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">LOOP:</span>
     <span style="color:#a6e22e">jp</span> <span style="color:#f8f8f2">LOOP</span>
</pre>

Vous pouvez faire **F9** dans CONTEXT, pour lancer la compilation, le fichier généré va s'appeler `"test1.ROM"` et c'est celui-là que vous faites glisser dans l'émulateur blueMSX.

Vous allez voir apparaître ceci :

![screen](/tuto asm 01/08/images/5.png?classes=shadow)

Mais où est passé le SPRITE représentant la balle (ou le lazer si vous voulez) ?

Le premier réflex est de regarder ce que l'on a déclaré comme coordonnées pour le SPRITE 2 qui le représente dans la TAS.

Et que voit-on ?   Position verticale = 200....alors que la résolution nous permet d'aller à un maximum de 191. En le positionnant au delà il disparait dans la bordure de l'écran mais il est déjà présent et il sera plus rapide à positionner correctement lorsqu'on tirera que si on devait le charger dans la TAS puis calculer sa position au moment du tir.

Si vous voulez le voir apparaitre et bien jouez avec ses coordonnées, compilez et essayez votre **.ROM** dans l'émulateur.

Voici le code complet de cet article :

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

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Démarrage du programme à INIT</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">INIT:</span>
<span style="color:#a6e22e">LD</span>   <span style="color:#f8f8f2">A,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; 1 pour screen 1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],A</span>      <span style="color:#75715e">; (bios)SCRMOD adresse $FCAF :mode courant de l'ecran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">95</span>             <span style="color:#75715e">; (bios) CHGMOD : change le mode l'écran</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Charge la TGS avec la forme des 3 sprites</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,donnee</span>      <span style="color:#75715e">; va chercher les données TGS au label: donnee</span>
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
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,donne1</span>      <span style="color:#75715e">; va chercher les données TAS au label: donne1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">6912</span>        <span style="color:#75715e">; adresse du début de la TAS</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">12</span>          <span style="color:#75715e">; longueur du bloc = 4 données pour chaque sprite (3 sprites)</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">92</span>             <span style="color:#75715e">; LIDRVM transfert de bloc de la RAM vers la VRAM</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Initialise les 3 variables </span>
<span style="color:#75715e">; indcol: indicateur de collision 41000 (adresse)</span>
<span style="color:#75715e">; indbal: indicateur de balle tirée  41001 (adresse)</span>
<span style="color:#75715e">; inddir: indicateur de direction de l'ennemi 41002 (adresse)</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indcol],a</span>       <span style="color:#75715e">; indcol = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[indbal],a</span>       <span style="color:#75715e">; indbal = 1</span>
<span style="color:#a6e22e">ld</span> <span style="color:#f8f8f2">[inddir],a</span>       <span style="color:#75715e">; inddir = 1</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Taille des sprites agrandis (registre 1)</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">c,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; registre 1 gère la talle des sprites</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">b,</span><span style="color:#ae81ff">225</span>          <span style="color:#75715e">; 225 c'est la valeur pour agrandir un sprite 8 x 8</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">71</span>             <span style="color:#75715e">; WRTVDP Ecrire dans le VDP (VRAM)</span>


<span style="color:#f8f8f2">LOOP:</span>
     <span style="color:#a6e22e">jp</span> <span style="color:#f8f8f2">LOOP</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Les données</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">donnee:</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span>  <span style="color:#75715e">; sprite 1 pour la TGS</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span>  <span style="color:#75715e">; sprite 1</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">60</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">126</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">153</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">255</span> <span style="color:#75715e">; sprite 2</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">102</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">60</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">66</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">36</span>   <span style="color:#75715e">; sprite 2</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span>    <span style="color:#75715e">; sprite 3</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">24</span>    <span style="color:#75715e">; sprite 3</span>
<span style="color:#f8f8f2">donne1:</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">170</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">100</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">15</span>   <span style="color:#75715e">; Attributs du sprite 0 (position x, y,n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">12</span>       <span style="color:#75715e">; Attributs du sprite 1 (position x, y,n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">200</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">2</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span>      <span style="color:#75715e">; Attributs du sprite 2 (position x, y,n°sprite,couleur)</span>



<span style="color:#75715e">;**************************************************************************************************</span>
<span style="color:#75715e">; Standard Libraries</span>
<span style="color:#75715e">;**************************************************************************************************</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Lib.asm"</span> <span style="color:#75715e">; intègre la librairie MSX</span>

<span style="color:#f8f8f2">END:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#66d9ef">$</span>
<span style="color:#f8f8f2">indcol:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#ae81ff">41000</span>       <span style="color:#75715e">; défini l'adresse de la variable indcol</span>
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
