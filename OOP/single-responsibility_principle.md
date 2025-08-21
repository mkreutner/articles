## Single-Responsibility Principle (SRP)

Le **principe de responsabilité unique (SRP)** vient de 
*Robert C. Martin*, plus connu sous le surnom d'*Oncle Bob*, figure 
respectée du monde de l'ingénierie logicielle et l'un des premiers signataires 
du Manifeste Agile. Il est d'ailleurs l'inventeur du terme SOLID.

Le principe de responsabilité unique stipule que :

> Une classe ne doit avoir qu'une seule raison de changer.

Cela signifie qu'une classe ne doit avoir qu'une seule **responsabilité**, 
exprimée par ses méthodes. Si une classe gère plusieurs tâches, il est 
conseillé de les séparer en classes distinctes.

> Remarque : Vous trouverez les principes SOLID formulés de différentes 
> manières. Dans ce tutoriel, vous les utiliserez selon la formulation 
> utilisée par Oncle Bob dans son livre "*[Agile Software Development: Principles, Patterns, and Practices](https://www.amazon.fr/dp/0131857258/?tag=devdetailpa0e-21)*". 
> Toutes les citations proviennent donc de ce livre.
>
> Pour un bref aperçu de ces principes et d'autres principes 
> connexes, consultez "*[The Principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)*" de l'*Oncle Bob*.

Ce principe est étroitement lié au [concept de séparation des tâches](https://en.wikipedia.org/wiki/Separation_of_concerns), 
qui suggère de diviser vos programmes en différentes sections. Chaque section 
doit répondre à une tâche distincte.

Pour illustrer le principe de responsabilité unique et son utilité pour 
améliorer votre conception orientée objet, imaginons que vous possédiez 
la classe `FileManager` suivante :

```Python
# file_manager_srp_v1.py

from pathlib import Path
from zipfile import ZipFile

class FileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def read(self, encoding="utf-8"):
        return self.path.read_text(encoding)

    def write(self, data, encoding="utf-8"):
        self.path.write_text(data, encoding)

    def compress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="w") as archive:
            archive.write(self.path)

    def decompress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="r") as archive:
            archive.extractall()
```

Dans cet exemple, votre classe `FileManager` a deux responsabilités distinctes. 
Elle utilise les méthodes `.read()` et `.write()` pour gérer le fichier. Elle 
gère également les archives `ZIP` grâce aux méthodes `.compress()` et `.decompress()`.

Cette classe **viole** le **principe de responsabilité unique**, car elle modifie 
son implémentation interne pour deux raisons. Pour corriger ce problème et 
renforcer votre conception, vous pouvez la scinder en deux classes plus 
petites et plus ciblées, chacune ayant sa propre responsabilité :

```python
# file_manager_srp_v2.py

from pathlib import Path
from zipfile import ZipFile

class FileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def read(self, encoding="utf-8"):
        return self.path.read_text(encoding)

    def write(self, data, encoding="utf-8"):
        self.path.write_text(data, encoding)

class ZipFileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def compress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="w") as archive:
            archive.write(self.path)

    def decompress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="r") as archive:
            archive.extractall()
```

Vous disposez désormais de deux classes plus petites, chacune ayant une seule 
responsabilité. `FileManager` gère un fichier, tandis que `ZipFileManager` gère 
la [compression](https://realpython.com/python-zipfile/#compressing-files-and-directories) et 
la [décompression](https://realpython.com/python-zipfile/#extracting-member-files-from-your-zip-archives) 
d'un fichier au format ZIP. Ces deux classes sont plus petites, donc plus 
faciles à gérer. Elles sont également plus faciles à analyser, à tester et à 
déboguer.

Dans ce contexte, le concept de **responsabilité** peut être assez subjectif. 
Avoir une seule responsabilité ne signifie pas nécessairement avoir une seule 
méthode. La responsabilité n'est pas directement liée au nombre de méthodes, 
mais à la tâche principale dont votre classe est responsable, selon votre 
conception de ce que la classe représente dans votre code. Cependant, cette 
subjectivité ne doit pas vous empêcher d'utiliser la SRP.
