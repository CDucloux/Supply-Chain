# Documentation de  `lib_probleme`

> L'objectif est de transposer le problème du Berger, Loup, Mouton et Chou en `python`

## Instanciation de la classe **Rive**
***
On crée la classe **Rive**, qui hérite elle-même de la classe **Enum** pour modéliser la rive.

**Enum** permet de définir des constantes nommées, ici  :
- `GAUCHE` $\Rightarrow$ *rive gauche*
- `DROITE` $\Rightarrow$ *rive droite*


```python
class Rive(Enum):

    GAUCHE = "gauche"
    DROITE = "droite"
```

## Instanciation de la classe **Etat**
***
La classe **Etat** possède 4 attributs qui sont tous de type **Rive** (classe précédemment créee).

L'utilisation du décorateur `@dataclass` permet de créer des méthodes spéciales :
- `__init__`
- `__repr__`
- `__eq__`
- `__hash__`
  
>**Intérêt** : écrire moins de code pour définir une classe de données et faciliter la gestion des instances de la classe.

Avec l'argument `frozen=True`, les instances de la classe seront immuables, c'est-à-dire qu'elles ne pourront pas être modifiées une fois créées.

```python
@dataclass(frozen=True)
class Etat:

    berger: Rive
    loup: Rive
    mouton: Rive
    chou: Rive
```
## Modélisation du départ et de l'arrivée
***

En utilisant la classe **Etat**, on peut représenter l'état du jeu à un moment donné !

- Au *début* du jeu, tous les individus sont localisés sur la rive gauche
- A la *fin* du jeu, tous les individus sont localisés sur la rive droite

```python
DEPART = Etat(
    berger=Rive.GAUCHE,
    loup=Rive.GAUCHE,
    mouton=Rive.GAUCHE,
    chou=Rive.GAUCHE,
)

ARRIVEE = Etat(
    berger=Rive.DROITE,
    loup=Rive.DROITE,
    mouton=Rive.DROITE,
    chou=Rive.DROITE,
)
```
## Recensement de tous les états
***

On cherche à créer une liste de toutes les combinaisons possibles d'emplacement pour chaque individu. On aura ainsi $2^4 = 16$ combinaisons totales.

::: mermaid
graph TD;
    style Rive_gauche fill:#c7e9b4,stroke:#48a868,stroke-width:2px,color:#ffffff;
    style Rive_droite fill:#fdae61,stroke:#d7191c,stroke-width:2px,color:#ffffff;
    berger-->Rive_gauche;
    berger-->Rive_droite;
    loup-->Rive_gauche;
    loup-->Rive_droite;
    mouton-->Rive_gauche;
    mouton-->Rive_droite;
    chou-->Rive_gauche;
    chou-->Rive_droite;
    berger-->loup-->mouton-->chou;

:::

> **/!\\** Le préfixe `_` devant la variable **etats** permet d'indiquer que cette variable est destinée à une utilisation interne et privée - Autrement dit, elle ne sera pas directement utiisée par l'utilisateur du module

```python
_etats = list()
for cote_berger in (Rive.GAUCHE, Rive.DROITE):
    for cote_loup in (Rive.GAUCHE, Rive.DROITE):
        for cote_mouton in (Rive.GAUCHE, Rive.DROITE):
            for cote_chou in (Rive.GAUCHE, Rive.DROITE):
                _etats.append(
                    Etat(
                        berger=cote_berger,
                        loup=cote_loup,
                        mouton=cote_mouton,
                        chou=cote_chou,
                    )
                )
```

## Vérification des états valides
***

Précedemment, on a récupéré l'ensemble des états sans se préoccuper si ceux-ci violaient les contraintes du problème. 

**Cependant :**

- Si le :wolf: et le :sheep: sont sur la même rive alors que le berger est sur l'autre rive, alors le :wolf: dévore le :sheep: $\Rightarrow$ **l'état est invalide** 
- Si le :sheep: et le 🥬 sont sur la même rive alors que le berger est sur l'autre rive, alors le :sheep: dévore le 🥬 $\Rightarrow$ **l'état est invalide** 

> **/!\\** La fonction utilise l'annotation de type pour indiquer que l'argument d'entrée etat est de classe **Etat** et que la fonction retourne un booléen.

```python
def _est_valide(etat: Etat) -> bool:
    if etat.loup == etat.mouton and etat.loup != etat.berger:
        return False
    if etat.mouton == etat.chou and etat.mouton != etat.berger:
        return False
    return True
```

## Génération des sommets
***

On obtient $10$ sommets valides.

> **/!\\** Par convention, pour signifier une constante en `python`, on écrit son nom en **MAJUSCULES**.

```python
SOMMETS = [etat for etat in _etats if _est_valide(etat)]
```
## Sont reliés ?
***
On souhaiterait maintenant vérifier si deux états valides donnés sont reliés, c'est-à-dire s'il est possible de passer d'un état à l'autre en une seule traversée.

- Il faut commencer par vérifier si deux états ont le berger sur la même rive $\Rightarrow$ Si c'est le cas, **les deux états ne sont pas reliés**. 
- Sinon, la fonction compte le nombre de différences entre les positions du loup, du mouton et du chou dans les deux états. Si ce nombre est inférieur à 2, cela signifie qu'il n'y a qu'un seul individu qui a changé de position entre les deux états $\Rightarrow$ **Les deux états sont reliés**.


