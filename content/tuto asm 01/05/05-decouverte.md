---
title: "05-Découverte du code"
weight: 5
---

## Fichiers sources et découverte du code assembleur MSX

Pour faire fonctionner l'environnement que l'on a mis en place, on a besoin d'un document où taper notre code.  
Il se présente sous la forme d'un petit fichier texte comme on peu en utiliser avec le *blocnote* de Windows. Sauf que le notre aura une extension de fichier qui se termine par **.asm** au lieu du traditionnel .txt

C'est lui que l'on va ouvrir avec l'éditeur de texte CONTEXT, c'est lui qui va recevoir le code que l'on va taper et c'est encore lui que l'on va compiler via la touche de fonction **F9**. Cette compilation nous donnera un fichier exploitable par l'émulateur MSX.

Ce fichier ne peut pas être compilé tout seul, il a besoin de se trouver dans le même dossier ou vont figurer 2 fichiers sources (appelées **librairies**) et qui aideront à la compilation.  
En effet notre fichier principal comporte un entête et un pied de page que l'on ne touchera pas mais qui à l'aide des librairies feront qu'à la compilation on se retrouvera avec un fichier de sortie qui aura l'extension **.ROM**.  
Et miracle, cela simulera les fameuses cartouches ROM de notre enfance qui démarraient automatiquement quand on allumait notre bon vieux MSX.

Ce fichier ne peut pas être compilé tout seul, il a besoin de se trouver dans le même dossier ou vont figurer 2 fichiers sources (appelées librairies) et qui aideront à la compilation.
En effet notre fichier principal comporte un entête et un pied de page que l'on ne touchera pas mais qui à l'aide des librairies feront qu'à la compilation on se retrouvera avec un fichier de sortie qui aura l'extension.ROM.
Et miracle, cela simulera les fameuses cartouches ROM de notre enfance qui démarraient automatiquement quand on allumait notre bon vieux MSX.

Bon on résume avec un schéma :  
![Execution](/tuto asm 01/05/images/1.png?classes=shadow)

Bon, j'espère que c'est plus clair maintenant.  
Avant que je ne vous donne le liens de téléchargement des fichiers sources, je vais vous les présenter:

Le fichier principal, celui où on tapera notre code (pour l'instant, il n'y a que l'entête et le pied de page à l'intérieur) :

````
example.asm
````
Les 2 librairies, qui doivent se trouver toujours dans le même dossier que le fichier principal sinon l'assembleur ne saura pas où les trouver :

````
MSXRom-Include.asm

MSXRom-Lib.asm 
````
Pour le lien du dossier : fichiers source à télécharger en bas de cette page.
{{% notice note %}}
 Je me dois encore remercier celui qui a fourni ces sources, c'est son travail :  [http://www.electricadventures.net/](http://www.electricadventures.net/)
{{% /notice %}}

Vous ouvrez `exemple.asm` avec **CONTEXT** et on va en profiter pour utiliser le colorateur syntaxique qui reconnait le langage assembleur et que l'on a téléchargé précédemment.  
Donc, dans le menu de CONTEXT, choisissez `Tools / Set Highlighter` et dansla liste des codes connus, vous devez trouver `WLA-Z80 Assembler`, sélectionnez-le.

Si vous voulez faire apparaître les numéros de ligne sous CONTEXT (utile pour le debuggage), dans le menu de CONTEXT, choisissez `Options / Environment Options / onglet Editor` et vous cochez `Line numbers`.  
Vous devez en être là :

![Execution](/tuto asm 01/05/images/2.png?classes=shadow)  

2 fenêtres:  

- La partie haute : zone où on tape le code. 
- La partie basse : zone de suivi de la compilation quand on fait **F9**.

Regardons ce code depuis le début :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">; ****************************************************************</span>
<span style="color:#75715e">; MSX ROM Cartridge Header and Function library</span>
<span style="color:#75715e">; ****************************************************************</span>
<span style="color:#a6e22e">FNAME</span> <span style="color:#e6db74">"exemple.ROM"</span>
<span style="color:#66d9ef">cpu</span> <span style="color:#f8f8f2">z80</span>

