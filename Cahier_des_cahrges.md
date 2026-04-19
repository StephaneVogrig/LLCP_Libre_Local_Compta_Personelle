# LLCP (Libre et Locale Compta Personnelle)

**LLCP** est une application de comptabilité personnelle en partie double, conçue pour être minimaliste, autonome et parfaitement privée.

## Philosophie
* **Libre :** Logiciel sous licence GPLv3. Le code source est ouvert et tout projet dérivé doit rester libre, sans aucune restriction d'utilisation.
* **Local :** Vos données restent sur votre ordinateur. Aucun système de tracking, aucune authentification distante, aucune fuite d'information.
* **Minimaliste :** Interface utilitaire brute, sans guide utilisateur superflu ni infobulles, pensée pour l'efficacité pure.

## Fonctionnalités principales
* **Comptabilité en partie double :** Gestion rigoureuse des écritures avec équilibre débit/crédit.
* **Réconciliation bancaire :** Outils intégrés de pointage et de lettrage par mouvement.
* **Plan comptable flexible :** Gestion dynamique des comptes (numérotation sur 5 chiffres).
* **Performance locale :** Conçu pour gérer nativement des volumes importants (plus de 10 000 écritures) via un système de pagination.
* **Format de données ouvert :** Stockage dans un fichier JSON Lines (JSONL), garantissant une lecture séquentielle rapide, une interopérabilité totale et une lisibilité immédiate par tout éditeur de texte standard.

## Architecture des données
* **Indépendance des modèles :** Le format de persistance (JSONL) est distinct du modèle de données en mémoire.
* **Source de vérité :** À l'ouverture, l’ordre des lignes dans le fichier fait foi. Les occurrences les plus récentes d'un même identifiant prévalent sur les précédentes.
* **Cycle de vie de la persistance :**
    * **Ouverture :** Lecture ligne par ligne et chargement en mémoire de l'état consolidé.
    * **Compaction :** Processus de réécriture complète produisant un instantané cohérent de l’état des données. Cette opération est exécutée après consolidation en memoire, le contenu  en mémoire est transféré sur le disque via une stratégie *write-rename* (écriture dans un fichier temporaire puis remplacement atomique de l'original). Les modifications effectuées ensuite sont enregistrées en append-only à la suite de cet instantané, formant un journal des changements de la session en cours.
    * **Usage courant :** Ajout *append-only* (chaque nouvelle écriture est ajoutée à la fin du fichier) pour une latence minimale.
* **Gestion des identifiants :** Lors du chargement, le système détermine la valeur maximale des identifiants existants afin de définir le prochain identifiant à attribuer. Les identifiants déjà présents dans le fichier sont conservés tels quels et ne sont jamais modifiés. Cette approche garantit l’unicité et la stabilité des enregistrements, y compris en cas d’édition manuelle du fichier.
* **Gestion des suppressions :** La suppression d'une écriture ou d'un compte s'effectue par l'ajout d'une ligne d'annulation portant le même identifiant. Cette ligne, d'un type spécifique, neutralise les occurrences précédentes lors du chargement. L'effacement physique des données obsolètes est exclusivement traité lors de l'opération de compaction.
* **Validation par schéma :** Chaque ligne JSON est validée à la lecture et à l'écriture par un schéma rigoureux défini dans le code source. Ce schéma garantit la conformité structurelle des objets (types, champs obligatoires, contraintes métier) avant tout traitement.
* **Versionnement :** Le fichier débute systématiquement par un objet meta indiquant sa version structurelle. Cela permet au parseur d'adapter le traitement des données en cas d'évolution future du format.

## Exemple de structure de données (JSONL)
Chaque ligne du fichier est un objet JSON autonome :
{"type": "meta", "version": 1}

{"type": "compte", "numero": 60600, "intitule": "Marchandises"}
{"type": "compte", "numero": 51200, "intitule": "Banque"}

{"type": "ecriture",
 "id": 1,
 "date": "2026-04-19",
 "libelle": "Achat fournitures",
 "mouvements": [
   {"id": 1, "compte": 60600, "debit": 50, "credit": 0, "lettrage": null, "pointage": null, "detail": ""},
   {"id": 2, "compte": 51200, "debit": 0, "credit": 50, "lettrage": null, "pointage": null, "detail": ""}
 ]
}

{"type": "compte_supprime", "id": 1}
{"type": "ecriture_supprime", "id": 1}

## Spécifications techniques
* **100% Client-Side :** Application monocouche (HTML5, CSS3, JS), zéro dépendance externe.
* **Accès aux données :** Utilisation native de l'API *File System Access* pour une lecture/écriture directe.
* **Compatibilité :** Nécessite un navigateur supportant l'API *File System Access* (Chromium-based : Chrome, Edge, Brave, Opera).
* **Atomicité :**
    * **Phase d'ajoute:**Format JSONL. Une coupure brutale ne corrompt que la ligne en cours d'écriture.
    * **Phase de compactage:**La stratégie *write-rename* protège l'intégrité des données : aucune corruption possible du fichier original en cas d'interruption. En cas de crash pendant cette phase, un fichier temporaire résiduel peut subsister. À l'ouverture suivante, l'application détecte et supprime automatiquement tout fichier temporaire orphelin avant de procéder à la compaction.

## Mise en œuvre
* **Prérequis navigateur :** LLCP requiert impérativement un navigateur Chromium-based (Chrome, Edge, Brave, Opera) pour fonctionner. Firefox et Safari ne supportent pas l'API File System Access et ne peuvent pas faire tourner l'application. Ce choix est délibéré et documenté dans "Périmètre exclus".
1. **Accès :** Ouvrez le fichier `index.html` dans un navigateur compatible (Chromium-based).
2. **Initialisation :** Au premier lancement, sélectionnez le répertoire de travail via l'invite de l'API *File System Access*.
3. **Sauvegarde :** Une simple copie du fichier JSONL suffit pour sauvegarder l'intégralité de votre comptabilité. Il peut également être versionné par Git.

## Règles métiers
* **Intégrité comptable :**
  * **Compte :** Afin de garantir la fiabilité des comptes, la suppression d'un compte est strictement bloquée si le compte contient des mouvements.
  * **Ecritures :** Afin de garantir la fiabilité du lettrage et du rapprochement bancaire, la suppression d'une écriture est strictement bloquée si l'un de ses mouvements est lettré ou pointé. L'utilisateur doit préalablement annuler le lettrage ou le pointage pour rendre l'écriture supprimable.

## Maintenance et limites
* **Validation systématique :** En complément du schéma interne, une routine de contrôle d'intégrité — vérifiant l'équilibre débit/crédit et la cohérence inter-objets — est exécutée automatiquement à l'ouverture du fichier et systématiquement avant chaque opération d'enregistrement sur le disque.
* **Gestion de la corruption :** Lors du chargement, le parseur vérifie la conformité de chaque ligne par rapport au schéma défini dans le code. Toute ligne invalide est détectée et isolée afin de garantir que l'application ne charge que des données comptablement intègres, tout en permettant à l'utilisateur de corriger manuellement les erreurs de syntaxe.
* **Périmètre exclus :** LLCP ne gère pas le multi-devises ni l'import automatique de fichiers OFX/QIF. Ces choix garantissent la simplicité et la pérennité du code.
* **Pérennité :** L'absence de bibliothèques tierces (Zero-Dependency) assure que l'outil restera fonctionnel indéfiniment sans risque d'obsolescence logicielle.
