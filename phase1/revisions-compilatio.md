# Rapport de phase 1 — révisions après analyse Compilatio

Analyse Compilatio du 3 novembre 2026 : **33 %** de textes suspects (détection IA 33 %, similitudes < 1 %).

Pour chaque passage souligné par Compilatio : le texte d'origine (début … fin) et une proposition de remplacement.
Les reformulations cassent les tournures trop symétriques (« X n'est pas A, c'est B »), raccourcissent les phrases
et ajoutent des éléments concrets tirés du terrain (KNIME, Power Query, SQL Server, nœuds GroupBy…).

> Conseil général : le rapport ne nomme presque jamais les outils (« l'outil de flux visuel », « le moteur de
> transformation »). Les nommer une fois dans l'introduction (KNIME, Power BI / Power Query, SQL Server) rend le texte
> plus concret. Les remplacements ci-dessous le font ; garde le reste du texte tel quel ou harmonise si tu préfères.

---

## 1. Introduction

**Origine** : « L'usage de cet outil s'étend ensuite par accumulation … dans une seconde base dite propre. »

**Remplacement** :
> Chaque nouveau besoin de reporting y ajoute quelques nœuds, et l'outil finit par porter une bonne partie de la logique de l'entreprise. C'est ce qui s'est passé chez Adiwatt, l'entreprise où se déroule notre apprentissage. Sur un serveur SQL Server, une première base reçoit telles quelles les extractions du progiciel de gestion. Un workflow KNIME de 193 nœuds et 219 connexions relit ces tables en entier, les nettoie et les croise, puis écrit douze tables dans une seconde base, dite propre.

---

## 2.1. La chaîne décisionnelle

**Origine** : « La préparation est traditionnellement décrite par l'acronyme ETL … question qui structurera l'essentiel de cette revue. »

**Remplacement** :
> On parle traditionnellement d'ETL (extraction, transformation, chargement) : un outil tiers lit les tables sources, les transforme, puis charge le résultat dans une base intermédiaire. Avec l'approche ELT, on charge d'abord et on transforme ensuite, directement dans la base qui contient les données. Au-delà de l'ordre des lettres, ce qui change surtout, c'est la machine qui fait le calcul. Cette question revient tout au long de la revue.

**Origine** : « Elle journalise ses exécutions et notifie les échecs par courriel … rien n'est produit entre-temps. »

**Remplacement** :
> En cas d'échec, la tâche l'inscrit dans son journal et envoie un courriel. En revanche, elle ne sait rien reprendre : il faut relancer tout le traitement, et tant qu'il n'a pas abouti, aucune donnée nouvelle n'arrive dans le rapport.

**Origine (légende)** : « Figure 2 — L'orchestration de la chaîne : une tâche planifiée … la publication du rapport. »

**Remplacement** :
> Figure 2 — La tâche « PBI - RAW TO REPORT TO CLOUD » dans le Planificateur de tâches Windows. Elle se déclenche quatre fois par jour : alimentation de la base brute, exécution du workflow, publication du rapport.

**Origine** : « La modélisation organise ensuite les données en un schéma interrogeable … qui portent les axes d'analyse. »

**Remplacement** :
> Vient ensuite la modélisation. Les données sont le plus souvent rangées en étoile : au centre, les tables de faits (une commande, une livraison) ; autour, les tables de dimensions qui servent à filtrer et à regrouper (le client, le mois, la famille de produits).

---

## 2.2. Deux manières d'écrire une préparation de données

**Origine** : « Surtout, l'évaluation est paresseuse … réorganiser le travail avant de le faire. »

**Remplacement** :
> L'évaluation y est paresseuse : le moteur assemble d'abord la description de toutes les étapes, et ne calcule qu'au moment où il a besoin du résultat. C'est ce qui lui permet de réorganiser le travail, par exemple de filtrer avant de joindre. Un flux de nœuds ne le peut pas, puisqu'il calcule chaque étape dès qu'il l'atteint.

**Origine** : « Une troisième différence ne se révèle qu'en cas d'anomalie … l'exécution s'arrête là où elle en est. »

**Remplacement** :
> Les deux modèles diffèrent aussi dans leur façon de traiter les erreurs, et c'est même l'une des raisons pour lesquelles l'entreprise veut migrer. Dans KNIME, quand un nœud ne parvient pas à traiter une valeur, il passe en erreur et le workflow s'arrête sur ce nœud.

