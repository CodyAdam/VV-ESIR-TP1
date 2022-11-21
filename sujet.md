# Practical Session #1: Introduction

1. Find in news sources a general public article reporting the discovery of a software bug. Describe the bug. If possible, say whether the bug is local or global and describe the failure that manifested its presence. Explain the repercussions of the bug for clients/consumers and the company or entity behind the faulty program. Speculate whether, in your opinion, testing the right scenario would have helped to discover the fault.

2. Apache Commons projects are known for the quality of their code and development practices. They use dedicated issue tracking systems to discuss and follow the evolution of bugs and new features. The following link https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-794?filter=doneissues points to the issues considered as solved for the Apache Commons Collections project. Among those issues find one that corresponds to a bug that has been solved. Classify the bug as local or global. Explain the bug and the solution. Did the contributors of the project add new tests to ensure that the bug is detected if it reappears in the future?

3. Netflix is famous, among other things we love, for the popularization of *Chaos Engineering*, a fault-tolerance verification technique. The company has implemented protocols to test their entire system in production by simulating faults such as a server shutdown. During these experiments they evaluate the system's capabilities of delivering content under different conditions. The technique was described in [a paper](https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf) published in 2016. Read the paper and briefly explain what are the concrete experiments they perform, what are the requirements for these experiments, what are the variables they observe and what are the main results they obtained. Is Netflix the only company performing these experiments? Speculate how these experiments could be carried in other organizations in terms of the kind of experiment that could be performed and the system variables to observe during the experiments.