```python
def _sont_relies(etat1: Etat, etat2: Etat) -> bool:
    if etat1.berger == etat2.berger:
        return False
    nombre_changements = 0
    if etat1.loup != etat2.loup:
        nombre_changements = nombre_changements + 1
    if etat1.mouton != etat2.mouton:
        nombre_changements = nombre_changements + 1
    if etat1.chou != etat2.chou:
        nombre_changements = nombre_changements + 1
    if nombre_changements < 2:
        return True
    else:
        return False
```

## Arrêtes
***

On définit une liste nommée ARRETES qui contient les paires de sommets reliées, c'est-à-dire les traversées possibles entre deux états différents


```python
ARRETES = [
    (sommet1, sommet2)
    for sommet1 in SOMMETS
    for sommet2 in SOMMETS
    if _sont_relies(sommet1, sommet2)
]
```

## Affichage propre

La dernière fonction du module permet de faire un affichage plus adapté pour l'utilisateur.

```python
def pretty_print(etat: Etat):
    return (
        f'{"B" if etat.berger == Rive.GAUCHE else "X"} '
        f'{"L" if etat.loup == Rive.GAUCHE else "X"} '
        f'{"M" if etat.mouton == Rive.GAUCHE else "X"} '
        f'{"C" if etat.chou == Rive.GAUCHE else "X"}\n'
        "-------\n"
        f'{"B" if etat.berger == Rive.DROITE else "X"} '
        f'{"L" if etat.loup == Rive.DROITE else "X"} '
        f'{"M" if etat.mouton == Rive.DROITE else "X"} '
        f'{"C" if etat.chou == Rive.DROITE else "X"}'
    )
```

# Documentation de  `lib_graphe`

Cette fonction permet de récupérer la liste des voisins d'un sommet en parcourant la liste des arêtes.

Elle prend en entrée un sommet et une liste d'arêtes. Le sommet correspond au sommet dont on veut récupérer les voisins et les arêtes correspondent à la liste des arêtes du graphe.

La fonction parcourt chaque arête de la liste et teste si le premier sommet de l'arête est égal au sommet d'entrée. Si c'est le cas, alors le deuxième sommet de l'arête est un voisin du sommet d'entrée, et est ajouté à la liste des voisins.

La fonction renvoie ensuite la liste des voisins trouvés.

***

La fonction utilise une stratégie de parcours en largeur (BFS) pour explorer le graphe à partir du sommet de départ, en vérifiant à chaque fois si le sommet courant est égal au sommet d'arrivée. Si c'est le cas, la fonction renvoie True car cela signifie qu'un chemin a été trouvé. Si aucun chemin n'est trouvé après avoir exploré tous les sommets, la fonction renvoie False.

La fonction _recupere_voisins est appelée pour récupérer les voisins de chaque sommet visité, c'est-à-dire les sommets qui sont reliés au sommet courant par une arrête. Cette fonction parcourt la liste des arrêtes et ajoute les sommets reliés au sommet courant à une liste de résultats.

En somme, la fonction sont_connectes permet de vérifier si deux sommets sont reliés entre eux dans le graphe en utilisant une stratégie de parcours en largeur.
***
La fonction _determine_chemin prend deux arguments : arrivee, qui est le sommet d'arrivée, et vu_en_premier_par, qui est un dictionnaire où les clés sont les sommets visités et les valeurs sont les sommets d'où ils ont été visités en premier (ou None s'ils ont été visités en premier lieu).

La fonction parcourt le dictionnaire en partant du sommet d'arrivée et remonte la chaîne de sommets visités en premier jusqu'au sommet de départ, en ajoutant chaque sommet visité à une liste intermédiaire resultat_intermediaire.

Ensuite, elle inverse l'ordre de la liste intermédiaire à l'aide de reversed et stocke les sommets dans une liste appelée resultat. Cette liste contient le chemin à suivre pour aller du sommet de départ au sommet d'arrivée.

En fin de compte, la fonction renvoie cette liste resultat.
***
La fonction cherche_chemin prend trois arguments en entrée: le sommet de départ, le sommet d'arrivée et la liste des arêtes du graphe. Elle renvoie une liste de sommets représentant le chemin de depart à arrivee.

La fonction commence par initialiser une liste a_visiter contenant uniquement le sommet de départ, une liste visites pour stocker les sommets visités et un dictionnaire vu_en_premier_par pour stocker le sommet qui a mené à chaque sommet visité.

Ensuite, tant qu'il y a encore des sommets à visiter (c'est-à-dire que a_visiter n'est pas vide), la fonction retire le premier sommet de a_visiter et l'ajoute à la liste visites. Elle itère ensuite sur tous les voisins de ce sommet (récupérés à partir de la liste des arêtes avec la fonction _recupere_voisins). Si le voisin n'a pas encore été visité, elle l'ajoute à la liste a_visiter et met à jour le dictionnaire vu_en_premier_par en ajoutant une entrée pour ce voisin avec comme valeur le sommet courant.

La fonction s'arrête si elle visite le sommet d'arrivée, auquel cas elle renvoie le chemin trouvé en appelant la fonction _determine_chemin. Si elle a parcouru tout le graphe sans trouver le sommet d'arrivée, elle lève une exception PasDeChemin.
