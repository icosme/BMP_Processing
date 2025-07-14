# SAE IMAGE

*Etude sur l'encodage des images BMP*

---
# A.0 - Analyse des valeurs de l'en-tête

![](TRACES_SCREENS/capture_valeurs.png)

 Après un moment de documentation grâce aux ressources disponibles dans l'énoncé, on découvre la structure suivante sur les premiers octets :

* **Tout d'abord, le format du fichier** `42 4D` : Ce sont les deux premiers octets. Ils correspond aux caractères " BM " lorsque l'on se réfère à la table ASCII. Le format de l'image est donc `Bitmap Windows`, soit BMP.

* **Puis la taille du fichier** : 
  - La valeur initiale semble être mal encodée : `99 73 0C 00` (qui correspond à une taille fichier de *2 574 453 760* octets, c'est beaucoup trop lourd)
  - Il faut donc corriger dans l'éditeur : On s'assure de la valeur d'octets en vérifiant dans les propriétés de l'image, puis on convertit *816 026* octets en base héxadécimal pour obtenir la bonne suite de nombres. Cela nous donne `9A 73 0C 00`.

* **Les dimensions** de l'image sont trouvables à l'adresse 0x0012 pour la largeur et à l'adresse 0x0016 pour la longueur donc 640×425 pixels (`80 02 A9 01`). Il suffit de convertir en base 10 les octets.

* **On retrouve aussi le nombre de plans** après les octets de dimension de l'image, soit ici `01 00`. **La profondeur de l'image** est de 24 bits par pixel : on convertit (`18 00`) en base 10. **Pour finir, l'encodage du corps de l'image** commence à l'adresse `0x001A`.


>  **À noter que** : Après avoir corriger la taille du fichier, le message d'erreur a disparu.
> 

![](TRACES_SCREENS/mess_erreurs.png)
---
# A.1 - Création de l'Image0.bmp  

![](TRACES_SCREENS/trace2_A1.png)


### **Pour  créer l'image Image0.bmp**, nous allons procéder de la manière suivante : 


On commence par saisir les caractères `BM` en ASCII : soit `42 4D`. 

Pour la taille du fichier, il faut la calculer à la fin : on laisse temporairement `00 00 00 00`. On saisit ensuite des espaces réservés `00 00 00 00` et on signale l’adresse de la zone de définition de l’image soit `1A 00 00 00` *(26 en décimal)*.


* Ensuite, on saisit la taille de la section : `OC 00 00 00` *(12 en décimal)*. Puis vient la saisie des dimensions de l'image 4x4 :
   - On rentre une largeur de `04 00` *(4 pixels)*.
   - Et une hauteur de `04 00`.
   
**Comme l'éditeur est en little endian**,  nous allons saisir les pixels RGB en partant de la dernière ligne en bas, et de gauche à droite :

   - Chaque carré fait un pixel, on alterne entre blanc `FF FF FF` et rouge `00 00 FF` ce qui donne la première ligne de pixels :

>  `FF FF FF 00 00 FF FF FF FF 00 00 FF`
> 
   
   Et ainsi de suite, **on continue à encoder les autres lignes** dans le même ordre. Une fois toutes les lignes écrites, nous comptons la taille totale du fichier et mettons  cette valeur en little endian à l’adresse correspondante : `4A 00 00 00`. 
     
        display -sample 5000% Image0.bmp
     

![](TRACES_SCREENS/trace_A1.png)
 --- 
# A.2 - Modification des couleurs de l'image

![](TRACES_SCREENS/trace2_A2.png)

### **Le procédé pour créer l'image est similaire à Image0.bmp :**


Ici, ce qui nous intéresse, ce sont **les données des pixels.** On les écrit en `RGB`, en commençant par la dernière ligne.

- D'abord le cyan `00 FF FF` qui correspond à *(0, 255, 255)* sur le **guide des couleurs**, puis le magenta `FF 00 FF`   *(255, 0, 255)*, le bleu céruléen `00 7F FF`   *(0, 127, 255)*, et enfin le blanc `FF FF FF`  *(255, 255, 255)*. 

Pour les autres lignes, on  rentre du rouge `FF 00 00` *soit (255, 0, 0)* et du vert `00 FF 00` aux octets appropriés. Il suffit enfin de sauvegarder le fichier et de vérifier le résultat.


![](TRACES_SCREENS/traces_A2.png)

---
# A.3 - Un fichier BMP d'un type plus récent


Lors de la conversion au format **Windows NT (3.1x)**, le **BITMAPCOREHEADER** de `12 octets` est remplacé par un **BITMAPINFOHEADER** de `40 octets`. Si on calcule la différence, la taille du fichier sera augmentée de `28 octets` (40 - 12 = 28).

### *Quelle est le poids alors de notre image?* 

* 74 octets + 28 octets = **`102 octets`.**

### *Combien y a-t-il de bits par pixel ?*

* Il y a **24 bits** par pixel (8 bits pour chaque couleur du `RGB` : rouge, vert, bleu).

### *Quelle est la taille des données pixels ?*

* **La taille des données pixels** est de 4 x 4 x 3 = `48 octets`.

### *Y a-t-il une compression utilisée ?*

* **Non**, il n'y a pas de compression utilisée.

### *Le codage des pixels a-t-il changé ?*

* **Il semble que oui**, car le codage des pixels est maintenant à l'adresse `0x038`. 
---

# A.4 - Un fichier BMP avec index de couleurs


On s'intéresse désormais à `Image2.bmp`. Celle-ci possède **8 bits par pixel** et elle **n’a pas été compressée**, comme l’indique `00 00 00 00` à l’adresse `0x1E`.


- De plus, l'image utilise **2 couleurs différentes**. Le nombre de couleurs est indiqué à l’adresse `0x2E` avec la valeur `02 00 00 00`.

- Les couleurs de la palette sont effectivemment celles de notre damier :

  - Avec le **rouge** : `FF 00 00 00` ,

  - Et le **blanc** : `FF FF FF 00` .


Le codage des pixels a bien changé. Au lieu de coder chaque pixel individuellement, les pixels sont maintenant encodés ensemble sur **32 bits** *(4 octets)*.


### *Changez la couleur rouge des pixels en bleu :*

Pour cela, nous remplacons l’encodage de la couleur rouge actuelle `00 00 FF 00` par l’encodage de la couleur bleue `FF 00 00 00`.

![](TRACES_SCREENS/trace_A4.png)

Et pour inverser le damier, il suffit d’inverser les positions des couleurs dans la palette, soit : `50` par `A0` et ainsi de suite.

### *Création de **Image3.bmp** :*

### Pour obtenir **Image3.bmp**, il faut changer les valeurs d’encodage des pixels :

- La première ligne, initialement remplie de bleu et blanc, aura ses pixels encodés comme `00` pour les blancs, et `FF 00 00 00` pour les rouges.

- Ensuite, pour la dernière ligne, les pixels seront encodés avec `50 00 00 00` (pour les couleurs restantes).

![](TRACES_SCREENS/traces1_A4.png)

---

### A présent, on passe l'ancien logo du département informatique en **mode index de couleurs**, ce qui nous donne:

![](TRACES_SCREENS/trace2_A4.png)


### *Adresse et palette :*
- Le nombre de couleurs est trouvé à l’adresse `0x2E`. La couleur la plus proche du blanc se trouve à l’adresse `0x66` avec l’encodage `00 00 00 00`, qui est en réalité du blanc.

- Quant au tableau de pixels: **il commence à l’adresse `0x76`.**


### Pour ajouter des pixels bleus en bas de l’image, il faut modifier les premiers pixels de la dernière ligne. 

![](TRACES_SCREENS/trace3_A4.png)

- Lorsque le nombre de couleurs dans la palette est réduit, les couleurs deviennent **sombres**. Visuellement, cela entraîne une **diminution de la qualité des couleurs**.

- En hexadécimal, cela se reflète par l'encodage des pixels qui seront désormais indexés dans **une petite palette**.

![](TRACES_SCREENS/trace4_A4.png)
---


# A.5 - Utilisation des négatifs


### Pour  **inverser l'image**, on modifie la hauteur dans l'en-tête en remplaçant sa valeur positive par sa valeur négative. 

- Nous savons que la hauteur de l'image est de **4 pixels**, encodée par `04 00 00 00`. Donc pour représenter une valeure négative, nous calculons la valeur négative en base 2. Sur `32 bits`, **-4** est représenté comme :  

>  1111 1111 1111 1111 1111 1111 1111 1100
>

**En hexadécimal, cela donne : `FF FF FF FC`.** 
Comme on est en little-endian, on n'oublie pas d'inverser les octets.

![](TRACES_SCREENS/trace_A5.png)


### On applique la méthode à **ImageExempleIndexBMP3_16.bmp** :
Pour obtenir une version inversée de **ImageExempleIndexBMP3_16.bmp**, on suit la même méthode en modifiant la hauteur dans l'en-tête.

* La hauteur est de **425 pixels**, encodée en `A9 01 00 00` *(425 en décimal)*. Donc pour obtenir la hauteur **-425**, calculons sa représentation binaire :  

>  1111 1111 1111 1111 1111 1110 0101 0111
>

**En hexadécimal, cela donne : `FF FF FE 57`.** Et comme toujours, on n'oublie pas d'inverser les octets.

![](TRACES_SCREENS/trace2_A5.png)

---

# A.6 - Un fichier BMP avec compression


Le fichier compressé **Image4.bmp** a un poids de `1120 octets`, soit environ **10 fois plus** que l'image d'origine non compressée.  Comme c'est un damier, la compression **RLE** est moins efficace car les pixels **alternent souvent le rouge et le blanc.** Il est à noter que l'adresse du début des pixels est située à `0x0436`. 

### Lorsque l'on observe l'encodage, on retrouve principalement la structure suivante:  
* **Le premier pixel :**  
   - qui indique une **couleur rouge** : `01 00`,  
* **Et le second pixel :**  
   - qui indique une **couleur blanche** : `01 01`.  

Puis un saut de ligne encodé par `00 00` qui indique que l'on passe à la ligne suivante.  Ce schéma se répète avec des alternances entre rouge, blanc, et sauts de ligne.

![](TRACES_SCREENS/trace_A6.jpg)

---

# A.7 - Image5.bmp


### Le fichier **Image5.bmp** pèse `1102 octets`, donc moins que **Image4.bmp**  

**Image5.bmp** contient des suites de pixels identiques, ce qui permet à la compression RLE d'être **plus efficace.** Concernant la structure de l'encodage : l'offset des pixels est indiqué à l'adresse `0x0436`. 

Il faut souligner que l’image **s’encode
d’abord par le bas à gauche.** Les octets `04 01` signifient que nous encodons **quatre pixels blancs**.
- Dans cette idée, les octets `04 00` signifient **4 pixels rouges**. 
- Chaque ligne est suivie par `00 00` pour indiquer un **retour à la ligne.**

La derniere ligne **alterne entre blanc et rouge**, soit `01 01` pour un pixel blanc et `01 00` pour un pixel rouge. Cela se répète pour couvrir les 4 pixels de cette ligne.

![](TRACES_SCREENS/trace_A7.png)

---
# A.8 - Création de l'Image6.bmp

Il faut **modifier quelques octets** : la ligne n’est plus unie, donc on code **deux pixels blancs** puis **un pixel rouge.**

![](TRACES_SCREENS/trace_A8.png)

---

# A.9 - Création de l'Image7.bmp

### Le procédé **reste le même** que vu auparavant. On encode de la manière suivante :
 

- La **première ligne** est une alternance de pixels blanc `01 01` et rouge `01 00`. La deuxième ligne est une **séquence** de quatre pixels verts encodés par `04 02` puis `04 00` pour les quatre pixels rouges.  

- La **quatrième ligne**, quant à elle, est un mélange de deux blancs, un bleu, et un blanc. Chaque pixel est **encodé respectivement** par `02 01`, `01 03` et `01 01`.  


![](TRACES_SCREENS/trace1_A9.png)
![](TRACES_SCREENS/trace2_A9.png)

---

# A.10 - Création de l'Image8.bmp

Cette image suit le même principe. Chaque **groupe de pixels** sont décrits par **deux octets indiquant leur nombre et leur couleur.**

![](TRACES_SCREENS/trace_A10.png)

---
# B.1 - Manipulation d'image en PYTHON 


```python
from PIL import Image

i1 = Image.open("Imagetest.bmp")

matrice = []
transpose = []

for y in range(i1.size[1]): # lit les pixels de l'image et rempli la matrice

    for x in range(i1.size[0]):
        c = i1.getpixel((x, y))
        matrice.append(c)

for x in range(i1.size[0]):
    for y in range(i1.size[1]):
        transpose.append(matrice[y * i1.size[0] + x])

sortie = i1.copy()

for y in range(i1.size[1]):
    for x in range(i1.size[0]):
        sortie.putpixel((x, y), transpose[y * i1.size[0] + x])

sortie.save("Imageout0.bmp")
 ```

### Le code pour **transposer l'image** fonctionne en **trois boucles for :**  

- Tout d'abord, l'image est importée dans  `i1`, puis une liste `matrice` est remplie avec les pixels, stockés sous forme **de tuples**, soit `(0, 0, 255)` par exemple pour du bleu.  

-  Une nouvelle liste `transpose` est ensuite construite en réorganisant les pixels de la liste `matrice` pour obtenir la transposée, c'est-à-dire que **les lignes deviennent colonnes et inversement.**

-  Puis une **copie de l'image** est créée dans `sortie`, et **les pixels de cette copie sont modifiés** en fonction des valeurs de `transpose`. Enfin, l'image transposée est sauvegardée sous le nom `"Imageout0.bmp"`.



![](TRACES_SCREENS/trace_B1.png)

---
# B.2 - Inversion d'image dans un miroir 

```python
from PIL import Image

i1 = Image.open('hall-mod_0.bmp')

matrice_valeurs = []

for y in range(i1.size[1]): # lit les pixels de l'image et les stocke dans la matrice

    ligne = []
    for x in range(i1.size[0]):
        c = i1.getpixel((x, y))
        ligne.append(c)
    matrice_valeurs.append(ligne)

matrice_miroir = []
for ligne in matrice_valeurs:
    matrice_miroir.append(ligne[::-1])  # inverse la ligne

sortie = i1.copy()

for y in range(i1.size[1]):
    for x in range(i1.size[0]):
        sortie.putpixel((x, y), matrice_miroir[y][x])

sortie.save("Imageout1.bmp")
```

### Le programme qui **inverse l'image dans un miroir** fonctionne comme ceci :


- En premier lieu, la matrice est **initialisée ligne par ligne**, où **chaque ligne est une liste des pixels** de cette ligne.

- Ensuite on utilise **`[: :-1]`** pour **inverser l'ordre des pixels** dans chaque ligne, ce qui crée donc une **version miroiroitée** de l'image.

- Et enfin, les pixels inversés sont **écrit**, encore une fois, **ligne par ligne dans l'image de sortie.**

![](TRACES_SCREENS/trace_B2.png)

---

# B.3 - Passer une image en niveaux de gris 

```python
from PIL import Image

i1 = Image.open('ImageExemple.bmp')

sortie = i1.copy()

for y in range(i1.size[1]):  # parcourt lignes
    for x in range(i1.size[0]):  # parcourt les colonnes

        couleur = i1.getpixel((x, y))

        rouge = couleur[0]
        vert = couleur[1]
        bleu = couleur[2]

        gris = (rouge + vert + bleu) // 3


        sortie.putpixel((x, y), (gris, gris, gris))

sortie.save("Imageout2.bmp")
```

### Pour le parcourt des pixels, on procède avec **deux boucles imbriquées :**
- Une qui parcourt la **hauteur (y),**
- Et une autre qui parcourt la **largeur (x).**

Ensuite, `getpixel((x, y))` va venir **récuperer** les couleurs du pixel sous **forme de tuple (R, G, B)** aux indices actuels.

Pour **calculer les valeurs de gris**, on **additionne R, G, et B** et on **divise la somme par 3** avec une division entière afin d'obtenir une **moyenne.**

Et enfin, `putpixel((x, y), (gris, gris, gris))` remplace chaque pixel par **un triplet avec la même valeur**, ce qui donne bien des nuances de **gris.**

![](TRACES_SCREENS/trace_B3.png)

---

# B.4 - Passer une image en noir et blanc:

```python
from PIL import Image

i1 = Image.open('ImageExemple.bmp')

sortie = i1.copy()

for y in range(i1.size[1]):  
    for x in range(i1.size[0]): 

        r, g, b = i1.getpixel((x, y))
        
        intensite = r + g + b  # additionne RGB pour la luminosité
       
        if intensite > 255 * 1.5:  # si intensité est sup à un seuil -> blanc
            couleur = (255, 255, 255)  
        else:
            couleur = (0, 0, 0)  # noir

        sortie.putpixel((x, y), couleur)

sortie.save("Imageout3.bmp")
```

### Une boucle for va parcourir chaque **ligne et chaque colonne** de l'image.

- La **fonction `getpixel((x, y))` récupère** la couleur `(R, G, B)` d'un pixel.

- Ensuite, il faut s'occuper de **l'intensité** d'un pixel : cette dernière est obtenue en **additionnant** les valeurs de **rouge, vert et de bleu.**
    - On calcule désormais la somme, et si elle est supérieure à une certaine valeur, alors on **affecte la couleur blanche** au pixel. Sinon, il devient **noir**.

Et enfin,  `putpixel` va venir **appliquer la nouvelle couleur au pixel.**

![](TRACES_SCREENS/trace_B4.png)

---

# B.5 - Cacher le logo en noir et blanc dans l'image hall-mod_0.bmp 

### Voici le programme pour **cacher le logo** :
```python
from PIL import Image
import module

i1 = Image.open('hall-mod_0.bmp')  
i2 = Image.open('Imageout3.bmp') 

sortie = i1.copy()

for y in range(i1.size[1]):
    for x in range(i1.size[0]):
        # récupère couleurs pixels 
        c1 = i1.getpixel((x, y))
        c2 = i2.getpixel((x, y))

        # si pixel de l'image cachée est blanc -> c2 devient 1 sinon 0
        if c2 == (255, 255, 255):
            c2 = 1
        else:
            c2 = 0

        sortie.putpixel((x, y), (c1[0], c1[1], module.cacher(c1[2], c2)))

sortie.save('Imageout_steg_1.bmp')

```
### Voici celui qui permet de **retrouver l'image** :

```python
from PIL import Image
import module

i1 = Image.open('Imageout_steg_1.bmp')

sortie = i1.copy()

for y in range(i1.size[1]):
    for x in range(i1.size[0]):

        c = i1.getpixel((x, y))

        # si le rouge est impair -> pixel blanc, sinon noir
        if module.trouver(c[2]) == 1:
            sortie.putpixel((x, y), (255, 255, 255))  # blanc
        else:
            sortie.putpixel((x, y), (0, 0, 0))  # noir

sortie.save('Imageout4.bmp')
```
Dans un premier temps, les valeurs des pixels des deux images sont **stockées dans deux variables.**

- Concernant le logo : si le **pixel est blanc**,  `c2` prend la valeur `1`, sinon elle prend la valeur `0`. 

- Ensuite, la copie de l'image principale est modifiée **grace a l'appel de fonction `cacher()`**, qui ajuste le rouge du pixel pour la rendre paire **(d'ou ici le `i%2` dans le module)** et **ajoute la valeur** de `c2`. 

- De ce fait, lorsque le rouge **devient impair**, le pixel caché **devient blanc.** Dans le cas inverse il **devient noir.**

Afin de retrouver l'image cachée, nous utilisons un autre programme qui **analyse chaque pixel de l'image modifiée :**

- Lorsque l'on appel la fonction `trouver()`, celle-ci vérifie si la couleur bleue du pixel est impaire. Si c'est **impair, le pixel retrouvé est blanc**. A l'inverse, il est **noir**.


---



# B.6 Stéganographie avec du texte

L'idée est d'intégrer un texte dans une image en utilisant la stéganographie. Dans ce cas, on peut coder chaque caractère du message en ASCII sur 8 bits, puis à insérer chaque bit dans les couleurs de l'image...



*Merci pour votre lecture*

