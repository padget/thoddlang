# Bienvenue sur Thoddlang.org

Ceci est mon premier article de blog.

## Thoddlang c'est avant tout

Thoddlang est avant un langage de programmation fonctionnel qui vise à
la simplicité et l'élégance du développement favorisant ainsi
la maintenabilité et réutilisabilité du code.

## Les bases

Dans thoddlang on a accès à deux principes fondamentaux :

* les structures (POD : plain of data)
* les fonctions

De plus, on retrouve l'immutabilité traditionnelle des langages de programmation fonctionnelle
garantissant la transparence référentielle (elle sera expliquée en détail dans un prochain article).

### Les structures (alias POD)

Thoddlang définit des types de bases sans lesquels il serait bien difficile de développer un programme.
On retrouve donc les int, float, char, string et autres types classiques des langages de programmation.

Thoddlang laisse aussi la possibilité de composer des nouveaux types (POD) à partir des types déjà défini.
C'est par les structures que cette composition peut se faire.

### Les fonctions

Thoddlang définit deux sortes de fonctions :

* fonctions pures
* fonctions IO

Les fonctions pures sont les fonctions ne permettant aucune altération d'état (aucun effet de bord). Pour un ensemble donné
de paramètres en entrée, la sortie sera toujours identique quelque soit le nombre d'appels de la fonction
(lié à la notion de transparence référentielle).

Les fonction IO sont des fonctions qui intéragissent avec le monde extérieur (extérieur au programme),
à savoir les fichiers, les sockets, le web, la sortie standard ...

## Gestion de la mémoire

La gestion de la mémoire est semi-automatique dans Thoddlang. La durée de vie est décidé par la développeur
au moment de la composition des POD, en revanche la destruction même des objets est assurée automatiquement par
Thoddlang. Vous décidez quand et il le fait pour vous.

Thoddlang met à disposition trois type de durée de vie :

* Valeur (lié au contexte appelant)
* Référence (aucune destruction programmée)
* Composant (lié à la durée de vie de possesseur)