Plan de travail pour la migration BDL → CVC
1. Backend
1.1 Identifier les fichiers à modifier

Rechercher tous les fichiers contenant :

Le nom “BDL” (à remplacer par “CVC”)

Textes principaux affichés aux utilisateurs

Gestion des comptes utilisateurs

1.2 Modifier les fichiers/fonctions

Remplacer “BDL” par “CVC” là où c’est nécessaire.

Mettre en place les identifiants simples (ex. m.vicq) et générer des e-mails fictifs pour Supabase lors de la création des comptes.

Conserver la structure existante du backend.

1.3 Vérification

Tester chaque fichier/fonction modifiée.

Noter et corriger immédiatement tout problème avant de passer au suivant.

1.4 Rôles et permissions

Admin/dev attribue les pouvoirs (créer, modifier, supprimer).

Vérifier que chaque rôle fonctionne correctement.

2. Frontend
2.1 Modifications essentielles

Modifier uniquement les textes principaux et logos dans le code.

CSS et autres styles peuvent être ajustés directement si nécessaire.

Le reste des contenus secondaires pourra être modifié via admin/dev si besoin.

2.2 Vérification

Tester l’affichage des textes principaux et logos après chaque modification.

Vérifier que le rendu visuel et la navigation restent corrects.

3. Vérification continue

Après chaque modification :

Tester localement

Vérifier que tout fonctionne (backend + frontend)

Corriger immédiatement tout problème

Ne passer à l’étape suivante que si tout fonctionne parfaitement.

4. Checklist finale

Backend : fonctionnel, identifiants et e-mails fictifs opérationnels, rôles et permissions corrects.

Frontend : textes principaux et logos modifiés, rendu visuel correct, contenu secondaire modifiable via admin/dev.

Tests globaux : toutes les fonctionnalités principales testées et validées, aucun bug restant.


Bonjour,
Je te fournis un fichier Excel contenant l’arborescence complète de mon projet.
À partir de cette arborescence, je veux que tu me dises exactement quels fichiers et dossiers doivent être modifiés, surtout pour le backend, et quoi modifier dans chaque fichier.

Rappel des points importants :

Backend :

Textes principaux à remplacer par “CVC”

Identifiants simples pour les utilisateurs (ex. m.vicq)

E-mails fictifs générés pour Supabase

Gestion des rôles et permissions

Frontend :

Logos

Textes principaux

CSS si nécessaire

Le reste sera modifiable via admin/dev

Après cette analyse, nous refaisons tout le projet correctement, étape par étape : backend d’abord, puis frontend, en testant après chaque modification.
