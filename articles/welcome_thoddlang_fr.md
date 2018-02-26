# Bienvenue sur Thoddlang.org

Ceci est mon premier article de blog.

## Thoddlang c'est avant tout

Thoddlang (Thodd de son petit nom) est avant tout un langage de programmation par flux et fonctionnel qui vise à la simplicité et l'élégance du développement favorisant ainsi la maintenabilité et réutilisabilité du code.

Thodd permet de modéliser les traitements des flux de données de manière aussi pure fonctionnellement que possible. Les interactions avec les IO (socket, fichier, ...) sont cloisonnées dans certains types de fonctions (***reader, writer, listener***).

Quant à la modélisation des données, Thodd met à disposition la classique structure (*struct*) présent dans bon nombre de langage (c, c++, java, ...). Un ensemble de type dit natif est a disposition pour permettre la composition de ceux-ci dans des structures plus complexes. On parlera de **type axiomatique**.

De même qu'il existe des types axiomatiques, Thodd possède un ensemble de fonctions de base (les fonctions axiomatiques) qui permettent les opérations de base nécessaires au développement des programmes modernes (fonctions arithmétiques, booléennes, les ***reader|listener*** natifs (socket, file), les ***writer*** natifs, ...).

Enfin, contrairement à de nombreux langages fonctionnels purs, Thodd permet la construction des données en plusieurs étapes, ne respectant ainsi pas entièrement le principe d'*immutabilité* cher aux autres langages fonctionnel. Cette construction multiphase se fait au travers de fonctions dites ***builder***.

