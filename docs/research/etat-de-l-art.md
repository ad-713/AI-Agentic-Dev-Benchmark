# État de l'art : benchmarks de développement logiciel agentique (novembre 2026)

> Recherche réalisée le 2026-11-02. Elle couvre les benchmarks, les harnais et les méthodes réutilisables pour les deux objectifs du projet :
> - **P1** : comparer des *workflows* complets de développement assistés par IA (humain + agent de code + skills/prompts) qui reconstruisent une application web de zéro.
> - **P2** : comparer des agents ou des skills sur des tâches unitaires (issues), avec une résolution de référence.
>
> **Sources.** Les sources primaires (papiers arXiv, dépôts officiels, documentation officielle) ont été privilégiées. Les licences et les dates de dernière activité des dépôts ont été vérifiées via l'API GitHub le 2026-11-02. Tout ce qui n'a pas pu être confirmé dans une source primaire est marqué **[non vérifié]**. Les scores de classement changent vite : ils sont donnés à titre indicatif, avec leur date.
>
> **Légende « Pertinence pour ce projet »** :
> - **Réutiliser tel quel** : on peut l'intégrer directement.
> - **Réutiliser la méthodologie** : on reprend la façon de construire ou de noter, pas l'artefact.
> - **Inspiration seulement**.

---

## 1. Benchmarks agentiques au niveau issue (famille SWE-bench)

### 1.0 Méthodologie commune et problèmes connus

**Construction d'une tâche.** Le protocole SWE-bench ([papier](https://arxiv.org/abs/2310.06770)) est devenu le standard de fait :

1. On part d'une PR fusionnée qui référence une issue.
2. On fixe le **commit de base** (le parent de la PR).
3. On sépare le diff de la PR en deux parties :
   - le **gold patch** (le code) ;
   - le **test patch** (les tests ajoutés ou modifiés).
4. On construit une image Docker par tâche.
5. On exécute les tests avant et après application du gold patch pour obtenir deux listes :
   - **FAIL_TO_PASS** : tests qui échouent avant le patch et passent après. Ils prouvent que l'issue est résolue.
   - **PASS_TO_PASS** : tests qui passent avant comme après. Ils détectent les régressions.

**Évaluation.** L'agent ne reçoit que l'énoncé de l'issue et le dépôt au commit de base. Les tests restent **cachés** : on les applique seulement après l'exécution de l'agent, sur l'état final du dépôt. Une tâche est « résolue » si tous les tests F2P et P2P passent.

**Problèmes de contamination et de fuite (2025-2026)** :

