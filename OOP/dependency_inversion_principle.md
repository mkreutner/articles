## Dependency Inversion Principle (DIP)

Le **principe d'inversion de dépendance (DIP)** est le dernier principe de 
l'ensemble SOLID. Ce principe stipule que :

> Les abstractions ne doivent pas dépendre de détails. 
> Les détails doivent dépendre d'abstractions.

Cela paraît complexe. Voici un exemple pour clarifier les choses. Imaginons 
que vous développiez une application et que vous disposiez d'une classe `FrontEnd` 
pour afficher les données de manière conviviale. L'application récupère actuellement 
ses données d'une base de données, ce qui vous donne le code suivant :

```python
# app_dip_v1.py

class FrontEnd:
    def __init__(self, back_end):
        self.back_end = back_end

    def display_data(self):
        data = self.back_end.get_data_from_database()
        print("Display data:", data)

class BackEnd:
    def get_data_from_database(self):
        return "Data from the database"
```

Dans cet exemple, la classe `FrontEnd` dépend de la classe `BackEnd` et de son 
implémentation concrète. On peut dire que les deux classes sont étroitement couplées. 
Ce couplage peut entraîner des problèmes d'évolutivité. Par exemple, si votre 
application connaît une croissance rapide et que vous souhaitez qu'elle puisse 
lire des données depuis une [API REST](https://realpython.com/api-integration-in-python/), 
comment procéderiez-vous ?

Vous pourriez envisager d'ajouter une nouvelle méthode à `BackEnd` pour récupérer les 
données depuis l'API REST. Cependant, cela nécessiterait également de modifier `FrontEnd`, 
qui doit être fermé à toute modification, conformément au principe 
[ouvert-fermé](./open-closed_principle.md).

Pour résoudre ce problème, vous pouvez appliquer le principe d'inversion de dépendances 
et faire en sorte que vos classes dépendent d'abstractions plutôt que d'implémentations 
concrètes comme `BackEnd`. Dans cet exemple précis, vous pouvez introduire une classe 
`DataSource` qui fournit l'interface à utiliser dans vos classes concrètes :

```python
# app_dip_v2.py

from abc import ABC, abstractmethod

class FrontEnd:
    def __init__(self, data_source):
        self.data_source = data_source

    def display_data(self):
        data = self.data_source.get_data()
        print("Display data:", data)

class DataSource(ABC):
    @abstractmethod
    def get_data(self):
        pass

class Database(DataSource):
    def get_data(self):
        return "Data from the database"

class API(DataSource):
    def get_data(self):
        return "Data from the API"
```

Lors de cette refonte de vos classes, vous avez ajouté une classe `DataSource` comme abstraction 
fournissant l'interface requise, ou la méthode `.get_data()`. Notez que `FrontEnd` dépend désormais 
de l'interface fournie par `DataSource`, qui est une abstraction.

Vous définissez ensuite la classe `Database`, une implémentation concrète pour les cas où vous 
souhaitez récupérer les données de votre base de données. Cette classe dépend de l'abstraction 
`DataSource` par héritage. Enfin, vous définissez la classe `API` pour prendre en charge la 
récupération des données depuis l'API REST. Cette classe dépend également de l'abstraction `DataSource`.

Voici comment utiliser la classe `FrontEnd` dans votre code :

```python
>>> from app_dip import API, Database, FrontEnd

>>> db_front_end = FrontEnd(Database())
>>> db_front_end.display_data()
Display data: Data from the database

>>> api_front_end = FrontEnd(API())
>>> api_front_end.display_data()
Display data: Data from the API
```

Ici, vous initialisez d'abord `FrontEnd` avec un objet `Database`, puis à nouveau avec un objet `API`. 
Chaque appel à `.display_data()` dépend de la source de données concrète utilisée. Notez que vous 
pouvez également modifier dynamiquement la source de données en réaffectant l'attribut `.data_source` 
dans votre instance `FrontEnd`.
