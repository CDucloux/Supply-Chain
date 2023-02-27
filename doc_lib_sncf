# Documentation de  `lib_sncf`

> L'objectif est ici de calculer les temps de trajet minimaux entre gares **SNCF** en `python`

## Instanciation de la classe **Gare**
***

On définit d'abord la classe **Gare** qui représente la gare d'une ville desservie par le réseau **SNCF**.

La classe est décorée avec le décorateur `@dataclass` et l'argument `frozen=True` est passé pour la rendre immuable.

La classe **Gare** a un attribut de type `str`, qui représente le nom de la gare (*Bordeaux, Paris...*). 
- Comme la classe est immuable, une fois qu'un objet **Gare** est créé, il ne peut plus être modifié.

```python
@dataclass(frozen=True)
class Gare:
    nom: str
```

## Instanciation de la classe **Itinéraire**
***

La classe **Itineraire** possède un attribut `etapes`, qui est une liste d'objets **Gare** $\Rightarrow$ cette liste représente les différentes gares que l'utilisateur doit visiter pour effectuer le voyage vers sa destination finale.

```python
@dataclass
class Itineraire:
    etapes: list[Gare]
```

## Instanciation de la classe **Carte**
***

La classe **Carte** a deux attributs :

- l'attribut `gares`, qui est une liste d'objets de classe **Gare**. Cette liste contient toutes les gares de France qui sont incluses sur la carte.

- l'attribut `connexions` est une liste de tuples contenant trois éléments :
  - Gare de départ
  - Gare d'arrivée
  - Durée du trajet (en heures) ~ `float`

$\Rightarrow$ Chaque tuple représente une connexion entre deux gares !

### [Méthode `__post_init__`]

La méthode spéciale `__post_init__` permet de vérifier certaines conditions directement après l'initialisation de la classe.

> En premier lieu, on vérifie si une des durées est inférieure ou égale à 0 dans chacun des tuple de la liste `self.connexions`
> - Une durée ne pouvant pas être négative, si `if any(poids <= 0 for _, _, poids in self.connexions)` renvoie le booléen `True`, alors une erreur de valeur est levée.