- **Mémorisation et saturation.** En février 2026, OpenAI a cessé de publier ses scores sur SWE-bench Verified ([billet OpenAI](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/) ; la page renvoie un 403 au fetch, son contenu a été lu via des extraits de recherche).
  - Raisons avancées : des modèles frontière reproduisent certains correctifs de mémoire, et un audit de tâches non résolues montre beaucoup de tâches défectueuses (tests trop stricts ou énoncés ambigus).
  - OpenAI recommande désormais SWE-bench Pro.
  - Le chiffre « ≥59,4 % de tâches défectueuses » circule ([The Decoder](https://the-decoder.com/openai-wants-to-retire-the-ai-coding-benchmark-that-everyone-has-been-competing-on/)) mais **[non vérifié]** dans le billet original.
- **Fuite par l'historique git.** Des agents lancent `git log --all` et lisent le commit futur qui corrige l'issue.
  - Datacurve rapporte que des modèles Claude Opus lisent le commit « gold » dans `.git` sur SWE-bench Pro ([DeepSWE](https://deepswe.datacurve.ai/blog/deepswe)).
  - OpenHands a corrigé une faille de reward hacking sur Commit0 avec des clones superficiels (`--depth 1`), ce qui a changé les scores ([blog OpenHands Index](https://openhands.dev/blog/analyzing-and-improving-openhands-index)).
  - SWE-Bench Pro Verified ([arXiv 2609.08149](https://arxiv.org/abs/2609.08149), sept. 2026) ajoute des garde-fous anti-fuite et corrige les tâches défectueuses. Avec ces corrections, certains modèles obtiennent des scores « nettement inférieurs » à ceux annoncés auparavant.
- **Tests imparfaits.** DeepSWE rapporte que le vérificateur de SWE-bench Pro est en désaccord avec un juge IA sur 32 % des essais, contre 1,4 % sur DeepSWE (chiffre du vendeur).

**Leçons pour notre projet** :
- Les tâches doivent être privées ou nouvelles.
- L'historique git doit être purgé dans l'environnement de l'agent.
- Les tests doivent être comportementaux et valider plusieurs implémentations possibles.
- Chaque tâche doit être validée par un « oracle » (la solution de référence doit obtenir 100 %) et par un agent nul (score de 0 attendu).

### 1.1 SWE-bench (original, Lite, Verified, Multimodal, Multilingual)

- **Liens** : [swebench.com](https://www.swebench.com/) ; [GitHub SWE-bench/SWE-bench](https://github.com/SWE-bench/SWE-bench) ; [papier ICLR 2024](https://arxiv.org/abs/2310.06770).
- **Variantes** :
  - **Original** : 2 294 tâches Python issues de 12 dépôts.
  - **Lite** : 300 tâches.
  - **Verified** : 500 tâches validées par des humains (OpenAI, 2024).
  - **Multimodal** : issues front-end JavaScript avec captures d'écran ([arXiv 2410.03859](https://arxiv.org/abs/2410.03859)).
  - **Multilingual** : 300 tâches, 42 dépôts, 9 langages : C, C++, Go, Java, JS, TS, PHP, Ruby, Rust ([page](https://www.swebench.com/multilingual.html)).
  - Un classement « bash-only » mesure les modèles avec mini-swe-agent.
- **Outils annexes** : `sb-cli` permet l'évaluation dans le cloud.
- **Licence** : MIT. Dépôt actif (dernier push 2026-09).
- **Pertinence** : **réutiliser la méthodologie** (F2P/P2P, image Docker par tâche, harnais d'évaluation). Verified est saturé et contaminé, donc à ne pas utiliser comme mesure principale.

### 1.2 SWE-bench Pro (Scale AI)

- **Liens** : [arXiv 2509.16941](https://arxiv.org/abs/2509.16941) ; [GitHub scaleapi/SWE-bench_Pro-os](https://github.com/scaleapi/SWE-bench_Pro-os) ; [Scale Labs](https://labs.scale.com/papers/swe-bench-pro).
- **Description** : 1 865 tâches « longue durée » issues de 41 dépôts, réparties en trois jeux :
  - **public** : dépôts sous licence copyleft (GPL), choisis pour limiter l'entraînement dessus ;
  - **held-out** : réservé à la détection de sur-apprentissage ;
  - **commercial** : codebases privées de startups.
- **Évolutions récentes** :
  - **Pro V2** (22/09/2026) : 642 tâches publiques validées, plus un sous-ensemble HARD-51. Les tâches sont **packagées au format Harbor**, et le patch de référence résout 642/642.
  - L'audit « Pro Verified » (arXiv 2609.08149) a corrigé les fuites.
- **Licence** : MIT pour le code. Les dépôts sources sont sous GPL.
- **Pertinence** : **réutiliser la méthodologie** (sources privées, held-out, packaging Harbor). On peut aussi l'utiliser tel quel comme point de comparaison externe via Harbor.

### 1.3 SWE-bench-Live (Microsoft) et RepoLaunch

- **Liens** : [GitHub microsoft/SWE-bench-Live](https://github.com/microsoft/SWE-bench-Live) ; [arXiv 2505.23419](https://arxiv.org/abs/2505.23419) ; [RepoLaunch](https://github.com/microsoft/RepoLaunch).
- **Description** : benchmark mis à jour en continu, avec 50 nouvelles issues Python vérifiées par mois.
  - Les splits `lite` et `verified` sont figés ; le split `full` contient les dernières issues.
  - **MultiLang** : 1 077 tâches, 8 langages (août 2026).
  - **Windows** : 66 tâches PowerShell.
  - **RepoLaunch** est l'agent qui construit automatiquement l'environnement Docker de n'importe quel dépôt et en déduit les commandes de test.
- **Licence** : MIT, pour les deux dépôts. Actifs (RepoLaunch accepté à NeurIPS 2026).
- **Pertinence** : **réutiliser la méthodologie** (fraîcheur des tâches contre la contamination). **RepoLaunch est réutilisable tel quel** si l'on veut dockeriser des variantes de l'app automatiquement.

### 1.4 SWE-rebench (Nebius)

- **Liens** : [annonce](https://nebius.com/blog/posts/introducing-swe-rebench) ; [dataset 21k tâches](https://nebius.com/blog/posts/swe-rebench-dataset) ; swe-rebench.com.
- **Description** : classement décontaminé, mis à jour mensuellement à partir de tâches récentes. La contamination est suivie par rapport à la date de sortie de chaque modèle, et l'échafaudage d'évaluation est fixe et standardisé. Le benchmark est disponible sur le Harbor Hub.
- **Licence** : **[non vérifié]**.
- **Pertinence** : **réutiliser la méthodologie** (fenêtre temporelle par rapport au cutoff des modèles).

### 1.5 Multi-SWE-bench

- **Liens** : [arXiv 2504.02605](https://arxiv.org/abs/2504.02605) ; [GitHub multi-swe-bench/multi-swe-bench](https://github.com/multi-swe-bench/multi-swe-bench).
- **Description** : 1 632 instances validées par des humains, sur 7 langages (Java, TS, JS, Go, Rust, C, C++), plus 4 723 instances et le pipeline de construction complet.
  - Affiliation ByteDance Seed **[non vérifié dans la source primaire]**.
  - Dépôt peu actif depuis décembre 2025.
- **Licence** : Apache-2.0.
- **Pertinence** : **inspiration**. Le pipeline est utile si l'app de référence n'est pas en Python.

### 1.6 SWE-PolyBench (Amazon)

- **Liens** : [blog AWS](https://aws.amazon.com/blogs/devops/amazon-introduces-swe-polybench-a-multi-lingual-benchmark-for-ai-coding-agents) ; [arXiv 2504.08703](https://arxiv.org/abs/2504.08703).
- **Description** : 2 110 instances issues de 21 dépôts (Java, JS, TS, Python), avec un sous-ensemble stratifié de 500 tâches.
- **Licence** : **[non vérifié]**.
- **Pertinence** : **inspiration**. TS/JS est proche d'une stack web.

### 1.7 SWE-Gym et SWE-smith (données d'entraînement)

- **SWE-Gym** :
  - Liens : [arXiv 2412.21139](https://arxiv.org/abs/2412.21139) ; [GitHub](https://github.com/SWE-Gym/SWE-Gym).
  - Description : 2 438 tâches Python réelles avec environnement exécutable, conçues pour entraîner des agents et des vérificateurs.
  - Licence : Apache-2.0.
- **SWE-smith** :
  - Liens : [arXiv 2504.21798](https://arxiv.org/abs/2504.21798) ; [GitHub SWE-bench/SWE-smith](https://github.com/SWE-bench/SWE-smith).
  - Description : génère des tâches **synthétiques** en injectant des bugs dans des dépôts dont les tests passent. Résultat : 50 000 instances sur 128 dépôts (NeurIPS 2025 Spotlight).
  - Licence : MIT.
- **Pertinence** : **la méthodologie de SWE-smith est très réutilisable pour P2**. Sur notre propre app de référence, on peut générer en masse des tâches de type « bug injecté → les tests le détectent ». C'est utile pour compléter les tâches tirées de vraies issues.

### 1.8 SWE-Lancer (OpenAI)

- **Liens** : [annonce](https://openai.com/index/swe-lancer/) ; [arXiv 2502.12115](https://arxiv.org/abs/2502.12115) ; [GitHub openai/SWELancer-Benchmark](https://github.com/openai/SWELancer-Benchmark).
- **Description** : environ 1 400 tâches freelance Upwork tirées du dépôt open source d'Expensify, pour 1 M$ de rémunérations réelles. Deux types de tâches :
  - **tâches IC**, notées par des tests end-to-end vérifiés trois fois par des ingénieurs ;
  - **tâches « manager »**, où l'agent choisit la meilleure proposition parmi plusieurs.
- **Statut** : le dépôt est **archivé**. Le code semble avoir migré vers [openai/frontier-evals](https://github.com/openai/frontier-evals) **[non vérifié]**. Le split public s'appelle « Diamond ».
- **Licence** : aucune licence sur le dépôt archivé ; frontier-evals est sous MIT.
- **Pertinence** : **réutiliser la méthodologie** :
  - noter avec des **tests E2E navigateur** plutôt qu'avec des tests unitaires, car c'est robuste aux implémentations différentes ;
  - pondérer les tâches par une « valeur » (coût réel).

### 1.9 DeepSWE (Datacurve, mai 2026)

- **Liens** : [blog](https://deepswe.datacurve.ai/blog/deepswe) ; [GitHub datacurve-ai/deep-swe](https://github.com/datacurve-ai/deep-swe).
- **Description** : 113 tâches **écrites de zéro** (jamais fusionnées en amont) sur 91 dépôts et 5 langages.
  - Les vérificateurs testent le comportement via les API publiques et sont exécutés trois fois pour détecter les tests instables (flaky).
  - Toutes les évaluations passent par mini-swe-agent.
- **Licence** : Apache-2.0.
- **Pertinence** : **réutiliser la méthodologie**. C'est le modèle le plus proche de ce qu'on veut pour P2 : tâches originales, vérificateurs comportementaux, solution de référence, contrôle du flakiness.

### 1.10 Interaction et ambiguïté : SWE-Interact, Ambig-SWE

- **SWE-Interact** (Scale AI, juin 2026) :
  - Liens : [arXiv 2606.30573](https://arxiv.org/abs/2606.30573) ; [Scale Labs](https://labs.scale.com/papers/swe-interact).
  - Description : transforme des tâches SWE en sessions multi-tours avec un **utilisateur simulé** (persona). Les exigences sont révélées progressivement, et la tâche reste notée par le vérificateur d'origine.
  - Résultat : les meilleurs modèles passent d'environ 50 % en un seul tour à environ 25 % en interactif. Les auteurs concluent que l'interaction est une capacité distincte de l'autonomie.
- **Ambig-SWE** (CMU) :
  - Lien : [arXiv 2502.13069](https://arxiv.org/abs/2502.13069).
  - Description : variante sous-spécifiée de SWE-bench Verified, où l'agent peut interroger l'utilisateur.
- **Pertinence** : **réutiliser la méthodologie pour P1**. Un humain simulé permet de rejouer un workflow « humain + agent » à moindre coût. Harbor l'intègre nativement (voir §3.1).

---

## 2. Benchmarks de génération d'applications complètes (greenfield)

### 2.1 Vibe Code Bench (Vals AI)

- **Liens** : [vals.ai/benchmarks/vibe-code](https://vals.ai/benchmarks/vibe-code) ; arXiv 2603.04601 ; [CAIS 2026](https://caisconf.org/program/2026/papers/vibe-code-bench-evaluating-ai-models-on-end-to-end-web-application-development/).
- **Description** : 100 spécifications d'applications web (moins d'une page chacune) totalisant plus de 10 000 sous-étapes.
  - L'app déployée est évaluée par un **agent navigateur** qui clique et teste le comportement, pas la syntaxe.
  - Exemple de tâche : « Zeeter », un mini-Twitter avec auth, posts, follow, likes, commentaires et recherche.
  - L'agent dispose de Supabase et d'un navigateur, et peut travailler jusqu'à 5 h ou 1 000 tours.
  - Les résultats sont publiés par harnais (OpenHands, Claude Code, Codex, Cursor CLI, Devin CLI, etc.), avec coût, tokens et durée.
- **Variante « 1-100 »** (vals.ai/benchmarks/vcb-1-100) : une longue chaîne de demandes de changement **dépendantes** sur une même app, où les erreurs se propagent d'une étape à l'autre.
  - Scores très bas : environ 28 % pour le meilleur modèle en septembre 2026, d'après un agrégateur **[non vérifié]**.
  - Jeu de tâches privé.
- **Licence** : jeu non ouvert, accès sur inscription.
- **Pertinence** : **réutiliser la méthodologie**. C'est **le plus proche de P1** :
  - spec courte → app complète → évaluation E2E par un agent navigateur → métriques de coût et de temps par harnais ;
  - la variante 1-100 est le bon modèle pour mesurer la **maintenabilité** produite par un workflow.

### 2.2 Web-Bench (ByteDance)

- **Liens** : [arXiv 2505.07473](https://arxiv.org/abs/2505.07473) ; [GitHub bytedance/web-bench](https://github.com/bytedance/web-bench).
- **Description** : 50 projets de 20 tâches **séquentiellement dépendantes** chacun (jeux, blogs, e-commerce, etc.), notés par des tests E2E **Playwright**.
  - Métriques : Pass@1, et Pass@2 avec une relance accompagnée du contexte d'erreur.
  - L'état de l'art de 2025 était à 25,1 % Pass@1.
- **Licence** : Apache-2.0. Dernier push avril 2026.
- **Pertinence** : **réutiliser la méthodologie** (chaîne de tâches dépendantes notées par Playwright). Des projets du benchmark pourraient servir **tels quels** de tâches secondaires.

### 2.3 Commit0

- **Liens** : [arXiv](https://arxiv.org/abs/2412.01769) ; [GitHub commit-0/commit0](https://github.com/commit-0/commit0) ; commit-0.github.io.
- **Description** : reconstruire une bibliothèque Python à partir d'une spec et d'un squelette d'API aux fonctions vides. La note est le taux de tests unitaires réussis, exécutés dans Docker.
  - Une quarantaine à une soixantaine de bibliothèques selon la version.
  - Utilisé par l'OpenHands Index pour le « greenfield ».
  - Attention à la fuite par git (corrigée par OpenHands avec `--depth 1`).
- **Licence** : MIT.
- **Pertinence** : **réutiliser la méthodologie**. « Squelette d'API + tests cachés » est une variante intermédiaire de P1, entre le from-scratch total et l'issue isolée.

### 2.4 ProgramBench (Meta, 2026)

- **Liens** : [GitHub facebookresearch/programbench](https://github.com/facebookresearch/programbench) ; programbench.com ; [NeurIPS 2026](https://neurips.cc/virtual/2026/poster/139090).
- **Description** : reconstruire un programme à partir d'un **binaire et de sa doc** seulement. 200 projets (Rust, Go, C, C++, Haskell, Java), avec des tests comportementaux générés par fuzzing.
  - Les modèles produisent des implémentations monolithiques très éloignées du code humain.
  - Avec accès internet, jusqu'à 36 % des runs ont été signalés pour recherche du code source **[non vérifié, interview secondaire]**.
- **Licence** : MIT.
- **Pertinence** : **inspiration** :
  - utiliser une **app de référence exécutable comme oracle** (tests différentiels « mon app vs la référence ») ;
  - bloquer l'accès réseau pour éviter la triche.

### 2.5 BaxBench (ETH / LogicStar)

- **Liens** : [arXiv 2502.11844](https://arxiv.org/abs/2502.11844) ; [GitHub logic-star-ai/baxbench](https://github.com/logic-star-ai/baxbench) ; [AutoBaxBuilder (arXiv 2512.21132)](https://github.com/eth-sri/autobaxbuilder).
- **Description** : 28 scénarios de backend combinés à 14 frameworks, soit 392 tâches.
  - Chaque scénario a des **tests fonctionnels** et des **exploits de sécurité**.
  - Le score « correct ET sécurisé » est nettement plus bas que le score fonctionnel seul.
  - AutoBaxBuilder génère automatiquement de nouveaux scénarios.
- **Licence** : MIT.
- **Pertinence** : **réutiliser tel quel ou la méthodologie** pour les aspects « production » de P1 (auth, injection, contrôle d'accès). Les exploits sont adaptables à notre app.

### 2.6 FullStack-Bench / WebGen-Bench

- **Liens** : [FullStack-Agent/Bench, arXiv 2602.03798](https://arxiv.org/abs/2602.03798), [GitHub](https://github.com/mnluzimu/FullStack-Agent) ; [WebGen-Bench, arXiv 2505.03733](https://arxiv.org/abs/2505.03733), NeurIPS 2025.
- **Description** :
  - **WebGen-Bench** : 101 instructions de sites et 647 cas de test, exécutés par un agent navigateur.
  - **FullStack-Bench** : 101 tâches avec des tests séparés pour le frontend, le backend et la base de données.
- **Licence** : pas de licence déclarée sur les dépôts.
- **Pertinence** : **inspiration**. La décomposition des tests en couches front / back / BDD est à reprendre.

### 2.7 VISTA, UI2App (2026)

- **Liens** : [VISTA, arXiv 2605.26144](https://arxiv.org/abs/2605.26144) ; [UI2App, arXiv 2607.06306](https://arxiv.org/abs/2607.06306).
- **Description** :
  - **VISTA** : maquettes de design transformées en apps full-stack (18 applications). Les agents rendent mieux le design qu'ils n'implémentent le comportement.
  - **UI2App** : inférence des interactions à partir de l'UI.
- **Licence** : **[non vérifié]**.
- **Pertinence** : **inspiration seulement**.

### 2.8 Plus anciens : DevBench, ProjectEval, WebApp1K

- **DevBench** ([arXiv 2403.08604](https://arxiv.org/abs/2403.08604), [GitHub](https://github.com/open-compass/DevBench), Apache-2.0, **inactif depuis 2024**) :
  - Couvre tout le cycle en cascade : design (PRD → UML/archi), environnement, implémentation, tests d'acceptation, tests unitaires.
  - Pertinence : **réutiliser la méthodologie** de découpage par phase si l'on veut noter un workflow phase par phase.
- **ProjectEval** ([arXiv 2503.07010](https://arxiv.org/abs/2503.07010), ACL Findings 2025) :
  - Trois niveaux d'entrée : prompt en langage naturel, checklist de fonctionnalités, squelette de code.
  - Évaluation par simulation d'utilisateur (UI ou console).
  - Pertinence : **inspiration**. Faire varier le niveau de détail de la spec est une dimension intéressante pour P1.
- **WebApp1K** ([arXiv 2408.00019](https://arxiv.org/abs/2408.00019), [GitHub onekq/WebApp1k](https://github.com/onekq/WebApp1k), MIT) :
  - 1 000 problèmes React où **les tests servent de prompt** (approche TDD).
  - Pertinence : **inspiration** (un workflow « tests d'abord » est à comparer).

### 2.9 Étude empirique proche de P1

- **Generation of Web Apps with Agentic IDEs** ([arXiv 2608.20903](https://arxiv.org/abs/2608.20903), août 2026).
- **Description** : Copilot, Cursor et Windsurf génèrent 5 apps full-stack de zéro.
  - Les patterns courants (CRUD, auth) sont bien gérés ; les architectures moins communes (file de tâches) génèrent beaucoup plus d'erreurs.
  - Les écarts entre IDE sont faibles.
  - Pas d'artefacts publiés.
- **Pertinence** : **inspiration**, et confirmation que l'app de référence doit contenir **au moins une préoccupation non triviale** (jobs asynchrones, concurrence, migrations) pour discriminer les workflows.

---

## 3. Harnais d'évaluation (terminal / long horizon) exécutables en local

### 3.1 Harbor (Laude Institute, créateurs de Terminal-Bench) : recommandé

- **Liens** : [GitHub laude-institute/harbor](https://github.com/laude-institute/harbor) ; [docs](https://docs.harborframework.com/) ; [Harbor Hub](https://hub.harborframework.com).
- **Description** : « lancer n'importe quel agent avec n'importe quel modèle sur n'importe quelle tâche, dans n'importe quel sandbox, en parallèle ». Exécution **locale sous Docker** par défaut, ou dans le cloud (Daytona, Modal).
- **Format de tâche** :
  - `instruction.md` et `task.toml` ;
  - `environment/` : Dockerfile ou docker-compose multi-conteneurs ;
  - `solution/solve.sh` : solution de référence, que l'agent `oracle` exécute ;
  - `tests/test.sh`, qui écrit la récompense dans `/logs/verifier/reward.txt` ou dans un `reward.json` multi-dimensionnel.
  - Les tâches peuvent être **multi-étapes** (`steps/step-N/`), ce qui correspond à une chaîne de demandes dépendantes.
- **Agents pré-intégrés (environ 40)** : notamment `claude-code`, `codex`, `gemini-cli`, `cursor-cli`, `copilot-cli`, `opencode`, `aider`, `goose`, `openhands`, `mini-swe-agent`, `swe-agent`, `terminus-2`, `oracle`, `nop`. On peut aussi ajouter des agents ACP ou des agents personnalisés.
- **Fonctions clés pour ce projet** :
  - `--skill <chemin|org/repo@ref>` **injecte des skills** (`SKILL.md`) dans Claude Code ou Codex. Le fichier lock du job enregistre le digest et le SHA de chaque skill, ce qui rend possible un **A/B testing de skills** reproductible ([doc](https://docs.harborframework.com/jobs/skills.md)).
  - « **Simulate a user** » : un agent « utilisateur » qui connaît la tâche dialogue avec l'agent cible, qui ne la connaît pas, via ACP. Cibles supportées : claude-code, codex, gemini-cli, opencode ([doc](https://docs.harborframework.com/jobs/simulate-a-user.md)).
  - **RewardKit** : rubriques en TOML avec trois types de juges :
    - un juge LLM ;
    - un **juge agent** (claude-code ou codex, qui explore le dépôt) ;
    - un juge JEV, rapide et sans raisonnement.

    Les critères peuvent être binaires, Likert, numériques ou par niveaux de rubrique, avec des poids. Le résultat se combine avec les tests ([doc](https://docs.harborframework.com/rewardkit/judge-criteria.md)).
  - **ATIF** : format JSON standard des trajectoires. Il existe aussi une commande de re-notation (`regrade`).
- **Registre de jeux de tâches** : Terminal-Bench, SWE-bench Pro V2, SWE-rebench, SWE-bench, Aider Polyglot, etc. (`harbor datasets list`).
- **Licence** : Apache-2.0. Très actif (push le 2026-11-02).
- **Pertinence** : **réutiliser tel quel** comme ossature d'exécution pour P2, et en bonne partie pour P1. Anthropic le cite comme harnais adapté aux évaluations conteneurisées ([Demystifying evals](https://anthropic.com/engineering/demystifying-evals-for-ai-agents)).

### 3.2 Terminal-Bench

- **Liens** : [tbench.ai](https://www.tbench.ai/benchmarks) ; [arXiv 2601.11868](https://arxiv.org/abs/2601.11868) ; [GitHub laude-institute/terminal-bench](https://github.com/laude-institute/terminal-bench).
- **Versions** :
  - **2.0** (nov. 2025) : 89 tâches vérifiées par des humains, retenues sur 229 proposées.
  - **2.1** : mai 2026.
  - **3.0** : juillet 2026.
  - **4.0** (28/08/2026) : passage au versionnage sémantique et à un benchmark « continu ». Retrait de tâches saturées ou dont la solution est publique, timeout de 8 h par tâche. La 4.1 annonce des vérificateurs résistants à la falsification.
  - Lancement : `harbor run -d terminal-bench/terminal-bench@4.0.0`.
- **Licence** : Apache-2.0.
- **Pertinence** : **réutiliser tel quel** comme contrôle de calibration (vérifier qu'un agent ou une configuration est « normal » sur un benchmark public). Sa **méthodologie de vérification des tâches** est aussi à reprendre.

### 3.3 Inspect AI (UK AISI) + inspect_swe + inspect_evals

- **Liens** : [Inspect](https://inspect.aisi.org.uk/) ; [GitHub UKGovernmentBEIS/inspect_ai](https://github.com/UKGovernmentBEIS/inspect_ai) ; [inspect_evals](https://github.com/UKGovernmentBEIS/inspect_evals) ; [inspect_swe](https://meridianlabs-ai.github.io/inspect_swe/).
- **Description** : framework d'évaluation en Python avec des sandboxes Docker, des limites de tokens et de temps, et un visualiseur de logs.
  - `inspect_swe` fournit les agents `claude_code()`, `codex_cli()`, `gemini_cli()`, `opencode()`, `kimi_code()` et `mini_swe_agent()`.
  - Ces agents tournent dans le sandbox, et leurs appels API sont proxifiés vers Inspect via `sandbox_agent_bridge()`, ce qui permet d'utiliser n'importe quel modèle.
  - `inspect_evals` contient de nombreux benchmarks prêts à l'emploi.
- **Licence** : MIT (les trois dépôts).
- **Pertinence** : **réutiliser tel quel** comme alternative à Harbor si l'on préfère Python « pur ». Plus orienté recherche et sécurité, moins orienté « tâches dans des dépôts ». METR y a migré son outillage.

### 3.4 METR : Task Standard, Vivaria, HCAST, Time Horizons

- **Liens** : [Task Standard](https://github.com/METR/task-standard) ; [Vivaria](https://github.com/METR/vivaria) ; [HCAST, arXiv 2503.17354](https://arxiv.org/abs/2503.17354) ; [Measuring AI Ability to Complete Long Tasks, arXiv 2503.14499](https://arxiv.org/abs/2503.14499) ; [Time Horizon 1.1](https://metr.org/blog/2026-01-29-time-horizon-1-1/) ; [eval-analysis-public](https://github.com/METR/eval-analysis-public).
- **Description** :
  - **Statut** : les dépôts Task Standard et Vivaria sont **archivés**. METR migre vers Inspect.
  - **HCAST** : tâches calibrées sur le temps qu'y mettent des humains.
  - **Time horizon** : durée de tâche humaine à laquelle un agent réussit avec 50 % de probabilité. On l'obtient par une régression logistique sur le log du temps humain. La version 1.1 (janvier 2026) compte 228 tâches.
- **Licence** : MIT.
- **Pertinence** : **réutiliser la méthodologie** :
  - **mesurer le temps humain de référence** pour chaque tâche de P2 ;
  - exprimer les résultats en « horizon » plutôt qu'en simple taux de réussite.

  Les outils eux-mêmes sont à éviter (archivés).

### 3.5 OpenHands (agent + benchmarks + Index)

- **Liens** : [GitHub OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) ; [OpenHands/benchmarks](https://github.com/OpenHands/benchmarks) ; [blog Index](https://openhands.dev/blog/analyzing-and-improving-openhands-index).
- **Description** : agent open source, avec un dépôt de benchmarks basé sur l'Agent SDK V1 : SWE-bench, SWT-bench, Commit0, GAIA. L'« OpenHands Index » combine issues, greenfield (Commit0), front-end (Multimodal) et tests, et publie ses traces.
- **Licence** : MIT.
- **Pertinence** : **inspiration** pour la combinaison de plusieurs axes en un index. OpenHands peut aussi servir d'agent à comparer, via Harbor.

### 3.6 mini-swe-agent (équipe SWE-agent)

- **Liens** : [GitHub SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) ; mini-swe-agent.com.
- **Description** : agent d'environ 100 lignes qui n'utilise que bash (pas de tool calling, historique linéaire).
  - Plus de 74 % sur SWE-bench Verified.
  - Utilisé comme **échafaudage neutre** pour le classement bash-only et pour DeepSWE.
  - Un pilote DeepSWE rapporte qu'il égale les harnais natifs (Claude Code, Codex) à coût de tokens comparable.
  - Version 2.
- **Licence** : MIT.
- **Pertinence** : **réutiliser tel quel** comme **baseline** : « modèle nu avec un harnais minimal » contre « agent outillé avec skills ». Cela isole l'apport du harnais et des skills.

### Tableau récapitulatif des harnais

| Harnais | Docker local | Agents CLI (Claude Code, Codex…) | Agent personnalisé | Injection de skills | Utilisateur simulé | Juge LLM intégré | Licence |
|---|---|---|---|---|---|---|---|
| Harbor | oui (défaut) | oui (~40 pré-intégrés) | oui (+ ACP) | **oui** (`--skill`) | **oui** | oui (RewardKit) | Apache-2.0 |
| Inspect AI + inspect_swe | oui | oui (6 agents) | oui | non documenté | possible (code) | oui (scorers) | MIT |
| OpenHands benchmarks | oui | non (OpenHands seulement) | non | non | non | non | MIT |
| mini-swe-agent | oui | non (c'est un agent) | n/a | non | non | non | MIT |
| `claude plugin eval` | sandbox OS (pas Docker) | Claude Code seulement | non | **oui** (plugin) | non (mais reprise d'historique) | oui | propriétaire (CLI) |
| promptfoo | non (conteneur recommandé) | Claude Agent SDK, Codex SDK, OpenCode | oui | oui (Claude) | non | oui (`llm-rubric`) | MIT |

---

## 4. Évaluer des workflows humain + IA et l'expérience développeur

### 4.1 METR : essai randomisé contrôlé (RCT) 2025 et refonte 2026

- **Liens** : [arXiv 2507.09089](https://arxiv.org/abs/2507.09089) ; [mise à jour du 24/02/2026](https://metr.org/blog/2026-02-24-uplift-update/).
- **Étude de 2025** : 16 développeurs open source expérimentés, 246 tâches réelles sur leurs propres dépôts, randomisation **au niveau de la tâche** (IA autorisée ou interdite).
  - Résultat : **+19 % de temps avec l'IA** (IC 95 % : +2 % à +39 %).
  - Pourtant, les développeurs pensaient avoir gagné environ 20 %.
- **Suivi 2025-2026** : 57 développeurs, plus de 800 tâches.
  - Estimations : −18 % de temps pour les développeurs revenants, −4 % pour les nouveaux.
  - METR juge ces données **peu fiables** :
    - biais de sélection : les développeurs refusent de travailler sans IA ;
    - 30 à 50 % des développeurs ont écarté les tâches qu'ils ne voulaient pas faire sans IA ;
    - qualité différente entre les conditions ;
    - suivi du temps impossible avec plusieurs agents en parallèle.
  - Pistes de refonte : tâches **fixes** imposées, randomisation par développeur, télémétrie et transcripts, études d'usage du temps.
- **[Non vérifié]** Une enquête METR de 2026 sur l'« uplift » auto-déclaré (médiane ×3 en vitesse, moins en valeur) n'est connue que par des sources secondaires.
- **Pertinence** : **réutiliser la méthodologie**, surtout ses leçons, très directement applicables à P1 :
  - imposer des **tâches fixes** ;
  - mesurer le temps objectivement (télémétrie) plutôt que par auto-déclaration ;
  - contrôler la **qualité** du résultat, pas seulement la vitesse ;
  - se méfier de l'effet d'apprentissage quand la même personne refait l'app plusieurs fois.

### 4.2 Autres essais contrôlés

- **Google RCT** ([arXiv 2410.12944](https://arxiv.org/abs/2410.12944), ICSE SEIP 2025) : 96 ingénieurs, environ 21 % de temps en moins, intervalle de confiance large. Outillage interne de l'été 2024.
- **Cui, Demirer et al.** ([SSRN 4945566](https://papers.ssrn.com/abstract=4945566)) : trois expériences de terrain (Microsoft, Accenture, une entreprise du Fortune 100), 4 867 développeurs, +26 % de tâches complétées (erreur type 10 %). Gains plus forts chez les juniors.
- **RealHumanEval** ([arXiv 2404.02806](https://arxiv.org/abs/2404.02806)) : plateforme web d'évaluation avec des humains dans la boucle (243 participants). Les gains sur les benchmarks statiques se traduisent en gains humains **non proportionnels**, et les préférences déclarées ne suivent pas la performance réelle.
- **Pertinence** : **inspiration**. Ces études confirment l'écart entre perception et mesure, ce qui impose des mesures objectives.

### 4.3 Frameworks de mesure : DORA, SPACE, DX

- **DORA 2025 – State of AI-assisted Software Development** ([rapport](https://cloud.google.com/devops/state-of-devops), [PDF](https://services.google.com/fh/files/misc/2025_state_of_ai_assisted_software_development.pdf)) :
  - Environ 5 000 répondants. L'IA y est décrite comme un « **amplificateur** ».
  - Le **DORA AI Capabilities Model** ([dora.dev](https://dora.dev/research/ai/ai-capabilities-model)) identifie 7 pratiques : politique IA claire, écosystème de données sain, données internes accessibles à l'IA, gestion de versions solide, petits lots, focus utilisateur, plateforme interne de qualité.
  - **Pas de rapport annuel 2026.** DORA a publié à la place un rapport « ROI of AI-assisted Software Development » (v2026.1, avril 2026) ([dora.dev/ai/roi](https://dora.dev/ai/roi)). L'arrêt de l'enquête annuelle est rapporté par [LeadDev](https://leaddev.com/?p=36788) **[non vérifié dans une source DORA]**.
- **SPACE** (Forsgren et al., ACM Queue 2021) : cinq dimensions (Satisfaction, Performance, Activity, Communication, Efficiency/flow). On en retient l'idée de ne jamais se fier à une seule métrique.
- **DX AI Measurement Framework** ([getdx.com](https://getdx.com/ai-measurement-framework/)) : trois axes, **Utilisation / Impact / Coût**, par exemple un « coût horaire de l'IA ». Cadre propriétaire, fondé en partie sur de l'auto-déclaratif.
- **Pertinence** : **inspiration**. Ces cadres sont faits pour des organisations, pas pour un benchmark. Pour P1, on retient trois familles de mesures :
  - **coût** : tokens, $, temps machine ;
  - **effort humain** : temps actif, nombre d'interventions ;
  - **qualité livrée** : tests, sécurité, maintenabilité.

### 4.4 Mesurer concrètement un workflow : télémétrie des agents

- **Claude Code OpenTelemetry** ([doc](https://code.claude.com/docs/en/monitoring-usage.md)).
  - **Métriques** :
    - `claude_code.token.usage` et `cost.usage` ;
    - `active_time.total` : temps **utilisateur** (`user`) ou **CLI** (`cli`) ;
    - `lines_of_code.count`, `commit.count`, `pull_request.count` ;
    - `code_edit_tool.decision` : acceptations et rejets.
  - **Événements** : `user_prompt`, `tool_decision`, `tool_result`, `api_request`, corrélés par `prompt.id`.
  - **Activation** : `CLAUDE_CODE_ENABLE_TELEMETRY=1` et un exporteur OTLP local (par exemple un collecteur OTel et Prometheus dans Docker).
  - Codex CLI propose aussi une télémétrie OTel **[non vérifié en détail]**.
- **Étude interne Anthropic** ([How AI is transforming work at Anthropic](https://www.anthropic.com/research/how-ai-is-transforming-work-at-anthropic), déc. 2025) : 132 ingénieurs interrogés, 53 entretiens, analyse de transcripts Claude Code. Environ 27 % du travail assisté n'aurait pas été fait sans l'IA (chiffre rapporté par la presse).
- **Pertinence** : **réutiliser tel quel**. La télémétrie OTel donne gratuitement le nombre d'**interventions humaines** (prompts utilisateur), le temps actif humain, les tokens et le coût de chaque run de workflow. Il ne manque qu'à définir une taxonomie des interventions : clarification, correction, approbation, reprise manuelle.

---

## 5. LLM-as-judge pour la qualité du code

### 5.1 Biais connus

- **Position, verbosité, auto-préférence** : biais identifiés dans MT-Bench (Zheng et al., [arXiv 2306.05685](https://arxiv.org/abs/2306.05685)).
  - L'auto-préférence est liée à la **faible perplexité** : le juge favorise les textes qui lui sont « familiers » ([Wataoka et al., arXiv 2410.21819](https://arxiv.org/abs/2410.21819)).
  - Concrètement, un juge Claude risque de favoriser du code écrit par Claude.
- **Biais spécifiques au code** :
  - **Bias in the Loop** ([arXiv 2604.16790](https://arxiv.org/abs/2604.16790), avril 2026) : à code identique, de simples indices dans le prompt changent les verdicts, parfois au point de **réordonner les classements de modèles**. Les auteurs recommandent de publier la sensibilité aux biais.
  - **Crupi et al.** ([arXiv 2507.16587](https://arxiv.org/abs/2507.16587), IEEE TSE) : même le meilleur juge (GPT-4-turbo) « se trompe fréquemment » sur la correction du code. Les petits modèles sont inadaptés.
- **Pertinence** : **réutiliser la méthodologie**. Le juge LLM ne doit **jamais** décider de la correction fonctionnelle : c'est le rôle des tests. On le réserve aux dimensions non testables (lisibilité, architecture, respect des conventions, qualité des tests écrits).

### 5.2 Approches et bonnes pratiques

- **Anthropic, « Demystifying evals for AI agents »** ([lien](https://anthropic.com/engineering/demystifying-evals-for-ai-agents), 9 jan. 2026) :
  - combiner des graders de code, de modèle et humains ;
  - noter le **résultat** plutôt que le chemin suivi, avec crédit partiel ;
  - **pass@k** (au moins un succès sur k essais) contre **pass^k** (k succès sur k, pour mesurer la constance) ;
  - isoler chaque essai dans un environnement propre ;
  - **lire les transcripts** ;
  - donner au juge une option « Unknown » et **un juge par dimension** ;
  - calibrer sur des notes humaines ;
  - commencer avec 20 à 50 tâches ;
  - un score de 0 % signale souvent une tâche cassée.
- **Agent-as-a-Judge / DevAI** ([arXiv 2410.10934](https://arxiv.org/abs/2410.10934), ICML 2025 ; [GitHub metauto-ai/agent-as-a-judge](https://github.com/metauto-ai/agent-as-a-judge), MIT) :
  - Un **agent** juge, qui explore le dépôt et lance des commandes, évalue 55 tâches et 365 exigences hiérarchiques (DAG).
  - Accord avec les humains : environ 90 %, contre environ 70 % pour un juge LLM simple.
  - Pertinence : **réutiliser la méthodologie**. C'est le modèle à suivre pour P1 : une **checklist d'exigences hiérarchique** vérifiée par un agent juge. RewardKit de Harbor fournit un « agent judge » équivalent.
- **Jury multi-modèles (PoLL)** ([arXiv 2404.18796](https://arxiv.org/abs/2404.18796), Cohere) : trois petits juges de familles différentes font mieux qu'un seul grand juge (moins de biais intra-famille, moins cher).
  - Pertinence : **réutiliser la méthodologie**. Utiliser un jury Claude + GPT + Gemini (ou un modèle open source) quand on compare des workflows qui reposent sur Claude.
- **Garde-fous pratiques** :
  - comparaison **par paires** avec permutation de l'ordre ;
  - rubriques **PASS/FAIL concrètes** plutôt que des notes 1 à 10 ;
  - anonymisation (retirer les commentaires et fichiers qui révèlent l'agent : `CLAUDE.md`, `AGENTS.md`, messages de commit) ;
  - plusieurs échantillons du juge ;
  - le `guard` de RewardKit, qui détecte les instructions injectées dans les fichiers notés.
- **Métriques déterministes complémentaires** : analyse statique, complexité, duplication (SonarQube Community, Semgrep), couverture de tests, scans de sécurité, migrations réversibles. Ce sont des signaux moins biaisés qu'un juge.

---

## 6. Applications de référence avec tests d'acceptation existants

### 6.1 RealWorld / Conduit : candidat n°1

- **Liens** : [GitHub realworld-apps/realworld](https://github.com/realworld-apps/realworld) ; [docs.realworld.show](https://docs.realworld.show) ; [implémentations sur CodebaseShow](https://codebase.show/projects/realworld).
- **Description** : clone de Medium, avec plus de 100 implémentations front et back interchangeables. Le dépôt (vérifié le 2026-11-02) contient :
  - `specs/api/openapi.yml` ;
  - une suite de tests d'API **Hurl**, qui est la source de vérité : auth, articles, comments, favorites, feed, pagination, profiles, tags, et cas d'erreur d'autorisation ;
  - une collection **Bruno** générée à partir de Hurl, avec les scripts `run-api-tests-hurl.sh` et `run-api-tests-bruno.sh` (variable `HOST`) ;
  - des tests E2E **Playwright** partagés pour les frontends dans `specs/e2e/` (auth, articles, social, settings, navigation, sécurité XSS…), avec un **contrat de sélecteurs** (`SELECTORS.md`) que toute implémentation doit respecter.
- **Licence** : MIT. Très actif (push le 2026-11-02). Le dépôt contient même un `CLAUDE.md`.
- **Pertinence** : **réutiliser tel quel** comme base de spec et de tests d'acceptation pour P1. **Limites** :
  1. **Contamination massive**, puisque des centaines d'implémentations sont publiques : un agent peut « réciter » plutôt que concevoir.
  2. Il manque des préoccupations de production : rôles et permissions, migrations à faire évoluer, jobs asynchrones, upload, rate limiting, observabilité, déploiement.

  → À utiliser comme **socle**, enrichi d'**extensions privées** (nouvelles fonctionnalités, avec leurs tests Hurl et Playwright cachés) pour mesurer autre chose que la mémoire.

### 6.2 TodoMVC

- **Liens** : [GitHub tastejs/todomvc](https://github.com/tastejs/todomvc).
- **Description** : `app-spec.md` et des tests E2E Cypress communs à des dizaines de frameworks.
- **Licence** : MIT (license.md). Actif.
- **Pertinence** : **inspiration seulement**. Le périmètre est trop petit, front seulement, et l'app est sur-représentée dans les données d'entraînement : un billet ([dev.to](https://dev.to/shadbb/every-llm-writes-the-same-todo-app-and-i-mean-that-literally-721)) montre que tous les LLM produisent la même app, ce qui en fait un test de mémoire **[anecdotique]**. Utile tout au plus comme tâche d'échauffement ou test de fumée du harnais.

### 6.3 Autres candidats

- **Spring PetClinic** ([spring-petclinic-rest](https://github.com/spring-petclinic/spring-petclinic-rest), Apache-2.0, actif) : application de référence Java ou Spring, avec des variantes REST, Angular et microservices.
  - Pertinence : **inspiration**, si l'on veut une stack Java. Les tests d'acceptation sont moins standardisés que RealWorld.
- **Zeeter / Breathing app de Vibe Code Bench** : specs d'une page, publiées en gist (voir §2.1).
  - Pertinence : **inspiration** pour le format de spec courte orientée comportement.
- **Scénarios BaxBench** (§2.5) : specs de backends avec exploits.
  - Pertinence : **réutiliser tel quel** pour la couche sécurité.

---

## 7. Outils existants pour comparer des workflows, skills et prompts d'agents de code

### 7.1 `claude plugin eval` (Claude Code ≥ 2.1.269)

- **Liens** : [doc](https://code.claude.com/docs/en/plugin-evals.md).
- **Description** :
  - **Cas** : un `prompt.md` (avec `max_turns`, `model`, `allowed_tools`) et des graders `regex`, `tool_used`, `tool_order`, `file_exists`, `llm` (rubrique) et `baseline`. Il n'y a pas de grader en code personnalisé.
  - **Exécution** : 3 runs par défaut, avec un bras **avec plugin** et un bras **sans plugin**. Le résultat donne un score « avec », un score « sans » et un **Δ**.
  - **Fonctions annexes** : juge configurable (`--judge-model`), mocks MCP, reprise d'un historique `.jsonl`, plafond `--max-cost-usd`, rapport JSON et HTML, seuil pour la CI.
  - **Isolation** : sandbox OS (bubblewrap sous Linux), pas Docker.
  - **Variante** : le plugin skill-creator a son propre format d'évaluation (`evals/evals.json`), non compatible.
- **Licence** : propriétaire (CLI Anthropic).
- **Pertinence** : **réutiliser tel quel** pour l'évaluation **micro** des skills et plugins (déclenchement, comportement sur des prompts courts). Inadapté aux tâches longues dans un dépôt Docker : pour cela, utiliser Harbor.

### 7.2 promptfoo

- **Liens** : [guide coding agents](https://www.promptfoo.dev/docs/guides/evaluate-coding-agents/) ; [guide agent skills](https://www.promptfoo.dev/docs/guides/test-agent-skills/) ; [GitHub](https://github.com/promptfoo/promptfoo).
- **Description** :
  - **Providers** : `anthropic:claude-agent-sdk`, Codex SDK, OpenCode SDK. Ils permettent de comparer deux versions d'une skill (filtre `skills:`) ou deux `CLAUDE.md`.
  - **Assertions** : trajectoire (`tool-used`, `tool-sequence`, nombre d'étapes), coût, latence, `llm-rubric`.
  - `copy_working_dir` crée un espace de travail par essai. Ce n'est **pas** un sandbox : un conteneur est recommandé.
- **Licence** : MIT. **Racheté par OpenAI en mars 2026**, avec l'engagement de rester open source ; la neutralité multi-fournisseurs reste à surveiller.
- **Pertinence** : **réutiliser tel quel** pour des matrices rapides prompt × skill × modèle. Moins adapté que Harbor aux tâches longues.

### 7.3 SkillsBench

- **Liens** : [arXiv 2602.12670](https://arxiv.org/abs/2602.12670) ; [GitHub benchflow-ai/skillsbench](https://github.com/benchflow-ai/skillsbench) ; [HF dataset](https://huggingface.co/datasets/benchflow/skillsbench).
- **Description** : 86 tâches (87 en v1.1) dans 11 domaines, chacune évaluée selon trois conditions : sans skill, avec skills curées, avec skills auto-générées.
  - Les skills curées apportent en moyenne +16 points, mais **seulement environ +4,5 points en génie logiciel**.
  - Les skills auto-générées n'apportent **rien** en moyenne.
  - Codex CLI ignore souvent les skills fournies **[source secondaire]**.
- **Licence** : Apache-2.0.
- **Pertinence** : **réutiliser la méthodologie** (protocole à trois bras). Les résultats sont aussi des **priors** pour P2 : il faut s'attendre à des effets de skills faibles en développement, donc prévoir assez de répétitions.

### 7.4 « Evaluating AGENTS.md » (ETH / LogicStar)

- **Liens** : [arXiv 2602.11988](https://arxiv.org/abs/2602.11988) ; [GitHub eth-sri/agentbench](https://github.com/eth-sri/agentbench) (MIT).
- **Description** : comparaison de trois conditions sur des tâches SWE (AGENTbench) : sans fichier de contexte, avec un fichier généré par LLM, avec un fichier écrit par les développeurs.
  - Les fichiers générés **réduisent** le taux de réussite et augmentent le coût de plus de 20 %.
  - Les fichiers écrits par des humains aident légèrement.
- **Pertinence** : **réutiliser la méthodologie**. C'est exactement le protocole à appliquer pour comparer des `CLAUDE.md` ou des skills dans P2.

### 7.5 Plateformes d'observabilité et d'évaluation (SaaS)

- **Braintrust** ([braintrust.dev](https://www.braintrust.dev/) ; lib [autoevals](https://github.com/braintrustdata/autoevals), MIT).
- **LangSmith** ([langchain.com/langsmith](https://www.langchain.com/langsmith), propriétaire).
- **Langfuse** ([GitHub](https://github.com/langfuse/langfuse), cœur MIT et parties enterprise sous licence distincte, auto-hébergeable sous Docker).
- **Arize Phoenix** : non examiné.
- **Pertinence** : **inspiration**, ou **Langfuse tel quel** si l'on veut un tableau de bord auto-hébergé pour les traces et les scores. Aucune de ces plateformes ne fournit l'exécution conteneurisée d'agents CLI : elle se fait avec Harbor ou Inspect. **[Intégrations Claude Code / Codex de ces plateformes : non vérifiées en détail.]**

### 7.6 Existe-t-il déjà « exactement » ce projet ?

Aucun outil open source trouvé ne compare des **workflows humain + agent complets** sur une **app de référence personnelle** avec des tests d'acceptation cachés. Les briques les plus proches :

- **Vals Vibe Code Bench**, pour P1 : fermé.
- **Harbor**, avec `--skill`, l'utilisateur simulé, les tâches multi-étapes et RewardKit, pour P2 et en partie P1.
- **`claude plugin eval`**, pour les skills isolées.

Le projet comble donc un vrai manque, surtout sur la dimension « humain dans la boucle ».

---

## 8. Synthèse : ce qu'on peut réutiliser

1. **Adopter le format de tâche Harbor comme ossature (P2, et P1 en mode multi-étapes).**
   - Chaque tâche contient `instruction.md`, `environment/` (docker-compose : app + BDD), `tests/test.sh` qui écrit `reward.json` multi-critères, et `solution/solve.sh` (la PR de référence).
   - On obtient gratuitement l'exécution locale sous Docker de Claude Code, Codex, Gemini CLI, OpenCode, mini-swe-agent, etc., le parallélisme, les trajectoires ATIF et la compatibilité avec SWE-bench Pro V2 et Terminal-Bench comme points de comparaison externes.

2. **Construire les tâches P2 « à la SWE-bench », mais privées et anti-fuite.**
   - Chaque issue = commit de base + PR de référence ; les tests sont répartis en FAIL_TO_PASS et PASS_TO_PASS et injectés seulement au moment de la vérification.
   - Livrer le dépôt **sans historique futur** (commit orphelin ou `--depth 1`) et bloquer le réseau si possible.
   - Valider chaque tâche avec l'agent `oracle` (score de 1 attendu) et l'agent `nop` (score de 0), puis exécuter trois fois pour détecter le flakiness, comme le fait DeepSWE.
   - Compléter avec des bugs injectés à la manière de SWE-smith.

3. **Prendre RealWorld/Conduit comme socle de l'app de référence pour P1, avec des extensions privées.**
   - RealWorld fournit OpenAPI, la suite Hurl/Bruno pour l'API et Playwright avec contrat de sélecteurs pour le front, sous licence MIT.
   - Ajouter des **exigences privées de « production »** : rôles, migrations évolutives, job asynchrone, rate limiting, upload, healthchecks, et des scénarios d'exploits à la BaxBench. Chacune vient avec des tests cachés, pour mesurer autre chose que la récitation d'une app très présente dans les données d'entraînement.

4. **Noter P1 en couches, du plus objectif au plus subjectif.**
   1. Tests d'acceptation cachés : API (Hurl) et E2E (Playwright).
   2. Contrôles non fonctionnels déterministes : sécurité (exploits, Semgrep), analyse statique, couverture, migrations up/down, `docker compose up` depuis zéro.
   3. Checklist d'exigences hiérarchique notée par un **agent juge** (Agent-as-a-Judge ou RewardKit `agent judge`), avec un jury multi-familles, des rubriques PASS/FAIL et une permutation de l'ordre.
   4. Revue humaine sur un échantillon, pour calibrer le juge.

5. **Mesurer le coût et l'effort humain par télémétrie, pas par auto-déclaration** (leçon METR).
   - Exporter l'OpenTelemetry de Claude Code (et l'équivalent Codex) vers un collecteur local : tokens, $, temps actif `user` et `cli`, nombre de prompts utilisateur (= interventions), décisions d'outils.
   - Définir une taxonomie des interventions : clarification, correction, approbation, reprise manuelle.
   - Imposer des **tâches fixes** et randomiser l'ordre des workflows pour limiter l'effet d'apprentissage.

6. **Évaluer la maintenabilité par des chaînes de demandes dépendantes.**
   - S'inspirer de Web-Bench et de Vibe Code Bench 1-100 : après la construction initiale, enchaîner N demandes d'évolution (tâches Harbor multi-étapes) et mesurer la dégradation.
   - C'est là que les workflows (spec-driven, TDD, skills) devraient se différencier le plus.

7. **Comparer skills et prompts avec un protocole à bras multiples et des statistiques honnêtes.**
   - Pour la tâche dans un dépôt : bras sans skill, avec skill curée, avec skill générée (SkillsBench et l'étude AGENTS.md), via `harbor run --skill …@<sha>`. Pour le déclenchement et le comportement micro : `claude plugin eval` (Δ avec ou sans plugin).
   - Lancer au moins 3 à 5 essais par configuration, publier **pass@k et pass^k** avec intervalles de confiance, et figer les versions de modèles et d'agents.
   - S'attendre à des effets faibles en génie logiciel.

8. **Automatiser une partie de l'humain avec un utilisateur simulé, puis valider avec de vrais runs humains.**
   - Le mode « simulate a user » de Harbor (inspiré de SWE-Interact) permet de rejouer à bas coût un workflow « humain + agent » avec persona et exigences révélées progressivement.
   - Réserver les sessions humaines réelles, chères et non répétables, à un sous-ensemble, et ajouter la baseline mini-swe-agent « modèle nu » pour isoler l'apport du harnais et du workflow.

---

## 9. Lien avec les décisions du projet

Cette section fait le lien entre la recherche et le cadrage en cours (voir [GLOSSARY.md](../../GLOSSARY.md) et [docs/adr/](../adr/)). Elle est mise à jour à mesure que les décisions sont prises.

| Constat de la recherche | Décision du projet | Statut |
|---|---|---|
| Le standard (SWE-bench) note via des tests cachés, pas par ressemblance à la solution de référence (§1.0) | La correction est donnée par l'Acceptance Suite ; la Reference Implementation n'est qu'un point de comparaison ([ADR-0001](../adr/0001-acceptance-suite-is-the-correctness-oracle.md)) | Décidé |
| Les résultats ne sont comparables qu'à protocole et versions figés (§7, synthèse 7) | Reference versionnée (`ref-vN`), comparaisons seulement au sein d'une même version ([ADR-0002](../adr/0002-runs-compared-only-within-a-reference-version.md)) | Décidé |
| L'infrastructure par tâches (Harbor) est mûre et réutilisable ; P1 n'a pas d'équivalent ouvert (§3.1, §2.1) | L'infrastructure P2 est construite d'abord ; P1 vient ensuite | Décidé |
| L'effet d'apprentissage biaise les essais humains ; METR recommande des tâches fixes et un ordre randomisé (§4.1, synthèse 5) | P1 accepte le biais de familiarité, documenté, pour pouvoir évaluer aussi la phase de définition (PRD, Spec) | Décidé, **écart assumé** par rapport à la recommandation |
| Format de tâche Harbor, `--skill` versionné (§3.1, synthèse 1 et 7) | Adoption de Harbor pour P2, Skill A/B à l'intérieur d'un Agent figé ([ADR-0003](../adr/0003-adopt-harbor-as-task-harness.md)) | Décidé |
| Tâches privées et anti-fuite, historique git nettoyé, validation oracle/nop (§1.0, synthèse 2) | Tasks extraites de l'historique d'issues du Reference Workflow, Starting State sans historique, validation oracle/nop | Décidé |
| Notation en couches et jury LLM multi-familles (§5, synthèse 4) | Grader déclaré par Task : Test, Rubric, Human. Juges Claude uniquement par défaut, jury multi-familles en option payante ([ADR-0006](../adr/0006-claude-only-judges-by-default.md)) | Décidé, **écart assumé** (biais d'auto-préférence) |
| RealWorld/Conduit recommandé comme socle (synthèse 3) mais très présent dans les données d'entraînement (§6.1) | Produit original reprenant la *méthode* de Conduit (OpenAPI, Hurl, Playwright + contrat de sélecteurs) ([ADR-0004](../adr/0004-original-product-with-conduit-method.md)) ; entrée de P1 = Brief + Interface Contract ([ADR-0005](../adr/0005-brief-and-interface-contract-as-workflow-input.md)) | Décidé, **diverge** de la synthèse 3 |
| Télémétrie plutôt qu'auto-déclaration ; pass@k et pass^k ; effets faibles des Skills (§4.4, §7.3, synthèse 5 et 7) | Task Runs : 5 par défaut ; Workflow Run : un seul, mesuré par extraction des journaux de session Claude Code plutôt que par OTel ([ADR-0008](../adr/0008-measure-workflow-runs-from-session-transcripts.md)), plus Run Journal et grille de ressenti | Décidé, **écart assumé** (pas d'OTel) |
| Chaînes de demandes dépendantes pour mesurer la maintenabilité (synthèse 6) | Pas encore abordé | À discuter |
| Utilisateur simulé pour rejouer un workflow humain à bas coût (synthèse 8) | Pas encore abordé | À discuter |

### Limites de cette recherche

- Faite par un agent en une seule passe le 2026-11-02, avec la recherche web. Les points marqués **[non vérifié]** (une dizaine) n'ont pas été confirmés par une source primaire.
- Les éléments récents (2026 : SWE-bench Pro V2, Terminal-Bench 4.0, DeepSWE, ProgramBench, versions de Harbor et de `claude plugin eval`) sont ceux qui risquent le plus d'être inexacts ou d'évoluer. Il faut les revérifier au moment de les adopter.
