---
title : "11-Gestion du tir"
weight: 11
---


Il ne nous reste plus que le tir à implémenter et notre exemple de shoot 'em up sera terminé.

Le principe est simple, une fois que la touche **ESPACE** sera appuyée, on va déclencher le tir et indiquer qu'il faut déplacer le laser en activant une variable `indbal` (**indicateur de balle tirée**), pendant ce déplacement du laser on va surveiller 2 choses :

1.    la collision entre le SPRITE de l'envahisseur et le SPRITE du laser.
2.    le laser qui qui finit sa course sans atteindre l'envahisseur.

{{% notice note %}}
Dans le cas 1, si le cas arrive c'est le déclenchement de l'explosion et le jeu recommence.  
Dans le cas 2, si cela arrive on remettra on fera disparaitre le laser et on remettra indbal en l'état intial pour permettre à nouveau de tirer.
{{% /notice %}}

Voici ce que donne l'algorithme que l'on codera ensuite :


![orga](/tuto asm 01/11/images/1.png?classes=shadow)

Comme on a initialisé le jeu avec toutes les variables à 1, si la touche espace n'est pas appuyée dans la boucle principale du programme, elle ignorera les 2 autres conditions (indbal et indcol égales à zéro).

---

#### FEU
Si on appuie sur la touche espace, on déclenche le tir au niveau des coordonnées de la fusée et on change la variable indbal qui servira à :

 - déplacer le laser (routine `TIRBAL`).  
 - et empêcher un autre tir par la touche espace tant que le premier n'est pas finit (routine `FEU`).


La routine `FEU:`
<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; FEU</span>
<span style="color:#75715e">; indbal=1 pas de balle tirée / indbal=0 balle tirée</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">FEU:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[indbal]</span>   <span style="color:#75715e">; on vérifie la valeur de indbal</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; si indbal=0 une balle est déjà en cours</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; donc on ne peut pas retirer</span>
<span style="color:#a6e22e">ret</span>    <span style="color:#f8f8f2">z</span>            <span style="color:#75715e">; et on quitte la routine</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6913</span>      <span style="color:#75715e">; si indbal=1 on récupère la coord X de la fusée</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; via la TAS (lecture)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6921</span>      <span style="color:#75715e">; et on la met sa valeur dans le X du laser</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; via la TAS (écriture)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6912</span>      <span style="color:#75715e">; on récupère la coord Y de la fusée</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; via la TAS (lecture)</span>
<span style="color:#a6e22e">sub</span>    <span style="color:#ae81ff">17</span>           <span style="color:#75715e">; enlève 17 à Y pour que le laser=tête de la fusée</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6920</span>      <span style="color:#75715e">; et on la met sa valeur dans le Y du laser</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; via la TAS (écriture)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">0</span>          <span style="color:#75715e">; on passe indbal à 0</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">[indbal],a</span>   <span style="color:#75715e">; pour indiquer qu'un tir est en cours</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; retour à la boucle principale</span>
</pre>

---

#### TIRBAL
**Comme on vient de tirer : indbal=0 maintenant**  
Et la boucle principale appelle la routine de déplacement de la balle (ou du laser si vous préférez).  
Cette routine ne fait que :

- faire appel à une autre routine pour le déplacement (`DEPBAL:`)  
- activer le registre de collision du MSX

Elle appelle donc 3 fois `DEPBAL` sinon le déplacement d'un pixel serait trop lent à l'affichage.  
Puis elle active un registre existant du MSX (en VRAM) qui surveille si des SPRITES entrent en collision. Ce registre est consulté via un `CALL 318`.  
Si parmi les 8 bits qui composent ce registre, le bit 5 est à 1 alors c'est qu'il y a collision.  
Du coup, on met à jour une variable à nous que l'on a appelé `indcol` et qui nous permettra de déclencher l'explosion dans la boucle principale.

Voici le code :
<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; TIRBAL</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">TIRBAL:</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPBAL</span>       <span style="color:#75715e">; appel routine pour déplacer le sprite laser vers le haut</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPBAL</span>       <span style="color:#75715e">; opération reproduite 3 fois</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPBAL</span>       <span style="color:#75715e">; pour que le laser se déplace très vite</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">318</span>          <span style="color:#75715e">; rappel du registre des collisions de sprites</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">5</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; on teste le bit 5</span>
<span style="color:#a6e22e">ret</span>    <span style="color:#f8f8f2">z</span>            <span style="color:#75715e">; si le bit 5=0 alors pas de collision et on quitte</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">0</span>          <span style="color:#75715e">; sinon bit 5=1 collision alors on passe indcol à 0</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">[indcol],a</span>   <span style="color:#75715e">; ce qui déclenchera une explosion via le boucle principale</span>
<span style="color:#a6e22e">ret</span>
</pre>

