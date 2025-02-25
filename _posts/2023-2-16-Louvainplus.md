---
layout: post
title: Improving the Louvain Algorithm for Community Detection with Modularity Maximization [FR]
mathjax: true
---
{::options parse_block_html="true" /}
Cet algorithme est une expansion de l'algorithe Louvain, en ajoutant une phase de raffinement et de uncoarsening.
Les étapes de Louvain+:
1. Exécuter Louvain et obtenir des graphes grossiers G  et des clustering $C^{1},C^{2},C^{3},...C^{l}$ avec L le plus haut niveau.
2. Appliquer la phase de dégrossissement de $C^{l-1}$ et projetter le clustering courant à un nouveau clustering $\bar{C}^{l-2}$ ou toute communauté grossière du clustering courant est uncoarced(dépliée, unfolded, dégrossie) vers des communautés qui la composent.
- Le nouveau clustering $\bar{C}^{l-2}$ est raffiné par l'heuristique VM pour améliorer sa qualité. Le clustering $\bar{C}^{l-2}$ amélioré sert comme clustering initial pour la prochaine projection(pour revenir à 2 )
- Ce processus est répété jusqu'au niveau 0.
- Ce n'est pas la peine de commencer par $C^{l}$ comme aucun changement ne peut-être effectué par l'heuristique VM durant la dernière itération de la phase de grossissement.

#### Description du processus de projection:

Au niveau *l*:
- On pose la relation $\Gamma$ définie par $v_1 \Gamma v_2$ veut dire que les sommets $v_1$ et $v_2$ appartiennent à la même communauté dans $C^{l}$ 
- On pose la relation $\gamma (v)$  qui désigne la communauté auquelle appartient $v$ dans $C^{l}$.
1. Pour chaque niveau *l* de *L-2* , *L-3*....., le clustering $\bar{C}^{l}$  est le résultat de la projection de  $\bar{C}^{l+1}$ sur  $C^{l}$ qui est optimisé par l'heuristique VM.
2. Dans  $\bar{C}^{l}$,  deux sommets *v1* et *v2* appartiennent à la même communauté, si les sommets dans  $G^{l+1}$  correspondent aux communautés $\gamma (v_1)$ et $\gamma (v_2)$ de la communauté  $C^{l}$ appartiennent à la même communauté dans  $C^{l+1}$.
	- Formellement on note:
		$v_1 \Gamma(l) v_2$  ≡ T( $\gamma (v_1)$ ) $\Gamma$(l+1) T( $\gamma (v_2)$ )
	-  $T^{l+1}(c)$ veut dire le sommet correspondant à la communauté c du clustering $C^{l}$  le sommet corresppondant dans $G^{l+1}$.

### Expériences et résultats:

#### Benchmark et protocole de test:

Pour l'évaluation, on utilise 13 réseaux de différents domaines d'application.
Les deux algorithmes sont codés en Free Pascal sur le même équipement. On les exécute sur 100 instances d'une manière déterministique(pas d'aléatoire).
On utilise un petit gain minimal de modularité $\epsilon$  entre deux itérations consécutives pour l'arrêt de la procédure de l'heuristique VM pour avoir peu d'itérations.
Louvain+ dans la première phase clairement ne modifie pas la modularité trouvée par Louvain comme c'est les mêmes étapes.

#### Temps d'exécution et modularité:

- Louvain+ prend un peu plus de temps(20%) que Louvain mais sa complexité est la même que Louvain, O(m), qui croit avec le nombre d'arêtes.
- La courbe delta montre le temps excessif que prend Louvain+.
- Le temps CPU de Louvain+ est plus grand que Louvain avec $\epsilon =10^-5$ mais si on relaxe la phase de grossissement avec $\epsilon = 10^-2$, le temps est réduit.
**Si on compare le gain en modularité :**

- Louvain+ avec $10^-5$ avec $10^-2$ on presque le même gain en modularité, tout en ayant un gain en temps.
- On peut expliquer ceci par le fait qu'avec le epsilon relaxé, la phase de grossissement est réduite, même si ça mène à une réduction de modularité à la fin de la phase de grossissement, la modularité est améliorée dans la phase de dégrossissement(uncoarsing phase).

### Mauvais sommets et changements structurels du clustering:

Une communauté est considerée bien connectée si le degré intérieur de toutes les arêtes est supérieur au degré extérieur.(même si ce critère n'est pas satisfait dans des réseaux réels). Selon nos observations, la modularité maximale est associée à un petit nombre d'arêtes mal placées.

La correction est le nombre d'arêtes qui ne **satisfaisent** pas ce critère.
Selon le tableau 2, Avec le raffinement,le pourcentage de correction pour tout les réseaux est positif entre 0.1 et 1.4, avec une moyenne de 0.5. Ceci permet de confirmer l'utilité de cette phase introduite dans Louvain+.

On compare aussi avec le critère NMI la différence de structure de Louvain et Louvain+ , et on trouve que ça varie de 0(un clustering totalement différent)  à 1 (exactement le même cluster). Pour la table 2, le NMI est entre 0.84 et 0.98 ce qui eut dire de grands changements entre les clusters de Louvain de Louvain+.




