## Liskov Substitution Principle (LSP)

Le principe de substitution de Liskov (LSP) a été introduit par Barbara Liskov 
lors de la [conférence OOPSLA](https://en.wikipedia.org/wiki/OOPSLA) en 1987. 
Depuis, ce principe est devenu un élément fondamental de la programmation orientée 
objet. Il stipule que :

> Les sous-types doivent être substituables à leurs types de base.

Par exemple, si vous avez un morceau de code qui fonctionne avec une classe `Shape`, 
vous devriez pouvoir substituer cette classe par n'importe laquelle de ses sous-classes, 
comme `Circle` ou `Rectangle`, sans altérer le code.

> Remarque : Vous pouvez lire les [actes de la conférence](./pdf/62138.62141.pdf) à 
> partir du discours d'ouverture où Barbara Liskov a partagé ce principe pour la 
> première fois, ou vous pouvez regarder un [court extrait](https://www.youtube.com/watch?v=-Z-17h3jG0A) 
> d'une interview avec elle pour plus de contexte.

En pratique, ce principe consiste à faire en sorte que vos sous-classes se comportent 
comme leurs classes de base sans perturber les attentes de quiconque lorsqu'elles 
appellent les mêmes méthodes. Pour poursuivre avec des exemples liés aux formes, 
imaginons que vous ayez une classe Rectangle comme celle-ci :

```python
# shapes_lsp_v1.py

class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height
```

Dans `Rectangle`, vous avez fourni la méthode `.calculate_area()`, qui fonctionne 
avec les [attributs d'instance](https://realpython.com/python-classes/#instance-attributes) 
`.width` et `.height`.

Un carré étant un cas particulier de rectangle à côtés égaux, vous envisagez de 
dériver une classe `Square` de `Rectangle` afin de réutiliser le code. Ensuite, 
vous surchargez la méthode [setter](https://realpython.com/python-getter-setter/) des 
attributs `.width` et `.height` afin que lorsqu'un côté change, l'autre côté change également :

```python
# shapes_lsp_v2.py

class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, side):
        super().__init__(side, side)

    def __setattr__(self, key, value):
        super().__setattr__(key, value)
        if key in ("width", "height"):
            self.__dict__["width"] = value
            self.__dict__["height"] = value
```

Dans cet extrait de code, vous avez défini `Square` comme une sous-classe de `Rectangle`. 
Comme l'utilisateur peut s'y attendre, le [constructeur](https://realpython.com/python-class-constructor/) 
de la classe ne prend que le côté du carré comme argument. 
En interne, la méthode `.__init__()` initialise les attributs parents, 
`.width` et `.height`, avec l'argument `side`.

Vous avez également défini une [méthode spéciale](https://realpython.com/python-classes/#special-methods-and-protocols), 
`.__setattr__()`, pour se connecter au mécanisme de définition d'attributs de 
Python et intercepter l'[affectation](https://realpython.com/python-assignment-operator/) d'une 
nouvelle valeur à l'attribut `.width` ou `.height`. Plus précisément, lorsque 
vous définissez l'un de ces attributs, l'autre attribut prend également la même valeur :

```python
>>> from shapes_lsp import Square

>>> square = Square(5)
>>> vars(square)
{'width': 5, 'height': 5}

>>> square.width = 7
>>> vars(square)
{'width': 7, 'height': 7}

>>> square.height = 9
>>> vars(square)
{'width': 9, 'height': 9}
```

Vous avez maintenant assuré que l'objet `Square` reste toujours un carré valide, 
ce qui vous simplifie la vie pour un petit gain de mémoire. Malheureusement, cela 
viole le principe de substitution de Liskov, car il est impossible de remplacer 
les instances de `Rectangle` par leurs homologues `Square`.

Lorsqu'un utilisateur s'attend à un objet rectangle dans son code, il peut supposer 
qu'il se comportera comme tel en exposant deux attributs `.width` et `.height` 
indépendants. Or, votre classe `Square` contredit cette hypothèse en modifiant 
le comportement promis par l'interface de l'objet. Cela pourrait avoir des conséquences 
inattendues et indésirables, probablement difficiles à [déboguer](https://realpython.com/python-debugging-pdb/).

Bien qu'un carré soit un type spécifique de rectangle en mathématiques, les classes 
qui représentent ces formes ne doivent pas être dans une relation parent-enfant si 
vous souhaitez qu'elles respectent le principe de substitution de Liskov. Une 
solution consiste à créer une classe de base pour `Rectangle` et `Square` :

```python
# shapes_lsp_v3.py

from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def calculate_area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def calculate_area(self):
        return self.side ** 2
```

`Shape` devient le type que vous pouvez substituer par polymorphisme avec `Rectangle` ou `Square`, 
qui sont désormais frères et soeurs plutôt que parents et enfants. Notez que ces deux types de 
formes concrètes possèdent des ensembles d'attributs distincts, des méthodes d'initialisation 
différentes et pourraient potentiellement implémenter des comportements encore plus distincts. 
Leur seul point commun est la capacité à calculer leur aire.

Avec cette implémentation en place, vous pouvez utiliser le type `Shape` de manière interchangeable 
avec ses sous-types `Square` et `Rectangle` lorsque seul leur comportement commun vous intéresse :

```python
>>> from shapes_lsp import Rectangle, Square

>>> def get_total_area(shapes):
...     return sum(shape.calculate_area() for shape in shapes)

>>> get_total_area([Rectangle(10, 5), Square(5)])
75
```

Ici, vous passez une paire composée d'un rectangle et d'un carré à une fonction qui calcule leur 
aire totale. Comme la fonction ne prend en compte que la méthode `.calculate_area()`, les différences 
de forme importent peu. C'est l'essence même du principe de substitution de Liskov.
