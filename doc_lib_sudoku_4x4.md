# Documentation de  `lib_sudoku_4x4`

> L'objectif est ici de remplir une grille vide ou partiellement remplie de sudoku de dimension $4\times4$ en `python`

| <!-- --> | <!-- --> | <!-- --> | <!-- --> |
| -------- | -------- | -------- | -------- |
| 1        | 2        |          | 4        |
|          | 3        | 2        |          |
| 3        |          | 1        | 2        |
| 2        | 1        |          | 3        |

***

**CONTRAINTES :**

- Pas de répétition sur une ligne
- Pas de répétition sur une colonne
- Pas de répétition dans un sous-bloc du sudoku : $\begin{bmatrix}
1 & 2\\
4 & 3
\end{bmatrix}$

## Instanciation de la classe **Grille**
***
On crée la classe **Grille**, à l'aide du décorateur `@dataclass` $\Rightarrow$ permet de créer rapidement des classes avec des propriétés typées et des méthodes spéciales pré-définies.

La classe **Grille** a une seule propriété appelée `cases`, qui est une liste d'entiers

La méthode spéciale `__post_init__` est définie dans la classe **Grille**. Elle est appelée automatiquement après l'intialisation d'une instance de la classe et est utilisée pour valider la propriété `cases` dont les contraintes sont :

- Il doit y avoir exactement $16$ cases
- Les cases doivent être $\in ]0,4[$

*Remarque : `self` est un paramètre spécial qui se réfère à l'instance de la classe elle-même.*


```python
@dataclass
class Grille:

    cases: list[int]

    def __post_init__(self):
        if len(self.cases) != 16:
            raise ValueError("Il doit y avoir exactement 16 cases.")
        if any(case < 0 or case > 4 for case in self.cases):
            raise ValueError("Les valeurs permises sont 0,1,2,3 ou 4!")
```
## Méthode spéciale `__str__`
***

Cette méthode permet de définir la représentation en *chaîne de caractères* d'une instance de la classe **Grille**. Lorsque la fonction `print` est appelée sur une instance de la classe, la méthode `__str__` est automatiquement appelée et renvoie une chaîne de caractères représentant la grille.

- On commence par initialiser une liste vide (*cases*) et on la remplit avec l'argument cases de la classe **Grille** (`self.cases`) $\Rightarrow$ la liste finale *cases* est de longueur $16$ et est de type `str`, dont les $0$ ont été modifiés.
- La chaîne de caractères multiligne **resultat** (`"""..."""`) est construite à l'aide d'une syntaxe de formatage de chaîne de caractères qui utilise l'opérateur `*` pour déstructurer la liste *cases* en une séquence d'arguments positionnels.
-  Les arguments positionnels sont utilisés pour remplacer les marqueurs de positionnement `{}` dans la chaîne de caractères.

```python
def __str__(self) -> str:

        cases = list()
        for case in self.cases:
            if case == 0:
                cases.append(" ")
            else:
                cases.append(str(case))
        resultat = """
---------
|{}|{}|{}|{}|
---------
|{}|{}|{}|{}|
---------
|{}|{}|{}|{}|
---------
|{}|{}|{}|{}|
---------
""".format(
            *cases
        )
        return resultat
```

*Remarque : l'ensemble des fonctions de vérification sont privées et l'utilisateur ne verra dans l'interface publique du module que la fonction de résolution stricte.*

## Vérification des lignes
***

- La première boucle `for` permet d'itérer à partir des premiers indices de chaque ligne : $(0,4,8,12)$ grâce à un pas de $4$.
- La seconde boucle `for` permet d'itérer dans les valeurs autorisées (on exclut $0$ car il peut y avoir des répétitions puisque $0$ représente une case vide !)
- La fonction compte ensuite le nombre d'occurrences de `valeur` dans la ligne actuelle : $(0,1,2,3)$, etc... et renvoie `True` si ce nombre est supérieur à 1 (c'est à dire si il y a 2 fois 1, 2, 3 ou 4) $\Rightarrow$**Il y a alors une répétition dans les lignes du sudoku.**
```python
def _detecte_probleme_ligne(grille: Grille) -> bool:
    for i in range(0, 16, 4):
        for valeur in (1, 2, 3, 4):
            if grille.cases[i : i + 4].count(valeur) > 1:
                return True
    return False
```
## Vérification des colonnes
***

- La première boucle `for` parcourt les indices de colonnes $[0,1,2,3]$
- La seconde boucle `for` permet d'itérer dans les valeurs autorisées (on exclut $0$ car il peut y avoir des répétitions puisque 0 représente une case vide !)
- Pour chaque colonne et chaque valeur permise, la fonction compte le nombre d'occurrences de cette valeur dans la colonne correspondante (en utilisant la notation de slicing de liste `[i::4]` pour sélectionner tous les éléments de la colonne) et renvoie `True` si ce nombre est supérieur à 1 $\Rightarrow$ **Il y a alors une répétition dans la colonne.**
- La notation `[i::4]` sélectionne tous les éléments de la colonne en partant de l'élément d'indice i et en sautant 4 éléments à chaque fois. *Exemple :* 
  - Si `i` vaut 0, `grille.cases[i::4]` sélectionne les éléments d'indices $[0,4,8,12]$
  - Si `i` vaut 1, `grille.cases[i::4]` sélectionne les éléments d'indices $[1,5,9,13]$

