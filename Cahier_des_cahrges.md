# Cahier des Charges : LLCP (Libre et Local - Compta Personnelle)

## 1. Vision et Objectifs
Créer une application de comptabilité en partie double, autonome, légère et pérenne. L'outil, nommé **LLCP**, est conçu pour un usage personnel et strictement privé sur ordinateur fixe. Il n'est pas destiné à être publié ni déployé sur un serveur web.

## 2. Règles Métier
* **Structure Comptable :**
    * **Écriture :** Regroupe les données transactionnelles (ID auto-incrémenté, Date, intitulé, mode de paiement).
    * **Mouvement :** Unité de base rattachée à une écriture (ID auto-incrémenté, Compte, débit, crédit, libellé complémentaire, lettrage, pointage).
* **Principe de Partie Double :** Toute écriture doit être équilibrée (somme des débits = somme des crédits).
* **Gestion des comptes :** Les numéros de compte sont des entiers codés sur 5 chiffres.
* **Réconciliation :** Chaque mouvement porte ses propres attributs de *lettrage* (état) et de *pointage* (état, numéro d'ordre).
* **Gestion du Plan Comptable :** Ajout dynamique de comptes via interface dédiée. Suppression autorisée uniquement si le compte ne contient aucun mouvement.
* **Intégrité :** Validation conditionnelle à l'équilibre comptable. Utilisation d'identifiants uniques auto-incrémentés pour chaque écriture et chaque mouvement afin de garantir la traçabilité.
* **Confidentialité :** Absence de système de gestion de confidentialité. Les données étant stockées en clair dans un fichier local, la sécurité repose exclusivement sur l'accès physique ou système au fichier.

## 3. Spécifications Techniques
* **Format de stockage :** YAML (multi-documents séparés par `---`).
    * **Version :** Intégration d'un numéro de version dans l'en-tête du fichier pour assurer la gestion de l'évolution du schéma de données.
* **Accès aux données :** API File System Access (lecture/écriture locale).
    * **Restriction de compatibilité :** L'application est conçue exclusivement pour les navigateurs supportant l'API File System Access (Chrome, Edge, Opera).
* **Stratégie de sauvegarde :** Append-only pour les nouvelles écritures, compactage complet à l'ouverture.
* **Atomicité :** Processus Write-Rename pour garantir la sécurité du fichier.

## 4. Fonctionnalités et Interface
* **Disposition :**
    * **Colonne de gauche :** Plan comptable (ajout dynamique via fenêtre flottante).
    * **Bloc central :**
        * **En-tête :** Info compte, bouton "Supprimer" (si vide), sélecteurs (Dates, Année), boutons radio (Lettrage/Pointage), Menu déroulant (Mode : Consultation, Pointage, Lettrage).
        * **Corps :** Tableau des mouvements. Chaque ligne comporte un triangle cliquable pour dérouler l'écriture complète. Tri dynamique par entête.
        * **Navigation :** Pagination classique (ex: 50 ou 100 lignes par page) pour gérer le volume > 10 000 écritures.
        * **Pied de page (liste) :** Solde du compte.
        * **Pied de page (bloc) :** Formulaire libre d'ajout d'écriture (ajout automatique de mouvements par "Enter", autocomplétion par intitulé, validation conditionnelle à l'équilibre).
* **Réconciliation :** Double-clic sur un mouvement pour alterner lettrage ou incrémenter le pointage. Mise à jour automatique de l'affichage selon le mode.
* **Ergonomie :** Interface fonctionnelle minimaliste. Absence totale de guide utilisateur intégré ou d'infobulles contextuelles.
* **Performance :** Support fluide d'un volume > 10 000 écritures via pagination.

## 5. Contraintes de Développement
* **Technologie :** 100% Client-Side (HTML5, CSS3, JS).
* **Déploiement :** Fichier unique (`index.html`). Usage local exclusivement (fixe).
* **Dépendances :** Aucune base de données externe. CDN pour Bootstrap et js-yaml.