4. [WebAssembly](https://webassembly.org/) has become the fourth official language supported by web browsers. The language was born from a joint effort of the major players in the Web. Its creators presented their design decisions and the formal specification in [a scientific paper](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf) published in 2018. The goal of the language is to be a low level, safe and portable compilation target for the Web and other embedding environments. The authors say that it is the first industrial strength language designed with formal semantics from the start. This evidences the feasibility of constructive approaches in this area. Read the paper and explain what are the main advantages of having a formal specification for WebAssembly. In your opinion, does this mean that WebAssembly implementations should not be tested? 

5.  Shortly after the appearance of WebAssembly another paper proposed a mechanized specification of the language using Isabelle. The paper can be consulted here: https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf. This mechanized specification complements the first formalization attempt from the paper. According to the author of this second paper, what are the main advantages of the mechanized specification? Did it help improving the original formal specification of the language? What other artifacts were derived from this mechanized specification? How did the author verify the specification? Does this new specification removes the need for testing?

## Answers

### 1.

**Partial Cloudflare outage on October 25, 2022**

Source: https://blog.cloudflare.com/partial-cloudflare-outage-on-october-25-2022/

Afin d'améliorer ses performances, Cloudflare ajoute régulièrement du code de contrôle permettant
de visualiser les performances de certains services.

Une équipe a récemment ajouté un code de contrôle à la logique de Tiered Cache.
Pour ce faire, ils utilisent une fonction (A) qui englobe la fonction (B) à surveiller.

Cependant, cette implémentation a eu pour effet de modifier le comportement de la fonction (B).
En effet, la fonction (A) supprime les en-têtes de contrôle avant de les envoyer à la fonction (B).

Ces en-têtes sont attendus par le système de cache et peuvent être critique pour le routage correct de l'information.
De ce fait, certains clients ont reçu un code d'erreur 530 lors de requêtes vers des sites utilisant Cloudflare.

---

C'est un bug local, car il est lié à une erreur d'implémentation dans le code.

---

Les conséquences pour les clients sont que certains sites ne sont plus accessibles. Cloudflare estime que 5% des requêtes ont été impactées.

---

Je pense que le test de la bonne implémentation de la fonction (A) aurait permis de détecter le bug.
C'est d'ailleurs ce que Cloudflare a indiqué s'engager à faire dans le futur. Notamment en utilisant un plus large panel de configuration.
Internet comporte de nombreux protocoles plus ou moins libre et il est très difficile de faire un système qui régisse correctement à tous les cas.


### 2.

> Issue link : [586](https://issues.apache.org/jira/browse/COLLECTIONS-586)
>
> Title : PatriciaTrie prefixMap clear throws NullPointerException
>
> Pr link : [PR-18](https://github.com/apache/commons-collections/pull/18)
- **Contexte** :
  La classe `PatriciaTrie` est une structure de données qui permet de stocker et de récupérer des valeurs en fonction d'une clé. C'est une structure en forme d'arbre où chaque noeud est un caractère de la clé. Sa fonctionnalité est similaire à une **HashMap** mais elle est plus efficace lorsque les clés ont un préfixe commun.

- Description du problème** : Lorsque la méthode `clear()` est appelée sur une instance de la classe `PatriciaTrie`, elle lève une NullPointerException.

- Solution** : Le contributeur qui a soulevé le problème a proposé une solution de contournement. Au lieu d'appeler la méthode `clear()`, vous supprimez chaque clé de l'instance PatriciaTrie avec la méthode remove(Object key).

- **Description de la solution** : La méthode `clear()` de la classe PatriciaTrie était surchargée pour appeler la méthode `clear()` de la classe PatriciaTrie au lieu de la méthode `clear()` de la classe parente AbstractMap.

- **Tests** : Deux cas de tests ont été ajoutés à la suite de tests de la classe. Le bug ne devrait plus apparaître si les tests passent.

Ce type de bug est **local** car il est dû à une omission du développeur qui n'a pas surchargé la méthode `clear()` de la classe `PatriciaTrie` même si le type de données interne de la classe a changé.

Après la revue de la pull request, les tests ont été passés, la pull request a été merged et l'issue a été fermé.


1. Netflix et le Chaos Engineering

Source: https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf

Voici quelques exemples d'expériences réalisées par Netflix :
- Mettre fin aux instances de machines virtuelles.
- Injecter de la latence dans les requêtes entre services.
- Faire échouer les requêtes entre les services.
- Faire échouer un service interne
- Rendre indisponible une région entière AWS

---

Il faut commencer par définir un "état stable" du système.
C'est-à-dire un état dans lequel le système est fonctionnel et répond correctement aux requêtes.

On part maintenant du principe que cet état stable sera conservé pendant toute l'expérience.

Après avoir défini cet état stable, on peut commencer à injecter des perturbations dans le système. 
On peut ainsi observer les effets de ces perturbations sur le système.

Netflix utilise plusieurs métriques pour évaluer le système. C'est une plateforme de streaming video, 
donc les métriques sont principalement liées à la qualité de l'image et la stabilité du stream.
Ils utilisent par exemple le SPS (stream starts per second).

Les métriques plus précises comme le CPU ou la mémoire sont moins utilisés, car elles ne mesurent pas directement l'expérience utilisateur.

En observant le SPS, on peut voir que le système est utilisé le plus entre 15h et 23h.

---

Netflix n'est pas l'unique entreprise à utiliser le Chaos Engineering. 
On cite notamment LinkedIn, Facebook, Google, Microsoft, Amazon et bien d'autres.

---

Ces experiences sont plus utiles pour de grosses infrastructures avec de nombreux services.
C'est un moyen de tester le système dans des conditions extrêmes.
Les variables à observer sont très dépendante de l'application. 
Il faut avant tout déterminer l'utilisation principale du système.


### 4. 
Les principaux avantages d'avoir une spécification formelle pour WebAssembly sont qu'il est "déterministe et facile à raisonner autour" ; cela rend le langage plus sûr et plus facile à mettre en œuvre en raison de moins d'ambiguïté étant donné qu'il est plus opinionné.
Cela ne signifie pas que les implémentations ne doivent pas être testées, mais qu'il peut être plus facile de trouver les bogues introduits dans les implémentations qui ne sont pas conformes à la spécification.

### 5.

L'article décrit explicitement comment tester l'implémentation pour qu'elle soit conforme aux spécifications de l'article précédent en utilisant des preuves. Cela signifie que les utilisateurs doivent toujours tester leur implémentation pour s'assurer qu'elle est correcte, mais qu'il n'est plus nécessaire de tester la spécification.