```python
def _detecte_probleme_colonne(grille: Grille) -> bool:
    for i in range(4):
        for valeur in (1, 2, 3, 4):
            if grille.cases[i::4].count(valeur) > 1:
                return True
    return False
```

## Vérification des blocs
***

- La première boucle parcourt les indices de blocs de 0 à 3 inclusivement, en sélectionnant les indices des cases de chaque bloc en utilisant une séquence de valeurs prédéfinie $(0, 2, 8, 10)$. 
- La deuxième boucle parcourt les valeurs permises $(1, 2, 3, 4)$. 
- Pour chaque bloc et chaque valeur permise, la fonction sélectionne les 4 cases correspondantes en utilisant : `[grille.cases[i + j] for j in (0, 1, 4, 5)]` qui ajoute l'indice i à chaque valeur de la séquence $(0, 1, 4, 5)$ pour sélectionner les cases de ce bloc.  *Exemple :* 
    - Si `i` vaut 0, la séquence $(0, 1, 4, 5)$ correspond aux indices des cases suivantes : 0, 1, 4 et 5, qui forment le premier bloc de la grille. 
    - Si i vaut 2, la séquence $(0, 1, 4, 5)$ correspond aux indices des cases suivantes : 2, 3, 6 et 7, qui forment le deuxième bloc de la grille, et ainsi de suite.

```python
def _detecte_probleme_bloc(grille: Grille) -> bool:
    """Détecte la répétition sur un bloc."""
    for i in (0, 2, 8, 10):
        for valeur in (1, 2, 3, 4):
            if [grille.cases[i + j] for j in (0, 1, 4, 5)].count(valeur) > 1:
                return True
    return False
```

## Assemblage des contraintes
***

Permet de déterminer si une grille du sudoku passée est valide ou non en utilisant les fonctions `_detecte_probleme` $\Rightarrow$ Si les 3 tests `if` passent, alors cela signifie que chaque fonction `_detecte_probleme` renvoie le *booléen* `True`, dans ce cas, la grille est valide !

```python
def _est_valide(grille: Grille) -> bool:
    if _detecte_probleme_ligne(grille):
        return False
    if _detecte_probleme_colonne(grille):
        return False
    if _detecte_probleme_bloc(grille):
        return False
    return True
```

## Grille remplie ou non ?
***

Cette fonction permet de déterminer si une grille est remplie testant si le nombre d'occurences de $0$ est égal à $0$ $\Rightarrow$ Si c'est le cas, la fonction renvoie le booléen `True` et la grille est remplie sinon elle ne l'est pas.

```python
def _est_remplie(grille: Grille) -> bool:
    return grille.cases.count(0) == 0
```

## Génération du remplissage
***

On commence par créer une copie de la liste contenant la grille de sudoku, car nous allons ensuite la modifier. Cette copie est la liste `nouvelles_cases` ci-dessous.

On crée ensuite la variable `index_case_vide` qui cherche la position du premier $0$ dans la liste `nouvelles_cases`, qu'on insère dans un bloc `try` & `except` pour gérer le problème d'une **grille déjà remplie**.

La fonction définie renvoie un `Generator` en sortie. Un générateur est une fonction qui utilise le mot-clé `yield` pour renvoyer une séquence d'éléments au lieu d'une liste complète. Contrairement aux fonctions qui renvoient une liste, les générateurs ne génèrent pas toutes les valeurs à la fois, mais une à la fois.

*Remarque : L'annotation de type `Generator[Grille, None, None]` indique que la fonction `_genere_voisines` renvoie un générateur qui génère des instances de la classe Grille. Les deux `None` suivants sont les types des arguments que le générateur peut recevoir et le type de valeur que le générateur peut renvoyer. Dans ce cas, ces deux types sont `None`, car notre générateur ne prend pas d'arguments et ne renvoie aucune valeur en fin d'exécution.*

- Enfin, la boucle `for` permet de remplacer la valeur du premier $0$ trouvé dans la grille par $1,2,3,4$.

```python
def _genere_voisines(grille: Grille) -> Generator[Grille, None, None]:
    nouvelle_cases = [case for case in grille.cases]
    try:
        index_case_vide = nouvelle_cases.index(0)
    except ValueError:
        raise ValueError("La grille est déjà remplie.")
    for remplissage in (1, 2, 3, 4):
        nouvelle_cases[index_case_vide] = remplissage
        yield Grille(cases=nouvelle_cases)
```

## Résolution du sudoku
***

Si la grille est remplie et si elle est valide, alors cela signifie que c'est une solution, on renvoie donc la grille d'entrée associée qui est la réponse. Si elle est remplie mais qu'elle n'est pas valide, alors on lève l'exception `PasdeSolution`.

Si la grille n'est pas remplie, la fonction génère toutes les grilles voisines valides en appelant la fonction `_genere_voisines` puis vérifie la validité de chaque grille voisine (il y en a 4 à chaque fois) en utilisant la fonction `_est_valide`

Si une grille voisine est valide, la fonction appelle récursivement `resoud_sudoku` avec cette grille voisine comme argument, pour continuer la recherche de la solution

```python
def resoud_sudoku(grille: Grille) -> Grille:
    if _est_remplie(grille):
        if _est_valide(grille):
            return grille
        else:
            raise PasDeSolution
    else:
        for voisine in _genere_voisines(grille):
            if _est_valide(voisine):
                try:
                    solution = resoud_sudoku(voisine)
                except PasDeSolution:
                    pass
                else:
                    return solution
        raise PasDeSolution
```

