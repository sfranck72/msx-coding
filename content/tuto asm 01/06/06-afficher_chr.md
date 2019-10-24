---
title: "06-La pratique"
weight: 6
---

## La pratique : Afficher un caractère

On va maintenant taper notre 1er code qui permettra :

>- **Activer le mode SCREEN 1.**
- **Afficher le caractère A à la position 5 de l'écran.**
- **Changer la couleur du caractère et du fond du caractère.**
- **Faire une boucle infinie pour que le résultat de cet affichage reste à l'écran.**

---

Voici ce que cela donne pour l'ensemble du code, mais on va le reprendre bout par bout :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Démarrage du programme à INIT</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">INIT:</span>
<span style="color:#a6e22e">Ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; 1 pour 'SCREEN 1'</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],a</span>      <span style="color:#75715e">; routine BIOS [SCRMOD] : mode courant de l'écran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">95</span>             <span style="color:#75715e">; routine BIOS [CHMOD]  : change le mode l'écran</span>

<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">65</span>           <span style="color:#75715e">; on charge l'accumulateur avec 65 (ASCII de 'A')</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6144</span><span style="color:#f92672">+</span><span style="color:#ae81ff">5</span>      <span style="color:#75715e">; on charge en HL l'adresse de la TNP + 5 (position 5)</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">77</span>             <span style="color:#75715e">; routine BIOS [WRTVRM] : écrit dans la VRAM (write to VRAM)</span>

<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">223</span>          <span style="color:#75715e">; on charge le code couleur (magenta sur blanc=223)</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">8200</span>        <span style="color:#75715e">; on charge l'adresse de la TC pour le caractère 'A'</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">77</span>             <span style="color:#75715e">; routine BIOS [WRTVRM] : écrit dans la VRAM (write to VRAM)</span>

<span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; A la fin d'INIT, le programme n'est pas stoppé alors il continue</span>
<span style="color:#a6e22e">JP</span>     <span style="color:#f8f8f2">BOUCLE</span>       <span style="color:#75715e">; dans le prochain label, ici BOUCLE (comparable à 100 GOTO 100)</span>
</pre>

C'est parti :  
En mode assembleur, en général, **on charge un accumulateur A avec une valeur entre 0 et 255**, et une adresse en HL (ce sont les registres internes avec lesquels la RAM fait les calculs) puis on lance l'odre d'exécution par un CALL.

#### SCREEN 1

C'est l'exemple type, l'adresse 64687 est une routine BIOS qui abrite le mode courant de l'écran (SCREEN 1, 2 ou 3).

1. Donc la 1ere ligne charge (c'est l'instruction 'LD') A avec 1 donc A=1.
2. On assigne cette valeur à l'adresse 64687 (2eme ligne).
3. Et enfin on donne l'ordre via une routine spécifique du BIOS en ROM d'exécuter cela. Après le CALL le SCREEN 1  est effectif.

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Démarrage du programme à INIT</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">INIT:</span>
<span style="color:#a6e22e">Ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; 1 pour 'SCREEN 1'</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],a</span>      <span style="color:#75715e">; routine BIOS [SCRMOD] : mode courant de l'écran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">95</span>             <span style="color:#75715e">; routine BIOS [CHMOD]  : change le mode l'écran</span>
</pre>