> En second lieu, on vérifie avec une boucle `for` si une gare spécifiée (de départ ou d'arrivée) est dans la liste des gares disponibles. Si ce n'est pas le cas, on lève une erreur de valeur.

```python
@dataclass
class Carte:
    gares: list[Gare]
    connexions: list[tuple[Gare, Gare, float]]

    def __post_init__(self):
        if any(poids <= 0 for _, _, poids in self.connexions):
            raise ValueError("Les durées sont forcément positives!")

        for depart, arrivee, _ in self.connexions:
            if depart not in self.gares:
                raise ValueError(f"{depart} n'est pas dans la liste des gares!")
            if arrivee not in self.gares:
                raise ValueError(f"{arrivee} n'est pas dans la liste des gares!")
```

## Transformation de la carte en graph `NetworkX`
***

On passe en entrée la carte de classe **Carte** et on attend en sortie un objet de classe **nx.graph** :

- On commence par initialiser la variable `resultat` qui est un graph `NetworkX` vide.
- On ajoute tous les nœuds de la carte en utilisant la méthode `add_nodes_from` de `NetworkX`, en passant la liste des objets **Gare** stockés dans l'attribut `gares` de l'objet **Carte**.
- Enfin, on ajoute les connexions de la carte en utilisant la méthode `add_edges_from` de `NetworkX`. Pour cela, on  parcourt la liste des connexions stockées dans l'attribut connexions de l'objet **Carte** et on ajoute chaque connexion sous forme d'un tuple `(départ, arrivée, {"duree": poids})`

```python
def _convertit_en_nx(carte: Carte) -> nx.Graph:
    resultat = nx.Graph()
    resultat.add_nodes_from(carte.gares)
    resultat.add_edges_from(
        (depart, arrivee, {"duree": poids})
        for depart, arrivee, poids in carte.connexions
    )
    return resultat
```

## Exceptions
***

**On définit 2 exceptions :**

- Quand il n'y a pas de chemin possible entre les 2 gares
- Quand une des 2 gares est inconnue

```python
class PasDeChemin(Exception):
    pass

class GareInconnue(Exception):
    pass
```

## Déterminer le chemin le plus court possible
***

La fonction `determine_trajet` prend en argument deux objets Gare *depart* & *arrivee*, ainsi qu'un objet Carte: carte, et renvoie un objet Itineraire représentant le trajet le plus court possible entre depart et arrivee sur la carte. 

La fonction lève une exception `GareInconnue` si *depart* ou *arrivee* ne sont pas des objets Gare valides, ou une exception `PasDeChemin` si *depart* et *arrivee* ne sont pas connectées sur la carte.

- La fonction commence par vérifier que *depart* et *arrivee* sont des gares valides en parcourant la liste des gares stockées dans l'attribut gares de carte à l'aide d'une boucle `for`. Si *depart* ou *arrivee* ne correspond à aucune des gares de la carte, la fonction lève une exception `GareInconnue` avec un message d'erreur spécifique.
  
- Ensuite, la fonction utilise la fonction _`convertit_en_nx` pour convertir l'objet carte en un objet Graph `NetworkX`, qui peut être utilisé pour calculer le chemin le plus court. La fonction `shortest_path` de NetworkX est appelée avec l'objet Graph, la gare de départ, la gare d'arrivée et l'attribut pondéré "duree". Cette fonction renvoie un itérateur qui représente le chemin le plus court possible entre les deux gares sur la carte.
  
- Si aucun chemin n'est trouvé, la fonction `shortest_path` lève une exception `NetworkXNoPath`. Dans ce cas, la fonction lève l'exception `PasDeChemin` avec un message d'erreur spécifique.
  
- Enfin, la fonction renvoie un objet **Itineraire** construit à partir de l'itérateur `resultat_nx` obtenu avec `shortest_path`. L'objet **Itineraire** est construit en passant l'itérateur en tant qu'argument de la méthode Itineraire(etapes=...). Cela crée un nouvel objet **Itineraire** contenant une liste ordonnée des gares à parcourir pour suivre le chemin le plus court entre *depart* et *arrivee*.

```python
def determine_trajet(depart: Gare, arrivee: Gare, carte: Carte) -> Itineraire:
    if all(depart != gare for gare in carte.gares):
        raise GareInconnue(f"{depart} n'est pas une gare valide!")
    if all(arrivee != gare for gare in carte.gares):
        raise GareInconnue(f"{arrivee} n'est pas une gare valide!")

    graphe = _convertit_en_nx(carte)
    try:
        resultat_nx = nx.shortest_path(
            graphe, source=depart, target=arrivee, weight="duree"
        )
    except nx.exception.NetworkXNoPath:
        raise PasDeChemin(f"{depart} et {arrivee} ne sont pas connectées!")
    return Itineraire(etapes=resultat_nx)
```

 ## Construction de la carte de France
 ***

La fonction `_construit_carte` commence par faire une **déstructuration de liste** (permet de définir plusieurs variables en une seule instruction en affectant les éléments d'une liste à chaque variable correspondante)

- Les noms des variables pa, to, bo, ly, mo et tls correspondent aux noms des gares respectives dans la liste gares, qui est créée en même temps. 
- Chaque élément de la liste est une instance de la classe **Gare**.

Ensuite, on ajoute plusieurs autres gares à la liste *gares* en utilisant la méthode `extend` (elle diffère de la méthode `append` car elle permet d'ajouter directement plusieurs éléments à la liste au lieu d'un à la fois)

On retourne l'objet de classe **Carte** correspondant à la carte de France avec les gares en question et leurs connexions.

> Attention cependant, les gares ajoutées avec la méthode `extend` n'ont pas été préalablement déstructurées, et ne sont donc reliées à aucune autre gare dans cet exemple !

***

- La dernière ligne de code construit simplement la carte et l'assigne à une variable **CONSTANTE**.

 ```python
def _construit_carte():
    pa, to, bo, ly, mo, tls = gares = [
        Gare(nom="Paris"),
        Gare(nom="Tours"),
        Gare(nom="Bordeaux"),
        Gare(nom="Lyon"),
        Gare(nom="Montpellier"),
        Gare(nom="Toulouse"),
    ]
    gares.extend(
        [
            Gare(nom)
            for nom in (
                "Lille",
                "Nice",
                "Marseille",
                "Strasbourg",
                "Brest",
                "Nantes",
                "Pau",
                "Rennes",
            )
        ]
    )
    return Carte(
        gares=gares,
        connexions=[
            (pa, to, 1),
            (to, bo, 2),
            (bo, tls, 4),
            (tls, mo, 4),
            (mo, ly, 2),
            (ly, pa, 2),
        ],
    )

CARTE_FRANCE = _construit_carte()
 ```
