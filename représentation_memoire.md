---
created: 2020-10-26T10:02:10+01:00
modified: 2020-10-26T10:02:13+01:00
---

# Représentation en mémoire

Dans le chapitre sur le binaire on a vu comment représenter des entiers avec uniquement des 0 et des 1. Il faut une chose tout d'abord. Tout ce qui stocké dans un ordinateur est un nombre. Un nombre entier, un nombre virgule, vos photos de vacances, l'intégral de Game of Thrones. Tout et donc par extension tout n'est que des suites de 0 et de 1.

Lorsque l'on écrit un programme on ne fait manipuler ces 0 et 1.

## Les entiers naturels
Il faut savoir une chose, c'est que plus on a de cases pour écrire nos 0 et 1, plus on peut stocker de grand nombre entier.

Par exemple avec une case le nombre binaire le plus grand est 1.

D'ailleurs une case en informatique, on nomme habituellement ça un `bit`

Avec 2 bits, on peut au maximum écrire 3.

Avec 3 bits, 7.

Et ainsi de suite.

Pour connaître le nombre maximal representable sur `n` cases. Il existe une formule.

```
Nombre maximal = 2^n - 1
```

En programmation "bas niveau" on doit à tout moment connaître la taille des variables que l'on manipule. Ceci implique que lorsque l'on manipule un entier celui va avoir une taille en mémoire ( sous entendu de cases de 0 et de 1 ) initialisé à la déclaration de la variable.

En Rust, il existe une série de tailles pré définies :

```rs
let a : u8
```
Défini un entier naturels sur une série de 8 cases. Le `u` de `u8`, signifie unsigned. On va voir dans la partie suivante ce que ça implique.

En prenant la formule `2^n-1`, on arrive à la conclusion qu'on ne peut pas stocker plus que 255.

Il existe aussi :

```rs
let a : u16; # entiers non signé sur 16 bits
let a : u32; # entiers non signé sur 32 bits
let a : u64; # entiers non signé sur 64 bits
```

### Overflow
Que se passe-t-il si on veut stocker un plus grand nombre ?

Par exemple 455, en binaire il s'écrit 111000111. Ce qui signifie qu'il a besoin de 9 bits pour être représenté en mémoire.

En Rust c'est tout bonnement impossible. 

```rs
let a : u8 = 455;
```
Provoque une erreur de compilation :

```
= note: `#[deny(overflowing_literals)]` on by default = note: the literal `455` does not fit into the type `u8` whose range is `0..=255`
```
Comme on va le voir quand on va rentrer dans le dur du langage. La philosophie de Rust est d'éviter à l'utilisateur de faire des choses qui peuvent s'avérer dangereuses.

Il faut donc être attentif aux types de variables que l'on déclare.

Il est possible d'outre passer ce comportement en ajoutant un flag

`#[allow(overflowing_literals)]`

Qui va nous permettre de faire nos tests et montrer pourquoi c'est dangereux de manipuler des nombres trop grands avec des entiers à trop faible nombre de bits.

Si on active le flag de compilation précédent et l'on reprend le code.

```rs
let a : u8 = 455;
```

Il n'y aura aucune erreur de compilation, par contre on n'importe quoi en mémoire. Pour être exact on a 455 tronqué de un bit, celui le plus à gauche. 

```
(1)11000111 = 199
```

Notre valeur `a` est donc 199 au lieu de 455. Manipuler des variables sans vraiment savoir ce qu'elles contiennent peut s'avérer dangereux dans certains cas.

Par exemple il y a l'anecdote bien connue du jeu _Civilisation_.  Dans celui ci, chaque héros possède une jauge d'agressivité, cette jauge paramètre l'instinct guerrier d'une IA, et en temps normal baisse au cours du temps. Un des personnages : Gandhi, possède une agressivité de base très faible. Sauf qu'en fin de partie il se met subitement à déclencher le feu nucléaire sur tous les autres joueurs. Qu'est ce qui pousse Gandhi à devenir Mad Ghandi ? Et bien c'est assez simple. Cette agressivité est codé sur un entier de 8 bits, donc peut osciller de 0 à 255. Je vous est dit que la jauge baissait progressivement. Mais que se passe-t-il lorsqu'on est déjà à 0? Et bien demandons à Rust!

