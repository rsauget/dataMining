# TP Data Mining - Flickr
Projet de Data Mining pour la découverte et la caractérisation de points d'intérêts dans la ville de Lyon, à partir de données Flickr.
Projet réalisé sur KNIME dans le cadre de mes études à l'INSA Lyon.

## Filtrage

Le méta-noeud _Filtrage_ permet de conserver seulement les photos pertinentes, en retirant :
- les photos sans tags,
- les doublons possédant le même id,
- les photos dont la date d'envoi est antérieure à la date de prise de vue,
- les photos prises hors de la ville de Lyon.

Un _Column Filter_ permet ensuite de conserver seulement les colonnes id, user, les coordonnées GPS et les tags.

## Clustering hiérarchique

Le bloc _Row Sampling_ permet de conserver seulement 1000 photos aléatoires du jeu de données initial, pour assurer un temps d'exécution acceptable. Le clustering hiérarchique est ensuite effectué en _Complete link_ et génère 40 clusters, qui sont ensuite affichés avec des couleurs différentes sur une carte.

## k-Means
Le méta-noeud _Test de Stabilité_ exécute 100 clusterings k-Means avec des centroïdes initiaux différents à chaque exécution, et compare le résultat avec un k-Means de référence via un _Entropy Scorer_. Le méta-noeud renvoie le résultat du k-Means de référence, les coordonnées des centres de chaque cluster, et une table donnant le minimum et maximum d'entropie lors des tests de stabilité.
Les différentes sorties sont affichées sous forme de tableaux et de cartes.

## DBSCAN
Le méta-noeud _Double DBSCAN_ permet de visualiser le résultat d'une exécution de DBSCAN sur toutes les données, puis divise les données en deux parties (le centre par un _Geo-Coordinate Row Filter_, puis la périphérie par un _Reference Row Filter_) et exécute deux DBSCAN avec des paramètres différents avant de les regrouper (_String Manipulation_ et _Concatenate_). On peut également observer les deux clusterings indépendamment sur une carte. La sortie du méta-noeud est la fusion des deux clusterings.
Ces clusters sont ensuites affichés sous forme de tableau et de carte, et le _Text Mining_ est réalisé sur ce jeu de données.

## Get Flickr Images
Grâce à ce méta-noeud, l'on peut visualiser 10 images parmi celles que l'on a préalablement mises en évidence (_HiLite_). Les photos sont téléchargées sur Flickr puis affichées dans un tableau, ce qui peut aider à donner du sens à un cluster.

## Text Mining
### Filtrage des mots-clés
Le traitement des mots-clés comporte les étapes suivantes :
- création d'un document à partir de l'ensemble des mots-clés
- suppression de la ponctuation, des mots de moins de 3 lettres, des nombres, passage en minuscules, suppression des mots de liaison (prépositions, etc.), radicalisation (suppression des marques de féminin, pluriel, etc.)
- filtrage des mots-clés en fonction de la fréquence d'apparition dans les documents (on ne garde que ceux présents dans au moins 1% des documents)
- création d'un vecteur de bits représentant la présence de chaque mot-clé dans le document

### Recherche des ensembles fréquents pour Fourvière
Un Row Filter permet de conserver uniquement le cluster 23 (ce qui représentait Fourvière lors de notre étude). Puis une recherche des ensembles fréquents (avec l'algorithme Apriori permettant de trouver les ensembles fermés) est effectuée par _Item Set Finder_. Les résultats sont ensuite affichés dans une table, on peut alors interpréter le cluster à l'aide des tags les plus représentés dans le cluster.

### Recherche des règles d'association
Un bloc _One to Many_ permet de remplacer la colonne _Cluster_ en créant une colonne par cluster. Dans ce cas, une photo possèdera un 1 dans la colonne correspondant à son cluster, et 0 dans les autres. Ceci permet ensuite de créer un vecteur de bits, qui pourra être utilisé par le _Association Rule Learner_ qui recherchera des règles d'association à partir des étiquettes de cluster et des tags. Un _Row Filter_ permet de conserver uniquement les règles comportant une étiquette de cluster dans ses antécédents. Une table permet enfin de visualiser les règles du type Cluster_XX -> tags-correspondants.
