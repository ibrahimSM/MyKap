> [!important] Hallucination Detection
> 
![[Pasted image 20261015102733.png]]




> [!info] Retrieval-Based Detection
> Retrieval-based hallucination detection methods compare the generated output of an LLM with trusted external knowledge sources, such as databases and encyclopedias. These methods aim to ensure factual consistency and reduce the risk of unsupported content by grounding the output in verifiable information.


- 
> [!tip] Wang et al. (2023) : savoir quand arrêter de cherche
- https://aclanthology.org/2023.emnlp-main.949.pdf
> Le problème : avec un nombre fixe de documents, on en récupère trop pour les affirmations faciles à vérifier (coût inutile) et pas assez pour les difficiles (erreurs).
> 
> La méthode : la réponse est découpée en sous-affirmations. Pour chacune, on récupère un document à la fois, et après chaque document on met à jour une croyance probabiliste (bayésienne) sur la véracité de l’affirmation. Une règle de décision séquentielle compare alors deux options : s’arrêter et trancher (vrai ou halluciné), ou payer le coût d’un document de plus pour réduire l’incertitude.
> Ils utilisent le jeu de données de SelfCheckGPT : 1 908 phrases tirées d’articles générés par GPT-3, chacune annotée « exacte », « légèrement inexacte » ou « très inexacte ». Les deux catégories inexactes sont regroupées en classe non factuelle, et les phrases exactes forment la classe factuelle. Chaque phrase joue le rôle de « claim » à vérifier. [aclanthology](https://aclanthology.org/2023.emnlp-main.949.pdf)
> L’analogie : un médecin qui prescrit les examens un par un et s’arrête dès qu’il est assez sûr de son diagnostic, plutôt que de commander toute la batterie d’emblée. 

> Le gain : moins de recherches pour une précision égale ou meilleure.
- ![[Pasted image 20261015110319.png]]

> [!tip] FAVA (Mishra et al., 2024) : localiser et typer l’erreu
> https://arxiv.org/pdf/2401.06855
> Le problème : dire « cette réponse contient une hallucination » est peu utile si on ne sait pas où ni de quel type.
> 
> La méthode : FAVA travaille au niveau du segment (span). Il repère le passage précis qui est faux, lui attribue un type d’erreur (entité erronée, relation erronée, contradiction, contenu inventé, affirmation subjective ou invérifiable), puis propose une correction. Le modèle est entraîné sur des données synthétiques : on insère volontairement des erreurs typées dans des textes corrects, ce qui évite une annotation manuelle coûteuse.
> 
> Le gain : il dépasse GPT-4 en détection fine, en détection binaire et en correction.
!![[Pasted image 20261015110238.png]]

![[Pasted image 20261015111105.png]]
> [!tip] . KnowHalu (Zhang et al., 2024) : vérifier sous plusieurs formes
> KnowHalu fonctionne en deux temps.
> 1. **Détection des hallucinations « non fabriquées »** : des réponses qui ne sont pas fausses mais qui ne répondent pas vraiment à la question, parce qu’elles sont trop vagues ou hors sujet. Une vérification factuelle classique les laisse passer puisque rien n’y est faux. 
> 2. **Vérification factuelle multi-forme** : la question est décomposée en sous-requêtes traitées pas à pas. Pour chacune, on récupère des connaissances qu’on reformule sous plusieurs formes (texte libre et triplets structurés sujet-relation-objet). On juge la réponse avec chaque forme, puis on agrège les jugements.
>![[Pasted image 20261015114149.png]]
>



> [!info] Uncertainty-Based Detection
> ![[Pasted image 20261015114450.png]]


> [!important]  Approches au niveau du token
> **Idée :** lire directement les probabilités que le modèle attribue aux tokens qu'il génère.
> $$ \text{Seq-Logprob}(y) = \frac{1}{L}\sum_{t=1}^{L} \log p(y_t \mid y_{<t}, x) $$


> [!important]  Approches sémantiques
>**Idée :** mesurer l'incertitude sur le **sens** plutôt que sur les mots. ### Entropie sémantique (Farquhar et al., 2024) [89] 
>1. Générer plusieurs réponses à la même question.
>2. Regrouper ces réponses en clusters de même sens. 
>3. Calculer l'entropie sur ces clusters : $$ SE(x) = -\sum_{c} p(c \mid x)\,\log p(c \mid x) $$


> [!important] Approches supervisées
> **Idée :** au lieu de définir l'incertitude à la main, entraîner un détecteur sur des données annotées.
> 


> [!info] Embedding-based detection
> Principe commun
> > Une sortie fidèle doit rester **sémantiquement proche de ses sources** dans un espace vectoriel partagé. On compare donc les représentations de l'entrée, de la sortie et éventuellement de références externes : un écart important signale une hallucination probable.
> >
> > Trois familles : **similarité**, **gradient**, **spectrale**.


> [!important]  Approches par similarité
> 1. **Dale et al.** : en traduction automatique, une faible similarité entre la phrase source et sa traduction (mesurée par LaBSE, LASER ou un modèle NLI inter-langues) signale une dérive sémantique, LaBSE étant le plus performant.
> 2. les embeddings contextuels du modèle permettent une détection non supervisée en temps réel, avec les meilleurs résultats obtenus sur l'embedding du dernier token de la dernière couche.
> 3. **Nonkes et al.** : les embeddings des phrases générées forment un graphe de similarité sur lequel un réseau d'attention de graphe détecte les hallucinations à partir du sens et de la structure des relations entre sorties



> [!important] Approches par gradient
> En comparant par un développement de Taylor la sortie générée avec et sans contexte, la méthode mesure à quel point la réponse dépend réellement de l'entrée (variation des embeddings et signaux de gradient), puis un classifieur MLP entraîné sur ces caractéristiques atteint l'état de l'art en détection d'hallucinations.


> [!important]  Approches spectrales
>  1. **HalluShift (Dasgupta et al.)** : la méthode détecte les hallucinations en mesurant les décalages de distribution dans les états cachés et les couches d'attention du modèle, avec un AUROC de 89,9 % sur TruthfulQA.
> 2. **LapEigvals** : chaque carte d'attention est traitée comme un graphe dont les valeurs propres du laplacien servent de caractéristiques spectrales pour repérer les hallucinations, avec un AUROC de 88,9 % sur TriviaQA.



> [!info] Learning-Based Detection : Détection par apprentissage
> 
**Principe** : des modèles supervisés ou non supervisés, entraînés sur des données annotées ou des pseudo-étiquettes, apprennent à classer les sorties d'un LLM comme hallucinées ou factuelles, ce qui leur permet de capter des motifs plus fins que les règles ou la similarité et de généraliser à différents types d'hallucinations, domaines et tâches.


> [!info]  Self-Consistency-based Detection : Détection par auto-cohérence
> 1. **Principe** : sans référence externe ni vérité terrain, on génère plusieurs réponses à une même requête et on mesure leur cohérence, une forte divergence signalant une hallucination probable.
>  2. **Méthodes fondées sur la réponse** : on varie les paramètres de décodage (température, top-k) pour obtenir plusieurs réponses à la même question, puis on évalue leur cohérence.
>3. **Méthodes fondées sur la question** : on reformule la question de plusieurs façons et on vérifie que le modèle répond de manière cohérente à toutes ces formulations.