---

## 2.3. Déléguer le calcul à la source

**Origine** : « Les éditeurs désignent ce mécanisme par des termes propriétaires … détermine donc à lui seul le volume rapatrié. »

**Remplacement** :
> Microsoft appelle ce mécanisme *query folding* ; dans les articles scientifiques, on parle plutôt d'optimisation fondée sur les capacités de la source, ou de *predicate pushdown*. Nous utiliserons le mot « délégation ».
>
> La délégation ne couvre jamais toute une requête par principe : à partir de la première étape que la source ne sait pas exécuter, tout le reste est calculé localement. C'est donc la position de cette étape dans la requête qui fixe la quantité de données ramenée depuis le serveur.

---

## 2.4. Équivalence, tolérance et oracle

**Origine** : « Nous dirons que deux pipelines sont équivalents … quelques unités du dernier chiffre significatif. »

**Remplacement** :
> Deux pipelines sont équivalents s'ils donnent la même sortie à partir des mêmes données. Dit comme cela, c'est simple ; en pratique, trois points posent problème. Il faut d'abord décider si l'on compare les tables comme des ensembles ou comme des multi-ensembles, c'est-à-dire si les doublons comptent, et nos tables en contiennent. Ensuite, l'ordre des lignes n'est en général pas garanti, si bien qu'un opérateur qui en dépend (garder la première ligne d'un groupe, par exemple) n'a pas de résultat bien défini. Enfin, l'addition en virgule flottante n'est pas associative : additionner les mêmes montants dans un autre ordre peut changer les derniers chiffres.

**Origine** : « Une migration se trouve dans la situation inverse … porte un nom, le test différentiel. »

**Remplacement** :
> Dans une migration, c'est l'inverse : l'ancienne chaîne tourne toujours, et ses sorties donnent exactement ce que la nouvelle doit produire. Faire tourner deux implémentations d'un même traitement et comparer leurs résultats, c'est ce qu'on appelle un test différentiel.

**Origine** : « L'oracle est ici un système vivant … l'équivalence devient indémontrable. »

**Remplacement** :
> Les sorties du workflow changent à chaque exécution, puisque la base brute est rechargée entre-temps. Avant de toucher à quoi que ce soit, il faut donc exporter les douze tables, datées, avec les paramètres de l'exécution. Une fois le workflow arrêté, cet export ne pourra plus être refait, et il n'y aura plus rien à quoi comparer.

---

## 3.1. Périmètre de l'étude

**Origine** : « L'optimisation de requêtes réparties et la modélisation des processus ETL … fournissent enfin les instruments de la validation. »

**Remplacement** :
> Pour savoir où placer le calcul, on peut s'appuyer sur l'optimisation de requêtes réparties et sur la modélisation des processus ETL. Comprendre un workflow dont personne n'a écrit la spécification relève de la rétro-ingénierie. Le portage lui-même touche à la traduction de code et à l'ingénierie dirigée par les modèles. Pour la validation, enfin, nous nous sommes tournés vers la vérification d'équivalence de requêtes, le test des systèmes de bases de données et les travaux sur la reproductibilité.

**Origine** : « La dépendance à un outil unique en est un cas particulier … comment prouver que la migration n'a rien changé. »

**Remplacement** :
> Notre workflow en est un bon exemple : KNIME est gratuit, mais la logique métier n'existe nulle part ailleurs que dans ses fichiers, et c'est cela qui nous lie à l'outil.
>
> Nous n'avons pas voulu présenter les travaux domaine par domaine, ce qui aurait donné une suite de résumés. Nous les avons rangés selon quatre questions que l'on peut poser à n'importe quelle approche : où et quand se fait le calcul ? Que garantit le modèle d'exécution ? Comment mettre en correspondance deux pipelines ? Comment vérifier que la migration n'a rien changé ?

**Origine** : « Les documentations d'éditeurs et les outils, nécessaires mais non évalués par les pairs, sont listés en annexe. »

**Remplacement** :
> Les documentations d'éditeurs et les outils logiciels nous ont été utiles, mais ils ne sont pas évalués par des pairs : nous les avons mis à part, en annexe B.

---

## 3.2. Premier axe : où et quand le calcul s'exécute

**Origine** : « Haas et al. (1997) la formalisent : dans un système interrogeant des sources hétérogènes … sans jamais renvoyer à cette filiation. »

**Remplacement** :
> Haas et al. (1997) l'ont formalisée pour les systèmes qui interrogent plusieurs sources hétérogènes : chaque source déclare ce qu'elle sait faire, et l'optimiseur ne retient que les plans qu'elles peuvent exécuter. Déléguer fait donc partie de l'optimisation elle-même. C'est, selon nous, l'origine académique du *query folding* de Power Query, même si la documentation de Microsoft ne cite jamais ce type de travaux.

**Origine** : « Ils le modélisent comme un état dans un espace d'états … ne change pas son résultat. »

**Remplacement** :
> Ils représentent un flux ETL comme un état, et l'optimisent en passant d'un état à l'autre par des transformations (échanger deux opérateurs, en factoriser, en fusionner) dont ils prouvent qu'elles ne changent pas le résultat. Placer un filtre avant une jointure devient alors une réécriture démontrée, et plus seulement une bonne habitude. C'est ce qui nous permettra de justifier, en phase de réalisation, qu'un réordonnancement des étapes du pipeline migré ne modifie pas sa sortie.

**Origine** : « Les travaux récents étendent la délégation au-delà du relationnel … reste donc un arbitrage plutôt qu'une règle. »

**Remplacement** :
> Yan et al. (2023) sortent du cadre relationnel : ils poussent des prédicats à travers des opérateurs non relationnels et des fonctions écrites par l'utilisateur, et vérifient formellement que la sortie ne change pas. C'est le seul article de notre corpus qui traite à la fois du déplacement du calcul et de la preuve que ce déplacement est correct. Il montre aussi que le gain disparaît quand un opérateur n'est délégable qu'en partie : choisir où calculer reste un compromis, à faire au cas par cas.

**Origine** : « Armbrust et al. (2015) donnent l'illustration inverse … jusque dans la source. »

**Remplacement** :
> Spark SQL (Armbrust et al., 2015) fonctionne à l'inverse : son interface ne calcule rien tant qu'on ne demande pas de résultat, et son optimiseur à base de règles en profite pour envoyer les filtres et les projections jusqu'à la source.

---

## 3.3. Deuxième axe : ce que le modèle d'exécution garantit

**Origine** : « Un opérateur d'agrégation qui retient la première valeur … il change silencieusement le jour où elles en changent. »

**Remplacement** (⚠️ vérifie que les agrégats First/Last étaient bien identifiés au moment de la phase 1 ; sinon, garde la formulation générale) :
> Dans notre workflow, plusieurs nœuds GroupBy utilisent les agrégats « First » et « Last », qui gardent la première ou la dernière valeur lue dans chaque groupe. Ce résultat n'a de sens que si l'ordre de lecture est fixé. Or KNIME ne le garantit pas, Power Query non plus, et aucun des deux ne le précise dans sa documentation. Tant que la base renvoie les lignes dans le même ordre, tout va bien ; si cet ordre change un jour, le résultat change aussi, sans aucun message d'erreur.

**Origine** : « fournissent le vocabulaire permettant de raisonner rigoureusement … elle n'est pas technique mais métier. »

**Remplacement** :
> Pritchard et Wicenec (2024) donnent des mots précis pour ces garanties : relancer, répéter, recalculer, reproduire, répliquer. Ils distinguent en particulier la réplication au bit près d'une équivalence simplement « utile ». Pour nous, la conséquence est concrète : avant de vérifier la migration, il faut choisir ce que l'on attend. Des tables identiques au bit près ? Ou seulement les mêmes indicateurs dans le rapport ? Ce choix revient aux utilisateurs du rapport autant qu'à nous.

---

## 3.4. Troisième axe : établir la correspondance entre deux pipelines

**Origine** : « Le patron de Böhm et al., en revanche, se transpose … puis à engendrer la cible à partir de celle-ci. »

**Remplacement** :
> La démarche de Böhm et al. est la seule que nous pouvons reprendre presque telle quelle : ne pas traduire nœud par nœud, mais écrire d'abord une description de la logique métier qui ne dépend d'aucun des deux outils, puis produire les requêtes Power Query à partir de cette description.

---

## 3.5. Quatrième axe : prouver que la migration n'a rien changé

**Origine** : « Cette propriété est décisive en pratique … non sur des chaînes de traitement complètes. »

**Remplacement** :
> C'est très utile en pratique : savoir que deux requêtes diffèrent n'aide pas beaucoup si l'on ne sait pas sur quelle donnée. Mais ces deux travaux comparent des requêtes deux à deux, pas des chaînes de traitement entières comme la nôtre.

**Origine** : « Le principe se transpose à la comparaison de deux implémentations … conditionne toute la validation. »

**Remplacement** :
> On peut appliquer la même idée à deux implémentations d'un pipeline, et notre cas est même plus simple : nous n'avons pas à construire l'oracle, puisque le workflow KNIME tourne encore et le fournit. Il faut seulement archiver ses sorties avant de l'arrêter. Cette question de calendrier est rarement évoquée dans les articles, alors que toute la validation en dépend.

**Origine** : « Vient ensuite l'assertion déclarative … écarts que le portage n'explique pas. »

**Remplacement** :
> Schelter et al. (2018) proposent une troisième voie, l'assertion déclarative. On écrit des contraintes de qualité (« aucun montant négatif », « une seule ligne par commande ») que l'outil traduit en requêtes d'agrégation, un peu comme des tests unitaires appliqués aux données. Cette méthode n'a pas besoin de référence, puisqu'elle vérifie des propriétés et non des valeurs précises. Elle prend tout son sens après la bascule : un traitement qui tourne quatre fois par jour doit être surveillé dans la durée, car les données amont évoluent et peuvent créer des écarts qui n'ont rien à voir avec la migration.

**Origine** : « Deux constats ressortent de cet axe … à l'échelle d'une chaîne complète. »

**Remplacement** :
> Nous retenons deux choses de cet axe. D'abord, plus on passe de la requête au pipeline complet, plus les garanties s'affaiblissent : nous n'avons trouvé aucun travail sur l'équivalence d'une chaîne de traitement entière. Ensuite, nous n'avons trouvé en accès ouvert aucun protocole de validation de migration qui propose une méthode reproductible et un critère de réussite clair ; les éditeurs fournissent surtout des listes de vérifications. La phase de réalisation essaiera de combler ce manque en appliquant le test différentiel à toute la chaîne.

---

## 4.1. Ce que la revue établit

**Origine** : « La migration ne se réduit donc pas à un changement d'outil … ce que calcule le moteur de restitution. »

**Remplacement** :
> Migrer ne consiste donc pas seulement à changer d'outil : il faut aussi redécider de ce que calcule SQL Server et de ce que calcule Power BI.

**Origine** : « Le deuxième axe nous fournit le vocabulaire des garanties … relève du métier avant de relever de la technique. »

**Remplacement** :
> Le deuxième axe nous a fourni le vocabulaire des garanties, venu de domaines où nous ne l'attendions pas, le génie logiciel et la science reproductible : Shi et al. (2016) pour le non-déterminisme caché, Pritchard et Wicenec (2024) pour les niveaux de reproductibilité. En pratique, cela nous oblige à fixer le niveau d'équivalence attendu avant la validation, en accord avec les utilisateurs du rapport.

---

## 4.2. Ce que la revue laisse ouvert

**Origine** : « La documentation d'éditeurs propose des listes de contrôle … seuil de tolérance annoncé. »

**Remplacement** :
> Les éditeurs fournissent des listes de points à vérifier, mais pas de méthode reproductible avec un critère de réussite et une tolérance fixés à l'avance.

**Origine** : « Le mécanisme de délégation des outils décisionnels … pour être discuté rigoureusement. »

**Remplacement** :
> Le *query folding* n'a pas été étudié en tant que tel par la recherche : c'est un terme commercial, documenté seulement par Microsoft, et il faut le rapprocher de l'optimisation fondée sur les capacités de la source pour pouvoir le discuter avec des références solides.

---

## 4.3. Démarche envisagée pour la phase de réalisation

**Origine** : « (ii) À quel niveau d'équivalence … à budget de vérification fini ? »

**Remplacement** :
> (ii) Quel niveau d'équivalence viser pour valider la migration (identité au bit près, égalité des multi-ensembles avec une tolérance numérique fixée à l'avance, ou seulement mêmes indicateurs dans le rapport), et quel protocole permet de le vérifier en un temps limité ?

