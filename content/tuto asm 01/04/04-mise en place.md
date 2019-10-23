---
title: "04-Mise en place"
weight: 4
---

# des outils de développement

On va avoir besoin d'installer 3 programmes :

- Un émulateur (simule un MSX)  
- Un éditeur de texte (pour taper du code texte assembleur)  
- un assembleur (compile le code texte en code machine utilisable par un MSX)


Pour cela j'ai suivi les explications du site d'un développeur anglais dont je vous donne l'adresse de sa chaine Youtube :  
[Lets Make a Retro Game](https://www.youtube.com/playlist?list=PLONcFUnqCtricqf3iXpxIleJC_mw1t23m)

---

#### EMULATEUR

L'émulateur MSX s'appelle *blueMSX*, je vous invite à le télécharger à l'adresse suivante dans la section téléchargement :  [http://www.bluemsx.com](http://www.google.com/url?q=http%3A%2F%2Fwww.bluemsx.com%2F&sa=D&sntz=1&usg=AFQjCNG2n4MaGPUiuuND_BjcMgS_FM5mQg)   
Vous l'installez où vous voulez, il n'y a rien de particulier.  
Quand vous le lancez pour la première fois, je vous invite à aller dans le le menu, choisir *Outils*, ensuite *Editeur de machine* et dans le champ *Configuration* vous sélectionnez *MSX-French* et vous cliquez sur le bouton [Lancer].  
Vous avez un MSX francais qui démarre,vous validez par [Entrée] quand il vous demande la date.

---

#### EDITEUR DE TEXTE
On va installer un éditeur de texte qui s'appelle **CONTEXT Editor**, il est simple d'utilisation et permettra de lancer la compilation avec des touches de raccourci.  
Le liens de téléchargement est le suivant :   
[CONTEXT Editor](http://www.google.com/url?q=http%3A%2F%2Fwww.contexteditor.org%2F&sa=D&sntz=1&usg=AFQjCNGgLv5_8lcGyj07z4qYWWiJwXSa-w)  
Installez en suivant la procédure proposée par défaut.

Le développeur du site que j'ai indiqué nous met à disposition un petit plugin qui permet de mettre en couleur le texte assembleur, il se trouve ici : [coloration de syntaxe](http://www.google.com/url?q=http%3A%2F%2Fwww.electricadventures.net%2FContent%2FLMARG%2FEP3%2FWLADXZ80Assembler.zip&sa=D&sntz=1&usg=AFQjCNHwvRdn18DniOvbiXhpxZ_IQMXw1A)

Pour ajouter le colorateur syntaxique à CONTEXT il faut coller le fichier `WLA DX Z80 Assembler.chl` dans le dossier suivant :  
`C:\Program Files (x86)\ConTEXT\Highlighters`
Le dossier sera le suivant si vous êtes en mode Windows 32-bit :  
`C:\Program Files\ConTEXT\Highlighters`

---
#### ASSEMBLEUR
Pour l'assembleur, c'est **tniASM** (v0.45).  
Il est simple et pratique lui aussi.  
Voici l'adresse : [tniasm 0.45](http://www.google.com/url?q=http%3A%2F%2Fwww.tni.nl%2Fproducts%2Ftniasm.html&sa=D&sntz=1&usg=AFQjCNGwmumSQvXBQccdxanfXY0c4oAL0A)  
Vous téléchargez la version gratuite et vous la dézippez dans le répertoire suivant sur votre PC :  
`C:\tniasm045`  
Vous devez ainsi avoir les 2 fichiers suivant dans le dossier 'tniasm045' :

- tniasm.exe  
- tniasm.txt 

Le fichier *tniasm.txt* est la documentation de l'outil, mais en anglais.
Vu que je me suis 'cogné' la traduction en français, autant qu'elle serve : doc française de tniasm à télécharger en bas de cette page.

---

#### CONTEXT-TOUCHE RACCOURCI
Afin de rendre CONTEXT plus facile d'utilisation, on peut paramétrer des touches de raccourci pour compiler le code automatiquement, et faire apparaître les lignes d'erreur.  
Dans le menu de CONTEXT choisissez '`Options / Environment Options / Execute Keys` vous devez avoir cet écran :  
![Execution](/tuto asm 01/04/images/exec.png?classes=shadow)  
Cliquez sur le bouton [Add] et remplissez avec **asm,z80**
![Execution](/tuto asm 01/04/images/e.png?classes=shadow)  
Ainsi, on va définir une action pour tous les fichiers qui ont une extension de type .asm ou .z80.   
Cliquez sur [OK]. 
Vous devriez obtenir cela dans la petite fenêtre de gauche :  
![Execution](/tuto asm 01/04/images/f.png?classes=shadow)  
Cliquez sur la touche de fonction **F9** dans cette petite fenêtre et ensuite remplissez les champs à droite comme suit : 


| champs | Valeur à taper |
| ------ | -------------- |
| Execute | c:\tniasm045\tniasm.exe |
| Start in| %p |
| Parameters | %n |
| Window| Normal |
| Hint | Compile |
| Save | Current file before execution |
| Use short Dos names | Not ticked |
| Capture console output | Ticked |
| Compiler output parser rule | \*line%l(%n)* |
| Scroll console to the last line | Ticked |

Vous obtenez ceci :  
![Execution](/tuto asm 01/04/images/g.png?classes=shadow)  
Quand on voudra compiler du code assembleur, on aura plus qu'à appuyer sur **F9** et cela se fera automatiquement, ou s'il y a une erreur de compilation on le verra dans la fenêtre basse de CONTEXT.

Pour tester cet environnement on utilisera des fichiers source, c'est l'objet du prochain article.

{{% notice note %}}
 Je tiens à remercier celui qui a fait cette procédure d'installation, que je n'ai fait que traduire et adapter à notre exemple, visitez son site :  [http://www.electricadventures.net/](http://www.electricadventures.net/)
{{% /notice %}}

{{%attachments style="blue" title="Fichiers associés" pattern=".*(txt)"/%}}  