Au départ ces adresses et routines appelées semblent obscures mais celles que l'on utilise régulièrement sont en nombre réduit et deviendront familières rapidement.
J'utilise  souvent le décimal, mais `CALL 95` peut s'écrire aussi  `CALL $5F` en hexadécimal, tniasm reconnait la plupart des expressions pour l'hexadécimal (voir sa doc).  
Un site pour voir la liste des routines BIOS : [http://map.grauw.nl/resources/msxbios.php] (http://map.grauw.nl/resources/msxbios.php)


>**CHGMOD**  
Address  : #005F  
Function : Switches to given screenmode  
Input    : A  - screen mode  
Registers: All

---

#### AFFICHAGE DU 'A' 

Là, ça doit vous parler, on connait la valeur du caractère 'A' en ASCII = 65.  
On a vu dans la théorie que le début de la table TNP (qui gère les positions d'affichage) est à l'adresse 6144 en SCREEN 1. On a choisit arbitrairement  la position 5 (vous pouvez vous amuser avec une autre valeur).  
Petite nouveauté, le **CALL 77** (CALL $4D) qui exécute l'ordre du calcul que l'on vient de faire attend  **A** et **HL** chargés en entrants :

>**WRTVRM**  
Address  : #004D  
Function : Writes data in VRAM  
Input    : HL - address write  
           A  - value write

Voici le bout de code correspondant :  
<pre style="line-height:125%;margin:0"><span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">65</span>           <span style="color:#75715e">; on charge l'accumulateur avec 65 (ASCII de 'A')</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6144</span><span style="color:#f92672">+</span><span style="color:#ae81ff">5</span>      <span style="color:#75715e">; on charge en HL l'adresse de la TNP + 5 (position 5)</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">77</span>             <span style="color:#75715e">; routine BIOS [WRTVRM] : écrit dans la VRAM (write to VRAM)</span>
</pre>
#### CHANGER LA COULEUR DU CARACTERE

Même punition. Tout est dans le commentaire du code :
<pre style="line-height:125%;margin:0"><span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">223</span>          <span style="color:#75715e">; on charge le code couleur (magenta sur blanc=223)</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">8200</span>        <span style="color:#75715e">; on charge l'adresse de la TC pour le caractère 'A'</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">77</span>             <span style="color:#75715e">; routine BIOS [WRTVRM] : écrit dans la VRAM (write to VRAM)</span>
</pre>

#### BOUCLE INFINIE  
On se rajoute cette boucle pour ne pas revenir au BASIC et perdre l'affichage.  
<pre style="line-height:125%;margin:0"><span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; A la fin d'INIT, le programme n'est pas stoppé alors il continue</span>
<span style="color:#a6e22e">JP</span>     <span style="color:#f8f8f2">BOUCLE</span>       <span style="color:#75715e">; dans le prochain label, ici BOUCLE (comparable à 100 GOTO 100)</span>
</pre>

Il n'y a plus qu'a compiler et demander à l'émulateur d'utiliser le fichier généré qui doit être "testA.ROM"....vous vous souvenez, le FNAME dans l'entête, au début du code, ben je l'ai modifié ainsi pour changer le nom de fichier généré.

#### COMPILATION

On clique sur la touche de fonction [F9], cela sauvegarde notre fichier texte et lance la compilation.  
*Vous pouvez aussi procéder autrement: Sauvegardez votre fichier et dans l'explorateur Windows, faites glissez votre "testA.asm" sur "tniasm.exe" avec la souris et cela va déclencher la compilation et créer le fichier en sortie dans le même dossier. Cela sous-entend que tniasm est dans le même dossier que votre code et ses librairies. L'inconvénient c'est que vous ne savez pas s'il y a des erreurs de compilation.* 

On voit que la fenêtre basse de CONTEXT a changée :
![Execution](/tuto asm 01/06/images/1.png?classes=shadow)

On voit les 2 passes pour générer le fichier .ROM et pas d'erreur.  
Si vous avez une erreur, elle se présentera de cette manière :  

![Execution](/tuto asm 01/06/images/2.png?classes=shadow)

CONTEXT indique la ligne d'erreur, ici pour l'exemple c'est la ligne 22.  
Quand on regarde notre code, on voit que l'on a ajouté un **t** par erreur....c'est pas toujours aussi évident, je vous l'accorde.


#### FICHIER .ROM GENERE  
Si tout c'est bien passé, notre répertoire où se trouve notre code doit ressembler à ceci :
![Execution](/tuto asm 01/06/images/3.png?classes=shadow)

- Les 2 librairies  
- Notre code principal : "testA.asm"
Le fichier "testA.ROM" généré, c'est celui-ci qui nous intéresse.  
- 3 fichiers de compilations (en surligné dans l'image ci-dessus) que l'on n'utilisera pas.  

On va prendre le **"testA.rom"** et le faire glisser avec la souris dans l'émulateur blueMSX que vous aurez ouvert au préalable.

BlueMSX va rebooter le MSX et au bout de quelques secondes on doit admirer.....un caractère.....mais en couleur s'il vous plait !!!

![Ecran 1](/tuto asm 01/06/images/4.png?classes=shadow)

On voit dans le bandeau en bas que dans le champs 'SCREEN' nous sommes bien passé en 'SCR 1'


#### EJECTER LA ROM ET QUITTER   
Vous vous rappelez que nous avons mis une boucle infinie à la fin de notre code, de ce fait nous ne pouvons avoir d'action sur l'émulateur et si vous le rebootez en appuyant sur le gros bouton rond en haut à gauche (à côté du point rouge), blueMSX va relancer la machine mais aussi notre fichier .ROM.  

En effet, nous venons de créer une cartouche 16ko et elles sont faites pour démarrer automatiquement.  
Donc si vous voulez **éjecter** la cartouche, vous appuyez sur le bouton rond à côté de **slot** et choisissez **Ejecter: testA.ROM** :

![Ecran 1](/tuto asm 01/06/images/5.png?classes=shadow)
 
blueMSX va rebooter la machine automatiquement mais sans la cartouche et rendre la main à l'invite de commande du BASIC.

Le prochain article on apprendra à afficher les sprites, pour cela on partira directement sur l'élaboration du shoot-em-up.