---

#### DEPBAL
Cette routine ne fait que déplacer le sprite laser d'un pixel vers le haut et vérifie si le laser est arrivé à lextrémité haute de l'écran (Y=0).

Si c'est le cas, il faut que le laser soit réinitialisé, donc caché de l'écran et à disposition pour un nouveau tir.  
Pour cela on change les coordonnées du sprite laser et on repasse `indbal` à 1 via une sous-routine `HAUT:`

Voici le code :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; DEPBAL</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">DEPBAL:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6920</span>      <span style="color:#75715e">; on récupère le Y du laser</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; via la TAS (lecture)</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on décrémente Y pour que le laser monte</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,HAUT</span>       <span style="color:#75715e">; si Y=0 alors appel routine HAUT:</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; sinon on charge la nouvelle valeur dans la TAS</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#f8f8f2">HAUT:</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">200</span>          <span style="color:#75715e">; on met la valeur 200 dans le Y du laser</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6920</span>        <span style="color:#75715e">; le sprite sera en attente en dehors de l'écran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">77</span>             <span style="color:#75715e">; via la TAS (écriture)</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; et on passe indbal à 1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[indbal],a</span>     <span style="color:#75715e">; ainsi on autorise on nouveau déclenchement de laser</span>
<span style="color:#a6e22e">ret</span>
</pre>

---

#### EXPLO
La boucle principale teste à chaque fois **si indcol=0** (*une collision a eu lieu*).
Elle appelle alors la routine `EXPLO:`i qui va produire une explosion.

Le mode écran est tout d'abord réinitialisé en mode 3 (multicouleur), les tables de la VRAM sont remplies avec des valeurs aléatoires qui affichera des carrés de toutes les couleurs, ce processus est appelé plusieurs fois et la routine finie par appeller la routine `DEBUT:` qui fait recommencer le jeu avec des variables remises à 1.

et voici :
<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; EXPLO</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">EXPLO:</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">3</span>           <span style="color:#75715e">; 3 pour screen 3</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],a</span>     <span style="color:#75715e">; (bios)SCRMOD adresse hexa $FCAF :mode courant de l'écran</span>
<span style="color:#a6e22e">call</span>  <span style="color:#ae81ff">95</span>            <span style="color:#75715e">; (bios) CHGMOD change le mode de l'écran</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">100</span>         <span style="color:#75715e">; valeurs de départ</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">0</span>

<span style="color:#f8f8f2">EXPLO1:</span>
<span style="color:#a6e22e">PUSH</span>   <span style="color:#f8f8f2">hl</span>
<span style="color:#a6e22e">PUSH</span>   <span style="color:#f8f8f2">af</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">2048</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">1000</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">92</span>
<span style="color:#a6e22e">pop</span>    <span style="color:#f8f8f2">af</span>
<span style="color:#a6e22e">pop</span>    <span style="color:#f8f8f2">hl</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">h</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">nz,EXPLO1</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">DEBUT</span>
</pre>

Il ne nous reste plus qu'à compléter la boucle principale pour tester la touche espace (lignes 27 à 30 du code ci-dessous) et tester les variables (lignes 10 à 17 du code ci-dessous).

#### BOUCLE PRINCIPALE

</pre>
</td><td><pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Boucle du jeu : Le programme démarre à DEBUT puis va à BOUCLE</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>

