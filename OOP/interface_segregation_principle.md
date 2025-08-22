## Interface Segregation Principle (ISP)

Le **principe de ségrégation des interfaces (ISP)** est issu du même esprit que le principe 
de responsabilité unique. Oui, c'est un atout supplémentaire. L'idée principale de ce principe 
est la suivante :

> Les clients ne devraient pas être contraints de dépendre de méthodes qu'ils n'utilisent pas. 
> Les interfaces appartiennent aux clients, et non aux hiérarchies.

Dans ce cas, les *clients* sont des classes et des *sous-classes*, et les *interfaces* sont 
constituées de méthodes et d'attributs. Autrement dit, si une classe n'utilise pas de méthodes 
ou d'attributs particuliers, ces méthodes et attributs doivent être séparés en classes plus 
spécifiques.

Prenons l'exemple suivant de [hiérarchie](https://realpython.com/python-classes/#class-hierarchies) 
de classes pour modéliser des imprimantes :

```python
# printers_isp_v1.py

from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

    @abstractmethod
    def fax(self, document):
        pass

    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

    def fax(self, document):
        raise NotImplementedError("Fax functionality not supported")

    def scan(self, document):
        raise NotImplementedError("Scan functionality not supported")

class ModernPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```

Dans cet exemple, la classe de base `Printer` fournit l'interface que ses sous-classes doivent implémenter. 
`OldPrinter` hérite de `Printer` et doit implémenter la même interface. Cependant, `OldPrinter` n'utilise 
pas les méthodes `.fax()` et `.scan()`, car ce type d'imprimante ne prend pas en charge ces fonctionnalités.

Cette implémentation viole l'ISP, car elle force `OldPrinter` à exposer une interface que la classe 
n'implémente pas ou dont elle n'a pas besoin. Pour résoudre ce problème, vous devez séparer les interfaces 
en classes plus petites et plus spécifiques. Vous pouvez ensuite créer des classes concrètes en héritant de 
plusieurs classes d'interface, selon vos besoins :

```python
# printers_isp_v2.py

from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

class Fax(ABC):
    @abstractmethod
    def fax(self, document):
        pass

class Scanner(ABC):
    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

class NewPrinter(Printer, Fax, Scanner):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```

Désormais, les classes `Printer`, `Fax` et `Scanner` sont des classes de base fournissant des 
interfaces spécifiques, chacune avec une responsabilité unique. Pour créer `OldPrinter`, vous 
héritez uniquement de l'interface `Printer`. Ainsi, la classe ne contient aucune méthode 
inutilisée. Pour créer la classe `ModernPrinter`, vous devez hériter de toutes les interfaces. 
En résumé, vous avez séparé l'interface `Printer`.

Cette conception de classe vous permet de créer différentes machines avec différents ensembles 
de fonctionnalités, ce qui rend votre conception plus flexible et extensible.