<span style="color:#66d9ef">ORG</span> <span style="color:#ae81ff">4000h</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Include.asm"</span>

<span style="color:#66d9ef">db</span> <span style="color:#e6db74">"AB"</span>
<span style="color:#66d9ef">DW</span> <span style="color:#f8f8f2">INIT</span>            <span style="color:#75715e">; la ROM lance le sous-programme INIT au démarrage</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#66d9ef">DW</span> <span style="color:#ae81ff">0</span>
<span style="color:#a6e22e">DS</span> <span style="color:#ae81ff">6</span>
</pre>

C'est ce que j'appelle l'entête.

**FNAME**: c'est le nom du fichier qui sortira à la compilation, vous pouvez donc mettre le nom que vous voulez entre les " " du moment que vous conservez l'extension.ROM.

**cpu Z80**: La compilation utilisera la librairie Z80 (tniASM en gère plusieurs).

**ORG 4000h**: Notre programme s'implantera à cette adresse au démarrage de la ROM dans l'émulateur. Pour info, quand un nombre est suivi de 'h' c'est que l'on indique que c'est un Hexadécimal sinon, c'est un décimal.

**INCLUDE**: C'est pour cela que je vous disais que les librairies doivent être dans le même dossier, car à la compilation, tniASM va chercher tous les fichiers cité par in INCLUDE et qui va lui permettre d'interpréter le code spécifique à l'assembleur MSX.

**DW INIT**: Cette suite de DW permet de créer une structure de ROM à la compilation d'après ce que j'en ai compris. Ce qui est important, c'est le mot INIT, cela aurait pu être n'importe quoi FRAISE, MARTEAU,etc...  
Cela intime l'ordre au démarrage de la ROM d'aller déclencher le code qui commence par ce 'label'. Ces 'labels' sont des bout de programmes appelés routines qui que l'on pourra appeler au besoin. En effet, en assembleur, on ne dispose pas des lignes comme en BASIC alors on appelle des 'labels' pour sauter à une partie du programme.

Le pied de page est présenté ici mais je n'en parlerai pas, il ne vous apprendra rien de plus : En fait, je ne le maîtrise pas, alors je le laisse ;-)  
Pour info, lorsque on tape un point virgule `;` ce qui suis sur la ligne est du commentaire et ne sera pas interprété comme du code par l'assembleur.

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;**************************************************************************************************</span>
<span style="color:#75715e">; Standard Libraries</span>
<span style="color:#75715e">;**************************************************************************************************</span>

<span style="color:#a6e22e">INCLUDE</span> <span style="color:#e6db74">"MSXRom-Lib.asm"</span> <span style="color:#75715e">; intègre la librairie MSX</span>

<span style="color:#f8f8f2">END:</span> <span style="color:#a6e22e">EQU</span> <span style="color:#66d9ef">$</span>


<span style="color:#75715e">;**************************************************************************************************</span>
<span style="color:#75715e">; RAM Definitions</span>
<span style="color:#75715e">;**************************************************************************************************</span>

<span style="color:#75715e">;-------------------</span>
<span style="color:#75715e">; Définit que le programme démarrera au début de la RAM disponible</span>
<span style="color:#75715e">;-------------------</span>
<span style="color:#66d9ef">ORG</span> <span style="color:#f8f8f2">RAMSTART</span>
</pre>

Ce qui nous intéresse se trouve entre ces 2 parties :

<pre style="line-height:125%;margin:0"><span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#75715e">; Démarrage du programme à INIT</span>
<span style="color:#75715e">;------------------------------------------------------------------------------</span>
<span style="color:#f8f8f2">INIT:</span>
<span style="color:#75715e">; Tapez votre code à partir d'ici</span>
</pre>

C'est ici que nous allons taper notre 1er code....dans le prochain article.

{{%attachments style="blue" title="Fichiers associés" pattern=".*(zip)"/%}}  