```rs
#[allow(overflowing_literals)]
fn main() {
    let mut a : u8 = 0;
    a -= 1;
}
```

Nouvelle interdiction du compilateur :

```shell
a -= 1; 
 ^^^^^^ attempt to compute `0_u8 - 1_u8` which would overflow
 = note: `#[deny(arithmetic_overflow)]` on by default 
```

Continuons d'encore casser les barrières que Rust créé pour notre sécurité !

```rs
#[allow(overflowing_literals, arithmetic_overflow)]
fn main() {
    let mut a : u8 = 0;
    a -= 1;
}
```

```shell
thread 'main' panicked at 'attempt to subtract with overflow'
```


Vous allez avoir droit à une belle erreur non pas à la compilation mais à l'exécution. Et ça c'est bien plus embêtant. Cela signifie que vous avez l'impression que votre code va tourner mais en faite il va planter, un code planté ne redémarre pas de lui même. Lorsque Rust plante, on dit qu'il panique. Le rôle d'un développeur Rust est que ça panique jamais ou presque.

Maintenant si on veut réellement simuler le comportement de Ghandi en Rust c'est possible. Mais il faut expliquer au compilateur qu'on est conscient de ce que l'on réalise. C'est le boulot de `wrapping_sub`. 

```rs
let mut a : u8 = 0;
a = a.wrapping_sub(1);// a = 255
```
 Je ne vous demande pas de tout comprendre maintenant. Ce que je  voudrais c'est que vous appréhendiez les divers problèmes que peut occasionner la manipulation des entiers en représentation binaire.
 
 ## Les entiers relatifs
 
 Une fois qu'on est capable de compter de 0 à presque l'infini ( faut juste une infinité de bits :p ).
 
 La question se pose de représenter quelque chose d'aussi simple que `-1` .
 
 Là on est un peu bloqué on a vu que `u8` allait de 0 à 255. Pas de trace de négatif. :/

L'astuce trouvé a été de détourner de son but premier le bit le plus à gauche.

Par convention un bit de valeur `0` est un `+` et `1` est un `-`.

Ce choix a été opéré car il permet à ce qu'un entier positif représenté en entier relatifs soient le même.

( Le `0b` est une notation très courante qui annonce au lecteur que ce qui suit est du binaire ).

Par exemple `1` s'écrit sur 8 bits `0b00000001`. Pour écrire `-1` sur 8 bits on va écrire `0b10000001`.

Ce qu'il faut bien comprendre c'est que rien ne distingue `0b10000001` l'entier naturel `129` de l'entier relatif `0b10000001` qui vaut `-1`. Seul la manière dont on interprète les données pousse vers la manipulation de -1 plutôt que 129.

Un autre détail à prendre en c'est qu'est-ce qui peut être représenté par un entier relatifs sur 8 bits.

Pour information ou appelle ceci un entier signé.

Et bien étant donné qu'on a réservé un des bits pour la représentation du signe. On n'est virtuellement plus que sur un entier sur 7 bits.

Si on reprend notre formule on est donc à une valeur répresentable de:

```
2^7-1 = 127
```
Aussi bien en positif qu'en négatif. L'amplitude d'un entier signé sur 8 bits est donc de -127 à 127.

À noter que le 0 possède deux représentations possible :
- `0b00000000`
- `0b10000000`

En effet `0` ou `-0` équivaut exactement à la même non quantité.

Bon après la théorie un peu de Rust, on est là pour ça à la base non ? :)

D'abord comment déclarer les entiers relatifs ?

Rust possède un mot-clef réservé de la même forme que les `u8` , `u16`, etc ...

Il s'agit de : 

```rs
let a : i8; # entier signé sur 8 bits
let a : i16; # entier signé sur 16 bits
let a : i32; # entier signé sur 32 bits
let a : i64; # entier signé sur 64 bits
```

### Overflow

Du coup l'overflow s'applique là aussi. Mais d'une manière différente. Au de repasser à 0 lorsque que l'on atteint la limite haute nous allons passer dans les négatifs !

Voyons ça avec Rust, je vous refais pas le topo. Rust vous en empêchera tout overflow. Vous devez donc wrapper pour avoir le comportement.

```rs
let mut a : i8 = 127;
a = a.wrapping_add(1);// a = -128
```
-128 ?? Oui vous avez bien lu, bon quelque cloche, je vous ai menti ou quoi ?

Non je vous ai pas menti c'est juste que les entiers relatifs ne sont pas stockés de cette manière à de divers problèmes dont :
- les 2 représentations de 0
- l'addition de nombre positif et négatif très peu optimisée.

On lui préférera donc un autre mode de représentation.

Il s'agit du **complément à 2**

#### Complément à 2

Qu'est-ce que c'est que ça encore ?

Il s'agit d'une manière différente et élégante de représenter un nombre relatifs en binaire sur un octet.

Prenons le nombre `-1` comment le passer en binaire ?

Et bien première étape, on représente son opposé : `1` en binaire, ça donne `0b00000001`.

Ce nombre binaire on va l'inverser, ce qui signifie que les bits à 1 passent à 0 et vice versa.

Ça nous donne un second nombre binaire, `0b11111110`.

Et on finit par lui ajouter 1.

`0b11111110  + 1= 0b11111111`.

Donc en mémoire -1 sera stocké sous la forme `0b11111111`

Si on résume l'algorithme :
- prendre la valeur absolue du nombre négatif
- inverser ce nombre
- lui ajouter 1

L'idée derrière cette représentation est de faciliter le travail du CPU. En effet

`4 -4 = 4 + (-4)` 

Si 4 = 0b00000100 alors -4 = 0b11111100

Donc

0b00000100 + 0b11111100 = 0b00000000 = 0

Le compte est bon !

Les nombres relatifs c'était déjà pas évident, mais maintenant on rentre dans un tout autre niveau d'abstraction.

Bienvenue dans le codage des nombres décimaux !

## Les décimaux

Alors tout d'abord il existe deux types de représentation s
- virgule fixe
- virgule flottante

### Virgule fixe

Prenons par exemple le nombre -12.1.875

Regardons comment il est composé :
- une partie entière, 12, celle ci peut s'écrire sur un minimum de 4 bits : `0b1100`.
- une partie décimale, qui sera un plus complexe à représenter, son nombre de bits nécessaires est pour le moment inconnu
- un signe négatif, representable sur un 1 bit.

#### Partie décimal
Vous vous rappelez quand je vous disais que le bit à gauche d'un autre était le double d'un autre ?
 
 Et bien par symétrie le bit à droite en est la moitié.
 
 Or le bit tout à droite est `2^0 = 1`  et donc sa moitié vaut 0.5
 
 On peut donc en sortir que 0b0.1 = 0.5.
 
 Maintenant comment trouver la représentation binaire de 0.1875 ?
 
 On va appliquer des multiplication successives:
 
 - 0.1875 x 2 = 0 + 0.375
 - 0.375    x 2 = 0 + 0.75
 - 0.75       x 2 = 1 + 0.5
 - 0.5          x 2 = 1 + 0.0
 
 La représentation binaire est donc 0b0.0011, on peut facilement le vérifier :
 
 ```
   0x2^0 + 0x2^-1 + 0x2^-2 + 1x2^-3 + 1x2^-4
 = 0 + 0 + 0 + 0.125 + 0.0625
 = 0.1875
 ```
 Le compte est bon!
 
 #### On rassemble le tout
 
 On résume on veut la représentation en binaire de `-12.1875`.
 
 On a
 
 - 12 = 0b1100
 - 0.1875 = 0b0011
 - le signe `-` est `1`
 
 En rassemblant les morceaux on se retrouve avec :
 
 -12.1875 = 0b1110.0011
 
 Comme la virgule n'existe pas en mémoire, ce qui sera réellement stocké est `0b111000011`
 
 On appelle cette représentation virgule fixe, car la position de la virgule est décidée à l'interprétation de la valeur binaire mais rien au sein de la représentation ne permet de distinguer :
 
 0b11100.0011 = -12.1875
 
 
 De 
 
 0b11100.011 =  -20.375
 
 C'est le fameux placement de cette virgule qui défini la valeur finale du nombre binaire.
 
 Ce qui implique que cette position doit être intégré dans le code source. C'est vraiment pas pratique. Ça signifie que deux codes sources qui n'ont pas fixé de la même manière vont interpréter les données différemment ! C'est absurde !
 
 De plus, à partir du moment où l'on veut stocker des grands nombres décimaux ou avec beaucoup de décimales. Le nombre de bits nécessaires explose!
 
 ### Précision de la représentation
 
 Si on prend par exemple le décimal 0.4. si on lui applique l'algorithme et qu'on le déroule :
 
 - 0.4 x 2 = 0 + 0.8
 - 0.8 x 2 = 1 + 0.6
 - 0.6 x 2 = 1 + 0.2
 - 0.2 x 2 = 0 + 0.4
 - ...

Oh ! Ça boucle !

Avec 4 steps on obtient :

```
0b0.0110 = 2^-2 + 2^-3 = 0.25 + 0.125 = 0.375
```
Avec 8 steps:

 - 0.4 x 2 = 0 + 0.8
 - 0.8 x 2 = 1 + 0.6
 - 0.6 x 2 = 1 + 0.2
 - 0.2 x 2 = 0 + 0.4
 - 0.4 x 2 = 0 + 0.8
 - 0.8 x 2 = 1 + 0.6
 - 0.6 x 2 = 1 + 0.2
 - 0.2 x 2 = 0 + 0.8


```
0b0.01100110 = 2^-2 + 2^-3 + 2^-6 + 2^-7 = 0,3984375
```

Pour 32 étapes et donc 32 bits on obtient :

```
0b01100110011001100110011001100110 = 0.39999999990686774
```

C'est presque 0.4 mais il y a tout de même une approximation qui subsiste.

Si on effectue des calculs avec ce nombre, l'approximation va se propager et amplifier l'erreur.

Il faut donc avoir à l'esprit qu'en informatique on ne peut matériellement pas stocker précisément n'importe quel décimaux. Il nous faudrait pour cela un nombre infini de bits !


### Représentation IEEE 754
 
 Il s'agit d'une convention qui permet de représenter un nombre réel avec le format plus mais résolvant le problème de la position de la virgule qui doit stocké dans le code décidant de la valeur du nombre binaire.
 
 Permettant à deux programmes ayant adopté le IEEE 754 de pouvoir interpréter un nombre binaire de la même façon sans information extérieur supplémentaires.
 
 La position de la virgule est encodé dans le binaire.
 
 #### Base décimal et notation scientifique
 
 Ce qui ont fait un peu de physique au lycée se rappelleront d'un concept appelé la **notation scientifiques**. Pour rappel il s'agit d'une notation qui a l'avantage de pouvoir à la représenter des nombres à la très grands ou très petits sur une quantité réduite de caractères. Le seul bémol est de sacrifier la précision de la donnée.
 
 Par exemple pour représenter 1 km 655 en mètres en utilisant l'écriture scientifique on écrirait:
 
 ```
 1.655 km =1.65 x 10^3 m
 ```
Le dernier 5 a disparut dans l'opération.

Pour écrire cette même distance en millimètres

```
1.655 km = 1.65 x 10^6 mm
```

Et ainsi de suite

De même on peut aller dans l'autre sens. 12 mm 345 en mètres.

```
12 mm 345 = 1.2 x 10^-3 m
```

Là aussi on perd de la précision mais l'ordre de grandeur est conservé.

Ce qu'il faut comprendre ici c'est que multiplier par `10^n` décale n fois la virgule vers la droite. Et que multiplier par `10^-n` revient à décaler n fois la virgule vers la gauche.

Si vous vous souvenez bien on déjà vu ce genre de comportement dans le chapitre sur le binaire où je vous avais expliqué les opérations usuelles. L'une d'elles le décalage à droite ou à gauche est particulièrement intéressante. En effet le décalage à droite de n crans signifie la multiplication de notre nombre binaire par `2^n`.

Exemple :

```
0b1 << 1 = 0b10 <=> 1 << 1 = 2
```

#### La Norme IEEE 754

Tout ce qui suit étant une norme, il y a une bonne dose d'arbitraire dans les choix.

Le premier choix est de est de représenter les nombres binaire sous la forme :

0bs1.M x 2^y

Où :
- s: signe du nombre ( 1 => "-1")
- M : la mantisse, représentation binaire du nombre en virgule fixe position 0
- y : l'exposant, le nombre de crans nécessaire à décaler la virgule pour retrouver le nombre réellement encodé.

Reprenons notre -12.1875.

Le signe est négatif, donc s vaut 1.

La représentation en virgule fixe de 12.1875 est 0b1100.0011. On appelle ce binaire le nombre M'

De combien de crans faut il décaler la virgule vers la gauche pour se retrouver sous la forme 0b1.M ?

Ici 3 crans, on a M = 0b1.1000011

Donc M' = M << 3

D'où M' = M x 2^3

On peut donc écrire que -12.1875 = (-1) x 0b1.1000011 x 2^3.

Maintenant il faut ranger cela en mémoire.

Je vous est dit que c'était une norme, et qu'il y avait de l'arbitraire. Et bien c'est ici qu'on en parle.

Tout d'abord le format de stockage 

Il existe 2 formalismes :

Une représentation 32 bits, en simple précision.

|   signe|  exposant |  mantisse |
|:-:|:-:|:-:|
|   s |   e |  M |
| 1bit  | 8bits  | 23 bits  |

Une représentation 64 bits, en double précision.

|   signe|  exposant |  mantisse |
|:-:|:-:|:-:|
|   s |   e |  M |
| 1 bit  | 11 bits  | 52 bits  |

Mais comme d'habitude il y a des subtilités:

Par exemple pour 2^-4, sa représentation M' sera : 0b0.0001

Il faut le ramener à une forme

M = 0b1.M' x 2^y

Ici M = 0b1.0 x 2^y

On cherche y où 0b0.0001 << y = 0b1.0

La réponse est 0b0.0001 >> 4 = 0b1.0 x 2^-4

Notre exposant est donc -4. On pourrait croire que les concepteurs de la norme aurait utilisé un formalisme avec bit de signe ou complément a 2. Mais ils ont décidé de faire autrement.

Afin de bénéficier du maximum d'amplitude dans cette représentation en 8bits et pour faciliter les opérations entre flottants. Il a été choisi de rajouter 127 à chaque exposant .

Donc notre - 4 devient - 4 + 127 = 123 que l'on passe en binaire.

123 = 0b1111011

Si on  veut un exposant en simple précision, on doit compléter avec des 0 à gauche pour atteindre les 8 bits, les 11 bits 

D'où y = 0b01111011 ou y2 = 0b00001111011.

Pour la mantisse on va aussi compléter, pour atteindre les 23 bits en simple précision et les 52 bits en double précision.

Pour rappel, la mantisse est de la forme 0b1.M

C'est donc 23 zéros ou 52 zéros en fonction de la précision demandé.

Voyons ce que ça donne avec du Rust, on est là pour ça à la base.

```rs
let a = 0.1;
println!("{:.32}", a);
```
Le print est un peu spécial ici, car par défaut Rust n'affiche pas toutes les décimales. En faisant cela on force son affichage. Ce qui nous donne :

0.10000000149011611938476562500000

On est d'accord c'est presque 0.1.

Mais que se passe-t-il si on fait

```rs