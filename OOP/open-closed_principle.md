## Open-Closed Principle (OCP)

Le principe d'ouverture-fermeture (OCP) pour la conception orientée objet 
a été introduit par Bertrand Meyer en 1988. Il signifie que :

> Les entités logicielles (classes, modules, fonctions, etc.) doivent 
> être ouvertes à l'extension, mais fermées à la modification.

Pour comprendre le principe d'ouverture-fermeture, considérons la classe 
`Shape` suivante :

```python
# shapes_ocp_v1.py

from math import pi

class Shape:
    def __init__(self, shape_type, **kwargs):
        self.shape_type = shape_type
        if self.shape_type == "rectangle":
            self.width = kwargs["width"]
            self.height = kwargs["height"]
        elif self.shape_type == "circle":
            self.radius = kwargs["radius"]

    def calculate_area(self):
        if self.shape_type == "rectangle":
            return self.width * self.height
        elif self.shape_type == "circle":
            return pi * self.radius**2
```

Le constructeur de `Shape` prend un argument `shape_type` qui peut être `rectangle` ou `circle`. 
Il prend également en charge un ensemble spécifique d'arguments mots-clés utilisant la syntaxe `**kwargs`. 
Si vous définissez le type de forme sur `rectangle`, vous devez également passer les arguments 
mots-clés `width` et `height` afin de construire un rectangle correct.

En revanche, si vous définissez le type de forme sur `circle`, vous devez également passer un 
argument `radius` pour construire un cercle.

> Remarque : Cet exemple peut paraître un peu extrême. Son but est d'exposer clairement l'idée 
> fondamentale du principe ouvert-fermé.

`Shape` dispose également d'une méthode `.calculate_area()` qui calcule l'aire de la forme 
actuelle en fonction de son `.shape_type` :

```python
>>> from shapes_ocp import Shape

>>> rectangle = Shape("rectangle", width=10, height=5)
>>> rectangle.calculate_area()
50
>>> circle = Shape("circle", radius=5)
>>> circle.calculate_area()
78.53981633974483
```

La classe fonctionne. Vous pouvez créer des cercles et des rectangles, calculer leur aire, etc. 
Cependant, la classe est plutôt mauvaise. À première vue, quelque chose semble anormal.

Imaginez que vous deviez ajouter une nouvelle forme, par exemple un carré. Comment feriez-vous 
cela ? L'option consiste à ajouter une autre clause `elif` à `.__init__()` et à `.calculate_area()` 
afin de répondre aux exigences d'une forme carrée.

Devoir effectuer ces modifications pour créer de nouvelles formes signifie que votre classe est 
ouverte aux modifications. Cela viole le principe d'ouverture-fermeture. Comment pouvez-vous 
corriger votre classe pour la rendre ouverte aux extensions, mais fermée aux modifications ? 
Voici une solution possible :

```python
# shapes_ocp_v2.py

from abc import ABC, abstractmethod
from math import pi

class Shape(ABC):
    def __init__(self, shape_type):
        self.shape_type = shape_type

    @abstractmethod
    def calculate_area(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        super().__init__("circle")
        self.radius = radius

    def calculate_area(self):
        return pi * self.radius**2

class Rectangle(Shape):
    def __init__(self, width, height):
        super().__init__("rectangle")
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        super().__init__("square")
        self.side = side

    def calculate_area(self):
        return self.side**2
```

Dans ce code, vous avez entièrement [refactorisé](https://realpython.com/python-refactoring/) la classe `Shape`, 
la transformant en [classe de base abstraite (ABC)](https://realpython.com/python-classes/#creating-abstract-base-classes-abc-and-interfaces). 
Cette classe fournit l'[interface (API)](https://en.wikipedia.org/wiki/API) requise pour toute 
forme que vous souhaitez définir. Cette [interface](https://realpython.com/python-interface/) se compose d'un attribut 
`.shape_type` et d'une méthode `.calculate_area()` que vous devez surcharger dans toutes les sous-classes.