Enfin (encore ? :D), Thodd innove en permettant plusieurs *"main"* (plusieurs points d'entrée) au sein du même programme. Le mot ***main*** n'est donc pas un nom de fonction comme en C, mais un mot clé qui peut être apposé sur toute fonction dite ***listener*** ou bien ***reader***. On entrevoit ici la génération de plusieurs processus (un par ***main***) favorisant un développement de Micro(Nano ?)-services.

Voilà, ce qu'est Thodd.

## Thoddlang et les fonctions : une grande histoire d'amour

Nous l'évoquions dans la présentation de Thodd, il possède plusieurs types de fonctions :

* ***reader*** : lit une donnée dans le flux.
* ***listener*** : écoute **indéfiniment** les données dans le flux.
* ***processor*** : transforme / calcule des résultat à partir des données lues / écoutées depuis le flux.
* ***builder*** : construit les structures de données pour les ***processor***.
* ***writer*** : écrit le résultat d'un ***processor*** ou d'un ***reader***/***listener*** dans le flux.

### ***Reader***

Un ***reader*** est une fonction permettant d'extraire une donnée depuis un flux (socket, fichier, entrée standard, mémoire, ...) :

    reader read_socket(string ip, int flags): bytes ;

Il existe un ensemble de fonctions axiomatiques ***reader*** permettant d'intéragir avec les flux classiques.

    reader read_socket (string ip, int flags) : bytes | string | int | ... ;
    reader read_file (string name, int flags) : bytes | string | int | ... ;
    reader read_stdin (int flags)             : bytes | string | int | ... ;
    ...

A partir de ces fonctions axiomatiques on peut définir des fonctions plus pratiques pour un développeur moderne (qui aura la volonté d'intéragir avec une socket réseau ? :p ). La création de ces nouveaux ***reader*** sera vu en détail dans un autre article. A titre d'exemple :

    reader read_mysql_table (mysql_schema schema, string tablename) : table {
      ... // Définition du reader
    }

### ***Listener***

Les fonctions ***listener*** sont semblables aux ***reader***, mais à la différence de ces dernières, elles lisent en permanence le flux de données. Et quand il n'y a plus de donnée à lire, elles attendent qu'une nouvelle donnée soit écrite sur le flux par un ***writer*** pour aller la lire.

Il existe une fonction ***listener*** axiomatique equivalente pour chacun des ***reader*** axiomatique fourni par Thodd.

    listener listen_socket (string ip, int flags) : bytes | string | int | ... ;
    listener listen_file (string name, int flags) : bytes | string | int | ... ;
    listener listen_stdin (int flags)             : bytes | string | int | ... ;
    ...

### ***Processor***

Les fonctions ***processor*** réprésentent le scope fonctionnel pur de Thodd. Quand on est dans le contexte de ces fonctions, on ne peut faire appel qu'à d'autres fonctions ***processor*** ou bien ***builder***. Il s'agit d'une erreur de compilation que de faire appel à tout autre type de fonction. Les effets de bords sont ainsi évités au sein des ***processor*** et éliminant ainsi un ensemble de bugs classiques de la programmation.

OK, mais c'est quoi un ***processor*** ?

Un ***processor*** est une fonction **pure** au sens mathématique du terme : **n entrées, UNE sortie, PAS D'EFFET DE BORD**.

    processor increment (int nb, int pas) {
      return __add (nb, pas) ;
    }

### ***Builder***

Les fonctions ***builder***, probablement la famille de fonctions qui risque de faire grincer des dents les puristes des langages fonctionnels. Il s'agit de fonction permettant de construire une donnée en plusieurs étapes (plusieurs appel de fonction ***builder***), mais tout de même avec un contrôle pour garantir l'absence de tout effet de bord. En dehors d'un builder, la donnée construite sera immutable.

### ***Writer***

Comme son nom l'indique, un ***writer*** est un type de fonction permettant l'écriture de données dans un flux. Il existe un ***writer*** axiomatique par ***writer|listener*** axiomatique existant.

## Définition d'un flux

Un flux se compose de trois parties bien distinctes :

* un ***reader|listener***
* un ***processor*** (optionnel)
* un ***writer***

Thodd fournit une syntaxe particulière pour la déclaration d'un flux.

    flow address_csv_to_hash:
        addr = listener read_file(string filename, int flags): string [[flags = 0]];
        hash = processor address_to_hash(string addr): int [[addr = addr]] ;
        writer write_file (string filename, int hash) : int [[filename = "hash.csv", hash = hash]] ;

On peut combiner des flux entre eux en voyant un flux comme une source de donnée potentielle pour un autre flux. Dans notre exemple précédent nous avons déclarer le flux *address_csv_to_hash*. Celui ci peut peut être utilisé comme ***reader*** pour un autre flux. Le ***reader*** équivalent serait :

    listener address_csv_to_hash (string filename, int flags) : int ;

*address_csv_to_hash* est un ***listener*** car le flux sous-jacent commence par un ***listener***. La sortie est un ***int*** car la sortie du ***writer*** est ***int***. Quand aux arguments, il s'agit des arguments du ***reader*** :

* un string filename
* un int flags

Voici un exemple de chainage  :

    flow hash_to_string :
        hash = listener address_to_hash (string filename, int flags): int [[filename = "address.csv", flag = 0]] ;
        hash_str = processor int_to_string (int hash) : string [[hash = hash]] ;
        writer write_file (string filename, string hash): string [[filename = "hash_str.csv", hash = hash_str]] ;

La valorisation des arguments des flux sera vue dans un article dédié.

## Hello World

Voici, comme le veut la tradition, un exemple de type Hello World : 

    processor __concat (string l, string r) : string ;  // __concat est une fonction axiomatique

    processor concat_hello_and_name (string name) : string {
        return __concat ("hello ", name) ;
    }

    main flow hello_world :
        name = reader read_stdin (string name) : string ;
        message = processor concat_hello_and_name (string name) : string [[name = name]] ;
        writer write_hello_name_stdout (string message) : string [[message = message]] ;

Ainsi se fini mon premier article sur le langage Thoddlang.
Je remercie l'ensemble des personnes ayant participé à la relecture de cette publication.