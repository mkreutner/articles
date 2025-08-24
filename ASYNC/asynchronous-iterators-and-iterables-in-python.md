# Asynchronous Iterators and Iterables in Python

Lorsque vous écrivez du code asynchrone en Python, vous aurez probablement besoin de créer 
des itérateurs et des itérables asynchrones à un moment ou à un autre. Les itérateurs asynchrones 
permettent à Python de contrôler les boucles `for` asynchrones, tandis que les itérables asynchrones 
sont des objets sur lesquels vous pouvez itérer à l'aide de boucles `for` asynchrones.

Ces deux outils vous permettent d'itérer sur des objets attendus sans bloquer votre code. 
Vous pouvez ainsi effectuer différentes tâches de manière asynchrone.

**Dans ce tutoriel, vous allez** :

* Découvrir ce que sont les **itérateurs** et les **itérables** **asynchrones** en Python
* Créer des **générateurs** d'**expressions** et d'**itérateurs** asynchrones
* Coder des itérateurs et des itérables asynchrones avec les méthodes `.__aiter__()` et `.__anext__()`
* Utiliser des itérateurs dans les boucles asynchrones et les compréhensions

Pour tirer le meilleur parti de ce tutoriel, vous devez avoir des bases sur les 
[itérateurs et les itérables en Python](https://realpython.com/python-iterators-iterables/). 
Vous devez également connaître les [fonctionnalités](https://realpython.com/python-async-features/) et 
les [outils](https://realpython.com/async-io-python/) asynchrones de Python.

## Découvrir les itérateurs et les itérables asynchrones en Python

Les itérateurs et les itérables sont des composants fondamentaux de Python. Vous les utiliserez 
dans presque tous vos programmes où vous itérez sur des flux de données à l'aide d'une 
[boucle `for`](https://realpython.com/python-for-loop/). Les itérateurs pilotent et contrôlent 
le processus d'itération, tandis que les itérables contiennent généralement les données sur 
lesquelles vous souhaitez itérer.

Les itérateurs Python implémentent le [modèle de conception itérateur](https://en.wikipedia.org/wiki/Iterator_pattern), 
qui permet de parcourir un conteneur et d'accéder à ses éléments. Pour implémenter ce modèle, 
les itérateurs ont besoin des [méthodes spéciales](https://realpython.com/python-magic-methods/) 
`.__iter__()` et `.__next__()`. De même, les itérables sont généralement des conteneurs 
de données qui implémentent la méthode `.__iter__()`.

> Remarque : pour approfondir les itérateurs et les itérables, consultez le didacticiel 
> [Itérateurs et itérables en Python : exécuter des itérations efficaces](https://realpython.com/python-iterators-iterables/).

Python a étendu le concept d'itérateurs et d'itérables à la **programmation asynchrone** grâce 
au module [`asyncio`](https://realpython.com/async-io-python/) et aux mots-clés 
[`async` et `await`](https://realpython.com/python-keywords/#asynchronous-programming-keywords-async-await). 
Dans ce scénario, les itérateurs asynchrones pilotent le **processus d'itération asynchrone**, principalement 
alimenté par des boucles for asynchrones et des [compréhensions](https://realpython.com/python-async-iterators/#asynchronous-comprehensions-and-generator-expressions).

> Remarque : Ce tutoriel ne vous permettra pas d'explorer les subtilités de la programmation 
> asynchrone en Python. Il est donc conseillé de bien connaître les concepts associés. 
> Si ce n'est pas le cas, vous pouvez consulter les tutoriels suivants :
> 
> * [Démarrer avec les fonctionnalités asynchrones en Python](https://realpython.com/python-async-features/)
> * [E/S asynchrones en Python : Présentation complète](https://realpython.com/async-io-python/)
>
> Ces tutoriels vous permettront d'acquérir les connaissances nécessaires pour explorer 
> plus en profondeur les itérateurs et les itérables asynchrones.

Dans les sections suivantes, vous examinerez brièvement les concepts d’itérateurs asynchrones 
et d’itérables en Python.

### Itérateurs asynchrones

La documentation de Python définit les **itérateurs asynchrones**, ou *async iterators* en abrégé, 
comme suit :

> An object that implements the `.__aiter__()` and `.__anext__()` [special] methods.
> `.__anext__()` must return an [awaitable](https://docs.python.org/3/glossary.html#term-awaitable) object. 
> [An] async for [loop] resolves the awaitables returned by an asynchronous iterator’s `.__anext__()` 
> method until it raises a `StopAsyncIteration` exception.
> ([Source](https://docs.python.org/3/glossary.html#term-asynchronous-iterator))

> Objet implémentant les méthodes spéciales `.__aiter__()` et `.__anext__()`. `.__anext__()` 
> doit renvoyer un objet [awaitable](https://docs.python.org/3/glossary.html#term-awaitable). 
> Une boucle `for` asynchrone résout les awaitables renvoyés par la méthode `.__anext__()`
> d'un itérateur asynchrone jusqu'à ce qu'elle  génère une exception `StopAsyncIteration`.
> ([Source](https://docs.python.org/3/glossary.html#term-asynchronous-iterator))

Tout comme les itérateurs classiques qui doivent implémenter `.__iter__()` et `.__next__()`, 
les itérateurs asynchrones doivent implémenter `.__aiter__()` et `.__anext__().` Dans les itérateurs 
classiques, la méthode `.__iter__()` renvoie généralement l'itérateur lui-même. Ceci est 
également vrai pour les itérateurs asynchrones.

Pour maintenir ce parallélisme, dans les itérateurs classiques, la méthode `.__next__()` 
doit renvoyer l'objet suivant de l'itération. Dans les itérateurs asynchrones, 
la méthode `.__anext__()` doit renvoyer l'objet suivant, qui doit être attendu.

Python définit les objets attendus comme décrit dans la citation ci-dessous :

> An object that can be used in an await expression. [It] can be a 
> [coroutine](https://docs.python.org/3/glossary.html#term-coroutine) 
> or an object with an `.__await__()` method. 
> ([Source](https://docs.python.org/3/glossary.html#term-awaitable))

> Un objet utilisable dans une expression d'attente. Il peut s'agir d'une 
> [coroutine](https://docs.python.org/3/glossary.html#term-coroutine) 
> ou d'un objet doté d'une méthode `.__await__()`. 
> ([Source](https://docs.python.org/3/glossary.html#term-awaitable))

En pratique, un moyen rapide de créer un objet waitable en Python consiste à appeler 
une fonction asynchrone. Ce type de fonction est défini avec le mot-clé 
[`async def`](https://docs.python.org/3/reference/compound_stmts.html#async-def). 
Cet appel crée un **objet coroutine**.

> Remarque : Vous pouvez également créer des objets waitable en implémentant la 
> méthode spéciale `.__await__()` dans une classe personnalisée. Cette méthode 
> doit renvoyer un [itérateur](https://docs.python.org/3/glossary.html#term-iterator) 
> qui redonne le contrôle à la [boucle d'événements](https://docs.python.org/3/library/asyncio-eventloop.html) 
> jusqu'à ce que le résultat attendu soit prêt. Ce sujet dépasse le cadre de ce tutoriel.

Lorsque le flux de données est épuisé, la méthode doit lever une exception 
`StopAsyncIteration` pour mettre fin au processus d'itération asynchrone.

Voici un exemple d'itérateur asynchrone permettant d'itérer sur une plage de 
nombres de manière asynchrone :

```python
# async_range_v1.py

import asyncio

class AsyncRange:
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def __aiter__(self):
        return self

    async def __anext__(self):
        if self.start < self.end:
            await asyncio.sleep(0.5)
            value = self.start
            self.start += 1
            return value
        else:
            raise StopAsyncIteration

async def main():
    async for i in AsyncRange(0, 5):
        print(i)

asyncio.run(main())
```

Dans la méthode `.__aiter__()`, vous renvoyez `self`, qui est l'objet courant 
(l'itérateur lui-même). Dans la méthode `.__anext__()`, vous générez un nombre 
compris entre `.start` et `.end`.

> Remarque : Dans les itérateurs asynchrones, la méthode `.__aiter__()` doit être 
> [une méthode d'instance](https://realpython.com/python-classes/#instance-methods-with-self) 
> standard, tandis que la méthode `.__anext__()` doit être une méthode asynchrone 
> définie avec le mot-clé `async def`. Vous en apprendrez plus sur ces particularités 
> dans la section "**Création d'itérateurs et d'itérables asynchrones basés sur des classes**".

Pour simuler l'objet attendu requis, utilisez la fonction `asyncio.sleep()` avec un délai 
de 0.5 seconde dans une instruction await. Une fois la plage couverte, une exception 
`StopAsyncIteration` est levée pour terminer l'itération ([raise](https://realpython.com/python-raise-exception/)).

L'[exécution](https://realpython.com/run-python-scripts/) du script produit le résultat suivant :

```shell
$ python async_range_v1.py
0
1
2
3
4
```

Dans cet exemple, vous obtiendrez chaque nombre après une demi-seconde d'attente, ce qui 
correspond à l'itération asynchrone.

L'exemple ci-dessus est un bref aperçu des itérateurs asynchrones et de leur définition. 
Vous en apprendrez plus sur les méthodes `.__aiter__()` et `.__anext__()` et les concepts 
associés dans les sections sur la **création d'itérateurs asynchrones**. Il est maintenant 
temps d'apprendre les bases des itérables asynchrones.

### Itérables asynchrones

Concernant les **itérables asynchrones** (**async iterables**), la documentation Python 
indique ce qui suit :

> An object, that can be used in an `async` for statement. Must return an 
> [asynchronous iterator](https://docs.python.org/3/glossary.html#term-asynchronous-iterator) 
> from its `.__aiter__()` method. 
> ([Source](https://docs.python.org/3/glossary.html#term-asynchronous-iterable))

> Un objet, utilisable dans une instruction for asynchrone, doit renvoyer un 
> [itérateur asynchrone](https://docs.python.org/3/glossary.html#term-asynchronous-iterator) 
> depuis sa méthode `.__aiter__()`.
> ([Source](https://docs.python.org/3/glossary.html#term-asynchronous-iterable))

En pratique, un objet n'a besoin que d'une méthode `.__aiter__()` renvoyant un itérateur 
asynchrone pour être itérable. Il n'a pas besoin de la méthode `.__anext__()`.

Voici comment modifier `AsyncRange` pour qu'il devienne un itérable asynchrone plutôt 
qu'un itérateur :

```python
# async_range_v2.py

import asyncio

class AsyncRange:
    def __init__(self, start, end):
        self.data = range(start, end)

    async def __aiter__(self):
        for i in self.data:
            await asyncio.sleep(0.5)
            yield i

async def main():
    async for i in AsyncRange(0, 10):
        print(i)

asyncio.run(main())
```

Cette nouvelle implémentation de votre classe `AsyncRange` est plus concise 
que la précédente. Elle a simplement la méthode `.__aiter__()`, qui génère 
des nombres à la demande. Là encore, vous utilisez la fonction `asyncio.sleep()` 
pour simuler l'objet attendu.

> Remarque : La méthode `.__aiter__()` de l'exemple ci-dessus est un itérateur 
générateur asynchrone défini avec l'instruction `yield`. Vous en apprendrez plus 
sur ce type d'objet dans la section "**Création de fonctions génératrices asynchrones**".

Voici le fonctionnement de la classe :

```shell
$ python async_range_v2.py
0
1
2
3
4
```

Vous obtenez le même résultat que dans la section précédente, mais au lieu d'utiliser 
un itérateur asynchrone, vous utilisez un itérable.

Avec ce bref aperçu des itérateurs et itérables asynchrones, vous pouvez maintenant 
approfondir le fonctionnement de l'itération asynchrone et les raisons de son utilisation 
dans votre code.

### Itération asynchrone

En Python, l'**itération asynchrone** consiste à parcourir des itérables asynchrones à 
l'aide de **boucles for asynchrones**. En pratique, les boucles for asynchrones s'appuient 
sur des itérateurs asynchrones pour contrôler le processus d'itération. 
L'itération asynchrone permet d'effectuer des **opérations non bloquantes** au sein de la 
boucle.

> Remarque : Les opérations bloquantes bloquent l'exécution du code suivant jusqu'à leur 
> achèvement. Les opérations non bloquantes ne bloquent pas l'exécution du code suivant. 
> Elles permettent plutôt l'exécution d'autres opérations en attendant la fin des tâches 
> chronophages.

L'itération asynchrone permet de gérer efficacement les tâches liées aux 
[E/S](https://en.wikipedia.org/wiki/I/O_bound) et de les exécuter simultanément. Parmi 
les tâches courantes liées aux E/S, on trouve :

* **Opérations système de fichiers**, telles que la 
[lecture et l'écriture de fichiers](https://realpython.com/read-write-files-python/), 
ainsi que l'accès à leurs métadonnées (taille, date de création et de modification).
* **Opérations réseau**, telles que les [requêtes HTTP](https://realpython.com/urllib-request/) 
et la [communication par socket](https://realpython.com/python-sockets/).
* **Opérations de base de données**, telles que l'exécution de requêtes SQL pour les 
opérations [CRUD](https://realpython.com/crud-operations/) (création, lecture, mise à 
jour et suppression).
* **Opérations d'entrée et de sortie utilisateur**, telles que la lecture des entrées 
utilisateur au clavier, à la souris ou sur tout autre périphérique et l'affichage à 
l'écran. Ces opérations sont essentielles dans les 
[applications d'interface utilisateur graphique (GUI)](https://realpython.com/python-pyqt-gui-calculator/), 
où le rendu de l'interface peut être gourmand en ressources.
* **Communication avec des périphériques externes**, comme l'interaction avec des capteurs 
externes, des imprimantes ou d'autres périphériques connectés à des ports série ou 
parallèle.

En Python, le code asynchrone s'exécute dans une [boucle d'événements](https://docs.python.org/3/library/asyncio-eventloop.html), 
généralement lancée par la fonction [`asyncio.run()`](https://docs.python.org/3/library/asyncio-runner.html#asyncio.run).

Lorsque vous itérez sur un itérable asynchrone à l'aide d'une boucle for asynchrone, 
la boucle redonne le contrôle à la boucle d'événements après chaque cycle afin que 
d'autres tâches asynchrones puissent s'exécuter. Ce type d'itération est non bloquant, 
car il ne bloque pas l'exécution de l'application pendant l'exécution de la boucle.

Les itérateurs et itérables asynchrones permettent l'itération asynchrone, ce qui 
permet d'exécuter des tâches simultanément.

[La concurrence](https://realpython.com/python-concurrency/) permet à plusieurs tâches 
de progresser en partageant le temps sur le même coeur de processeur ou de s'exécuter en 
parallèle sur plusieurs coeurs. Cette technique de programmation peut vous aider à 
optimiser votre code. Elle vous permet également d'éviter de bloquer l'exécution de 
votre programme avec des tâches chronophages comme celles mentionnées ci-dessus.

La programmation asynchrone est un type spécifique de concurrence basé sur des 
**opérations non bloquantes** et une **exécution pilotée par événements**. 
C'est pourquoi le code asynchrone s'exécute dans une boucle d'événements principale, 
qui gère les événements asynchrones.

Dans votre aventure de programmation asynchrone en Python, vous serez probablement 
amené à créer vos propres itérateurs et itérables asynchrones. En pratique, la méthode 
privilégiée consiste à utiliser des **itérateurs générateurs asynchrones**, ce qui 
est le sujet de la section suivante.

## Création de fonctions de générateur asynchrone

Dans la documentation Python, un **générateur asynchrone** est défini comme suit :


> A function which returns an [asynchronous generator iterator](https://docs.python.org/3/glossary.html#term-asynchronous-generator-iterator). 
> It looks like a coroutine function defined with [`async def`](https://docs.python.org/3/reference/compound_stmts.html#async-def) 
> except that it contains yield expressions for producing a series of values usable in an async `for` loop.
> ([Source](https://docs.python.org/3/glossary.html#term-asynchronous-generator))

> Une fonction qui renvoie un **itérateur de [générateur asynchrone](https://docs.python.org/3/glossary.html#term-asynchronous-generator-iterator). 
> Elle ressemble à une fonction coroutine définie avec [`async def`](https://docs.python.org/3/reference/compound_stmts.html#async-def), 
> sauf qu'elle contient des expressions `yield` pour produire une série de valeurs utilisables dans une boucle `for` asynchrone. 
> ([Source](https://docs.python.org/3/glossary.html#term-asynchronous-generator))

Pour une illustration rapide d'un itérateur de générateur, considérez la modification 
suivante de votre itérateur de plage asynchrone :

```python
# async_range_v3.py

import asyncio

async def async_range(start, end):
    for i in range(start, end):
        await asyncio.sleep(0.5)
        yield i

async def main():
    async for i in async_range(0, 5):
        print(i)

asyncio.run(main())
```

Un générateur asynchrone est une fonction coroutine définie à l'aide du mot-clé `async def`. 
Cette fonction doit comporter une instruction `yield` pour générer des objets waitables à 
la demande. Dans cet exemple, vous simulez l'objet waitable avec `asyncio.sleep()`, comme 
précédemment.

Les fonctions de générateur asynchrone peuvent contenir des expressions [`await`](https://docs.python.org/3/reference/expressions.html#await), 
des boucles `for` async et des instructions [with async](https://docs.python.org/3/reference/compound_stmts.html#async-with). 
Ce type de fonction renvoie un itérateur générateur asynchrone qui génère des éléments à la demande.

Pour un exemple plus détaillé, imaginons que vous souhaitiez créer un script pour sauvegarder 
les fichiers d'un répertoire donné. Ce script traitera les fichiers de manière asynchrone et 
générera un [fichier ZIP](https://realpython.com/python-zipfile/) avec leur contenu.

Voici une implémentation possible de votre script de sauvegarde. Pour que le script fonctionne, 
vous devez installer les packages `aiofiles` et `aiozipstream` depuis PyPI à l'aide de `pip` 
et de la commande suivante :

```shell
$ python -m pip install aiofiles aiozipstream
```

Vous pouvez utiliser egalement [`uv`](https://docs.astral.sh/uv/)

```shell
# utiliser la configuration du project actuelle
$ uv add --requirements
# installer depuis un nouveau projet
$ uv init
$ uv add aiofiles aiozipstream
```

Maintenant que vous avez installé les dépendances externes, vous pouvez jeter un oeil au code :

```python
# compress.py

import asyncio
from pathlib import Path

import aiofiles
from zipstream import AioZipStream

async def stream_generator(files):
    async_zipstream = AioZipStream(files)
    async for chunk in async_zipstream.stream():
        yield chunk

async def main(directory, zip_name="output.zip"):
    files = [
        {"file": path}
        for path in directory.iterdir()
        if path.is_file()
    ]
    async with aiofiles.open(zip_name, mode="wb") as archive:
        async for chunk in stream_generator(files):
            await archive.write(chunk)

directory = Path.cwd()
asyncio.run(main(directory))
```

Dans cet exemple, vous importez d'abord les modules et classes requis. Ensuite, à la ligne 7, vous 
définissez une fonction de génération asynchrone. Cette fonction prend une liste de fichiers en argument. 
Les éléments de cette liste doivent être des dictionnaires dont la clé "`file`" correspond au chemin 
d'accès du fichier. À la ligne 8, vous créez un `AioZipStream` en utilisant la liste de fichiers en argument.

À la ligne 9, vous démarrez une boucle `for` asynchrone sur le flux de données compressées. Par défaut, 
la méthode `.stream()` renvoie les données compressées sous forme de blocs de 1024 octets maximum. 
À la ligne 10, vous produisez des blocs de données à la demande avec l'instruction `yield`. Cette 
instruction transforme la fonction en générateur asynchrone.

À la ligne 22, vous créez la variable directory pour contenir le répertoire cible. Dans cet exemple, 
vous utilisez `Path.cwd()` qui vous donne le répertoire de travail actuel. Autrement dit, le répertoire 
par défaut est le dossier d'exécution de votre script. Enfin, vous exécutez la boucle d'événements. 
Si vous exécutez ce script en ligne de commande, vous obtiendrez une archive ZIP contenant les fichiers 
du répertoire du script.

En pratique, l'utilisation de fonctions de génération asynchrone comme celles présentées dans cette 
section est l'approche la plus rapide et la plus recommandée pour créer des itérateurs asynchrones en 
Python. Cependant, si vos itérateurs doivent conserver un état interne, vous pouvez utiliser des itérateurs 
asynchrones basés sur des classes.