<span style="color:#f8f8f2">DEBUT:</span>              <span style="color:#75715e">; c'est ici que débute le programme !!!</span>
<span style="color:#a6e22e">call</span>  <span style="color:#f8f8f2">INIT</span>          <span style="color:#75715e">; on appelle le sous-prog INIT</span>
<span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; c'est la boucle principale du programme !!!</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPENV</span>       <span style="color:#75715e">; on appelle le déplacement de l'envahisseur</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on appelle le sous-prog DELAI afin de ralentir l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[indcol]</span>   <span style="color:#75715e">; on charge la valeur de indcol dans a</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on fait un calcul sur a</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,EXPLO</span>      <span style="color:#75715e">; si a=0 alors on appelle EXPLO</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[indbal]</span>   <span style="color:#75715e">; on charge la valeur de indbal dans a</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on fait un calcul sur a</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,TIRBAL</span>     <span style="color:#75715e">; si a=0 alors on appelle TIRBAL</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche gauche = ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; (bios) SNSMAT retourne dans la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">4</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; bit teste la colonne 4 (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSGAU</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSGAU (déplacer à droite)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; la touche droite est aussi sur la ligne 8 de la matrice</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; on appelle le bios pour qu'il donne la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">7</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; la colonne 7 est testée (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSDRO</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSDRO (déplacer àgauche)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche espace=ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; on appelle le bios pour qu'il donne la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; la colonne 0 est testée (0=pressée)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FEU</span>        <span style="color:#75715e">; si résultat=0 on va au sous-prog FEU (déclenchement laser)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">BOUCLE</span>       <span style="color:#75715e">; on redémarre la boucle principale</span>
</pre>

Comme d'habitude je vous donne le code final mis à jour à compiler et à vérifier avec votre émulateur :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"test4.ROM"</span>
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
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,ENGAU</span>      <span style="color:#75715e">; si a=0 c'est que indir valait 1,on va au sous-prog ENGAU (à gauche)</span>
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
<span style="color:#75715e">; FEU</span>
<span style="color:#75715e">; indbal=1 pas de balle tirée / indbal=0 balle tirée</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">FEU:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[indbal]</span>   <span style="color:#75715e">; on vérifie la valeur de indbal</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; si indbal=0 une balle est déjà en cours</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; donc on ne peut pas retirer</span>
<span style="color:#a6e22e">ret</span>    <span style="color:#f8f8f2">z</span>            <span style="color:#75715e">; et on quitte la routine</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6913</span>      <span style="color:#75715e">; si indbal=1 on récupère la coord X de la fusée</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; via la TAS (lecture)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6921</span>      <span style="color:#75715e">; et on la met sa valeur dans le X du laser</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; via la TAS (écriture)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6912</span>      <span style="color:#75715e">; on récupère la coord Y de la fusée</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; via la TAS (lecture)</span>
<span style="color:#a6e22e">sub</span>    <span style="color:#ae81ff">17</span>           <span style="color:#75715e">; enlève 17 à Y pour que le laser=tête de la fusée</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6920</span>      <span style="color:#75715e">; et on la met sa valeur dans le Y du laser</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; via la TAS (écriture)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">0</span>          <span style="color:#75715e">; on passe indbal à 0</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">[indbal],a</span>   <span style="color:#75715e">; pour indiquer qu'un tir est en cours</span>
<span style="color:#a6e22e">ret</span>                 <span style="color:#75715e">; retour à la boucle principale</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; DEPBAL</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">DEPBAL:</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6920</span>      <span style="color:#75715e">; on récupère le Y du laser</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">74</span>           <span style="color:#75715e">; via la TAS (lecture)</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on décrémente Y pour que le laser monte</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,HAUT</span>       <span style="color:#75715e">; si Y=0 alors appel routine HAUT:</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">77</span>           <span style="color:#75715e">; sinon on charge la nouvelle valeur dans la TAS</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#f8f8f2">HAUT:</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">200</span>          <span style="color:#75715e">; on met la valeur 200 dans le Y du laser</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">6920</span>        <span style="color:#75715e">; le sprite sera en attente en dehors de l'écran</span>
<span style="color:#a6e22e">call</span> <span style="color:#ae81ff">77</span>             <span style="color:#75715e">; via la TAS (écriture)</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">1</span>            <span style="color:#75715e">; et on passe indbal à 1</span>
<span style="color:#a6e22e">ld</span>   <span style="color:#f8f8f2">[indbal],a</span>     <span style="color:#75715e">; ainsi on autorise on nouveau déclenchement de laser</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; TIRBAL</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">TIRBAL:</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPBAL</span>       <span style="color:#75715e">; appel routine pour déplacer le sprite laser vers le haut</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPBAL</span>       <span style="color:#75715e">; opération reproduite 3 fois</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPBAL</span>       <span style="color:#75715e">; pour que le laser se déplace très vite</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">318</span>          <span style="color:#75715e">; rappel du registre des collisions de sprites</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">5</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; on teste le bit 5</span>
<span style="color:#a6e22e">ret</span>    <span style="color:#f8f8f2">z</span>            <span style="color:#75715e">; si le bit 5=0 alors pas de collision et on quitte</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">0</span>          <span style="color:#75715e">; sinon bit 5=1 collision alors on passe indcol à 0</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">[indcol],a</span>   <span style="color:#75715e">; ce qui déclenchera une explosion via le boucle principale</span>
<span style="color:#a6e22e">ret</span>

<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; EXPLO</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">EXPLO:</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">3</span>           <span style="color:#75715e">; 3 pour screen 3</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">[</span><span style="color:#ae81ff">64687</span><span style="color:#f8f8f2">],a</span>     <span style="color:#75715e">; (bios)SCRMOD adresse hexa $FCAF :mode courant de l'écran</span>
<span style="color:#a6e22e">call</span>  <span style="color:#ae81ff">95</span>            <span style="color:#75715e">; (bios) CHGMOD change le mode de l'écran</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">100</span>         <span style="color:#75715e">; valeurs de départ</span>
<span style="color:#a6e22e">ld</span>    <span style="color:#f8f8f2">hl,</span><span style="color:#ae81ff">0</span>

<span style="color:#f8f8f2">EXPLO1:</span>
<span style="color:#a6e22e">PUSH</span>   <span style="color:#f8f8f2">hl</span>
<span style="color:#a6e22e">PUSH</span>   <span style="color:#f8f8f2">af</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">de,</span><span style="color:#ae81ff">2048</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">bc,</span><span style="color:#ae81ff">1000</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">92</span>
<span style="color:#a6e22e">pop</span>    <span style="color:#f8f8f2">af</span>
<span style="color:#a6e22e">pop</span>    <span style="color:#f8f8f2">hl</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">h</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">nz,EXPLO1</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">DEBUT</span>



<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Boucle du jeu : Le programme démarre à DEBUT puis va à BOUCLE</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>

<span style="color:#f8f8f2">DEBUT:</span>              <span style="color:#75715e">; c'est ici que débute le programme !!!</span>
<span style="color:#a6e22e">call</span>  <span style="color:#f8f8f2">INIT</span>          <span style="color:#75715e">; on appelle le sous-prog INIT</span>
<span style="color:#f8f8f2">BOUCLE:</span>             <span style="color:#75715e">; c'est la boucle principale du programme !!!</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DEPENV</span>       <span style="color:#75715e">; on appelle le déplacement de l'envahisseur</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on appelle le sous-prog DELAI afin de ralentir l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[indcol]</span>   <span style="color:#75715e">; on charge la valeur de indcol dans a</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on fait un calcul sur a</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>
<span style="color:#a6e22e">jp</span>     <span style="color:#f8f8f2">z,EXPLO</span>      <span style="color:#75715e">; si a=0 alors on appelle EXPLO</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,[indbal]</span>   <span style="color:#75715e">; on charge la valeur de indbal dans a</span>
<span style="color:#a6e22e">inc</span>    <span style="color:#f8f8f2">a</span>            <span style="color:#75715e">; on fait un calcul sur a</span>
<span style="color:#a6e22e">dec</span>    <span style="color:#f8f8f2">a</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,TIRBAL</span>     <span style="color:#75715e">; si a=0 alors on appelle TIRBAL</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche gauche = ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; (bios) SNSMAT retourne dans la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">4</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; bit teste la colonne 4 (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSGAU</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSGAU (déplacer à droite)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">DELAI</span>        <span style="color:#75715e">; on ralenti l'exécution</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; la touche droite est aussi sur la ligne 8 de la matrice</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; on appelle le bios pour qu'il donne la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">7</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; la colonne 7 est testée (0=pressé 1=non pressé)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FUSDRO</span>     <span style="color:#75715e">; si résultat=0 on va au sous-prog FUSDRO (déplacer àgauche)</span>
<span style="color:#a6e22e">ld</span>     <span style="color:#f8f8f2">a,</span><span style="color:#ae81ff">8</span>          <span style="color:#75715e">; on teste la touche espace=ligne 8 de la matrice du clavier</span>
<span style="color:#a6e22e">call</span>   <span style="color:#ae81ff">321</span>          <span style="color:#75715e">; on appelle le bios pour qu'il donne la valeur de la ligne 8</span>
<span style="color:#a6e22e">bit</span>    <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,a</span>          <span style="color:#75715e">; la colonne 0 est testée (0=pressée)</span>
<span style="color:#a6e22e">call</span>   <span style="color:#f8f8f2">z,FEU</span>        <span style="color:#75715e">; si résultat=0 on va au sous-prog FEU (déclenchement laser)</span>
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
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">170</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">100</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">15</span>   <span style="color:#75715e">; Attributs du sprite 0 (position x, y,n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">12</span>       <span style="color:#75715e">; Attributs du sprite 1 (position x, y,n°sprite,couleur)</span>
<span style="color:#66d9ef">db</span>     <span style="color:#ae81ff">200</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">0</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">2</span><span style="color:#f8f8f2">,</span><span style="color:#ae81ff">1</span>      <span style="color:#75715e">; Attributs du sprite 2 (position x, y,n°sprite,couleur)</span>



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