**Origine** : « (iii) Que gagne-t-on et que perd-on, mesurément … en présence d'une donnée invalide ? »

**Remplacement** :
> (iii) Que fait gagner ou perdre la suppression de l'étage intermédiaire, chiffres à l'appui : temps d'actualisation, volume de données transféré, nombre d'éléments à maintenir, comportement face à une donnée invalide ?

**Origine** : tout le paragraphe « Notre démarche se déroulera en cinq temps … l'intérêt de la migration devra se chercher ailleurs. »

**Remplacement** :
> Le travail se fera en cinq étapes. (1) Figer l'oracle, à faire sans attendre : exporter les douze tables de sortie avec leur date et relever, tant que le workflow tourne encore, ses caractéristiques d'exécution (temps par étape, volume intermédiaire, pic mémoire, configuration). (2) Reconstruire la logique du workflow à partir de ses fichiers (workflow.knime et le settings.xml de chaque nœud), sous la forme du modèle pivot décrit plus haut. (3) Écrire les requêtes Power Query à partir de ce modèle, en déléguant le plus possible à SQL Server et en notant, pour chaque requête, l'étape où la délégation s'arrête. (4) Valider : comparer chaque table ligne à ligne avec l'oracle, avec une tolérance fixée avant de lancer la comparaison, puis écrire les règles métier sous forme d'assertions vérifiées à chaque actualisation. (5) Mesurer et comparer les deux chaînes. Nous avons déjà un premier ordre de grandeur, à prendre avec précaution puisqu'il n'a pas été mesuré selon un protocole : le workflow actuel tourne en moins de cinq minutes, et une première version reconstruite s'est actualisée en moins de deux. Même confirmé, ce gain de temps resterait modeste ; l'intérêt de la migration est surtout ailleurs.

