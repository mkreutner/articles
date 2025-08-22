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

