# 🚀 Procédure de Mise en Production

Ce document récapitule les règles de gouvernance, la chaîne de déploiement et le journal des versions mises en ligne pour l'API de classification de genre cinématographique.

---

## 1. Governance et Rôles

* **Qui décide (Décideur) :** Data Engineer
* **Qui exécute (Opérateur) :** Data Engineer (déploiement manuel contrôlé pour la gestion budgétaire Azure).

---

## 2. Cycle de Mise en Production

### A. Déclencheur
Un commit poussé sur la branche principale (`main`) après la validation complète des tests et vérifications automatiques en CI/CD.

### B. Vérifications Préalables Automatisées (GitHub Actions)
1. Exécution des tests unitaires et d'intégration via `pytest`.
2. Exécution du script de barrière de qualité `quality_gate.py` (vérification que le $F1$-score respecte la limite fixée dans `config.yaml`).
3. Construction (*Build*) et validation de l'image Docker autonome.

### C. Déploiement Manuel sur Cloud Azure
1. Récupération du hash unique du commit stable (`COMMIT_HASH=$(git rev-parse --short HEAD)`).
2. Construction et transfert de l'image conteneurisée vers le registre Azure (**ACR**) :
   ```bash
   docker build -t <REGISTRE_ACR>.azurecr.io/film-classifier-api:$COMMIT_HASH .
   docker push <REGISTRE_ACR>.azurecr.io/film-classifier-api:$COMMIT_HASH
   ```
3. Mise à jour de l'instance **Azure Container Apps (ACA)** :
   ```bash
   az containerapp update \
     --name app-film-classifier \
     --resource-group rg-film-classifier \
     --image <REGISTRE_ACR>.azurecr.io/film-classifier-api:$COMMIT_HASH
   ```

### D. Contrôle Post-Déploiement
* Appel de l'endpoint de santé : `GET https://<URL_AZURE>/health`
* **Vérification obligatoire :** S'assurer que le champ JSON `"commit_hash"` renvoie exactement l'identifiant du commit déployé.
* Validation fonctionnelle sur l'interface Swagger `/docs`.

---

## 3. Conditions et Procédure de Retour Arrière (Rollback)

* **Déclencheurs de Rollback :** 
  * Statut d'erreur HTTP 5xx sur `/predict`.
  * Non-correspondance entre le `commit_hash` retourné par `/health` et la version attendue.
  * Indisponibilité ou latence anormale du service.
* **Procédure de Rollback à la main :**
  1. Identifier le commit stable précédent (`PREVIOUS_COMMIT_HASH`).
  2. Ré-aiguiller immédiatement l'application Azure Container Apps vers l'image précédente conservée sur ACR :
     ```bash
     az containerapp update \
       --name app-film-classifier \
       --resource-group rg-film-classifier \
       --image <REGISTRE_ACR>.azurecr.io/film-classifier-api:$PREVIOUS_COMMIT_HASH
     ```
  3. Valider le retour au vert par un nouvel appel à `/health`.

---

## 4. Journal des Versions Réelles (Version Log)

| Date | Identifiant Commit (SHA) | Changement / Fonctionnalité | Résultat CI/CD | Statut Déploiement | Remarques |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **23/10/2026** | `a1b2c3d` | Initialisation FastAPI & structure Docker | ✅ Succès | 🟢 Déployé | Version initiale locale et test conteneur |
| **26/10/2026** | `e5f6g7h` | Ajout script `quality_gate.py` (F1 > 0.80) | ✅ Succès | 🟢 Déployé | Validation de la pipeline verte (Preuve 3) |
| **26/10/2026** | `890abcd` | Test barrière de qualité (seuil F1 = 0.999) | ❌ Échec | 🔴 Bloqué | Preuve 4 : Blocage automatisé du pipeline |
| **27/10/2026** | `1234ef5` | Correction seuil + Endpoint `/health` avec Hash | ✅ Succès | 🟢 Déployé | Preuve 5 & 6 : API publique en ligne |
| **27/10/2026** | `6789gh0` | Ajout temporaire de la route `/hello` | ✅ Succès | 🟢 Déployé | Preuve 7 (Partie 1 : Évolution) |
| **27/10/2026** | `abcdef1` | Retrait de la route `/hello` (Rollback) | ✅ Succès | 🟢 Déployé | Preuve 7 (Partie 2 : Nettoyage & Rollback) |