**Origine** : « Notre critère de réussite est binaire et vérifiable … à la tolérance annoncée près. »

**Remplacement** :
> Le critère de réussite est simple à vérifier : les douze tables du nouveau pipeline doivent être identiques à celles de l'oracle, ligne par ligne, à la tolérance près.

**Origine** : « Le troisième est la dérive des données amont … puis seulement en exploitation. »

**Remplacement** :
> Le troisième risque est que les données amont changent pendant la comparaison et créent des écarts sans lien avec la migration. C'est pourquoi nous comparerons d'abord sur un jeu de données figé, et seulement ensuite en conditions réelles.

**Origine** : « Ainsi posée, la contribution que nous attendons … ce que le déplacement du calcul change effectivement. »

**Remplacement** :
> Ce que nous attendons de la phase de réalisation n'est donc pas seulement la migration, qui est un travail d'ingénierie, mais surtout les moyens de la vérifier : une description de la logique métier indépendante de KNIME et de Power Query, un protocole qui applique le test différentiel à une chaîne entière et non plus à une seule requête, et des mesures de ce que le changement d'outil apporte réellement.

---

## À ajouter à la fin (avant les références, ou en annexe C)

### Déclaration d'utilisation de l'intelligence artificielle

> Conformément aux règles de la formation, nous signalons avoir utilisé un assistant d'intelligence artificielle générative ([nom de l'outil]) dans la préparation de ce rapport, pour deux usages.
>
> **Recherche bibliographique.** L'outil nous a proposé des pistes de travaux et en a fourni des résumés, ce qui nous a aidés à repérer les sources pertinentes et à choisir celles à retenir. Toutes les références citées ont ensuite été lues par nous ; le choix final du corpus et son organisation en quatre axes sont les nôtres.
>
> **Aide à la rédaction.** L'outil a servi à reformuler et à relire certains passages.
>
> Le sujet, l'analyse du terrain, la problématique et les conclusions relèvent de notre propre travail, et nous restons responsables de l'ensemble du contenu.

---

## Avant de rendre

- [ ] Vérifier chaque référence (titre, auteurs, année, lien), en priorité **Böhm et al. (2026), arXiv:2602.04341** et **Pritchard & Wicenec (2024), arXiv:2406.01146**.
- [ ] Vérifier que chaque article dit bien ce que le rapport lui attribue.
- [ ] Relire les remplacements à voix haute et ajuster avec tes propres mots là où ça ne sonne pas comme toi.
- [ ] Repasser le texte dans Compilatio.
