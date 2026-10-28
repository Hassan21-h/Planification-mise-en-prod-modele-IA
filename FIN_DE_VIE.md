# 🧹 FIN_DE_VIE.md - Plan et Exécution de la Fin de Vie

Ce document définit la stratégie de fin de vie (partielle et totale) des ressources du projet afin de prévenir les dépenses inutiles sur le cloud Azure et de garantir un recyclage propre de l'environnement.

---

## 1. Distinction Fin de Vie Partielle vs Totale

* **Fin de Vie Partielle (Mise en pause) :**
  Appliquée entre deux sessions de travail ou à la fin d'un sprint. On stoppe le calcul sur Azure Container Apps (passage du nombre de répliques à zéro) tout en conservant le registre d'images (ACR) et le code source sur GitHub. Cela bloque les dépenses de calcul tout en permettant un redéploiement rapide.
* **Fin de Vie Totale (Clôture du projet) :**
  Appliquée à l'issue de la validation des 8 preuves du projet. Elle consiste à supprimer l'intégralité des ressources cloud payantes (ACA, ACR, Resource Group) tout en conservant le code source, la documentation et les artefacts versionnés sur Git.

---

## 2. Inventaire et Stratégie par Composant

| Composant | Nature / Rôle | Localisation | Retrait (Quand & Ordre) | Ce que l'on garde | Coût d'inactivité (si laissé actif) | Coût Réel Constaté |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Application (ACA)** | Service API actif sur Internet | Azure Container Apps | Fin de projet (**Ordre 1**) | Logs d'exécution & Métriques | ~15,00 € / mois | 0,04 € |
| **Registre d'images (ACR)** | Entrepôt des images Docker | Azure Container Registry | Fin de projet (**Ordre 2**) | `Dockerfile` & Recette | ~5,00 € / mois | 0,02 € |
| **Groupe de Ressources** | Conteneur global de ressources | Cloud Azure | Fin de projet (**Ordre 3**) | Aucun composant Cloud | 0,00 € (si vide) | 0,00 € |
| **Secrets GitHub** | Identifiants et clés d'accès | GitHub Secrets | Post-destruction (**Ordre 4**) | Historique des runs CI/CD | 0,00 € | 0,00 € |
| **Modèle & Vectoriseur** | Artefacts du modèle (`.keras`, `.joblib`) | Dossier `model/` | **Conservation permanente** | Fichiers d'artefacts versionnés | 0,00 € (Git / Local) | 0,00 € |
| **Code Source & Workflows** | Code API, `Dockerfile`, CI/CD | Dépôt Git (GitHub) | **Conservation permanente** | Dépôt complet archivé | 0,00 € | 0,00 € |

---

## 3. Ordre Chronologique de Suppression (Fin de Vie Totale)

```
[ 1. Suppression Azure Container App ] ──> [ 2. Suppression Azure Container Registry ] ──> [ 3. Destruction Resource Group ] ──> [ 4. Révocation Secrets Git ]
```

### Procédure et commandes d'exécution :
1. **Suppression de l'Application Azure Container Apps :**
   ```bash
   az containerapp delete --name app-film-classifier --resource-group rg-film-classifier --yes
   ```
2. **Suppression du Registre Azure Container Registry :**
   ```bash
   az acr delete --name acrfilmclassifier --resource-group rg-film-classifier --yes
   ```
3. **Suppression du Groupe de Ressources complet :**
   ```bash
   az group delete --name rg-film-classifier --yes
   ```
4. **Récurage des secrets sur GitHub :**
   Suppression manuelle des identifiants dans *Settings > Secrets and variables > Actions*.

---

## 4. Ce que l'on conserve (Justification)

* **Code source (`/src`) et Dockerfile :** Nécessaires pour garantir la reproductibilité intégrale de l'application.
* **Artefacts du modèle (`model.keras`, `vectoriseur.joblib`) :** Conservés sur Git pour permettre un redéploiement immédiat de l'API sans devoir ré-entraîner le modèle.
* **Documentation & Fichiers de projet (`README.md`, `PLAN.md`, `PROCEDURE_MISE_EN_PRODUCTION.md`, `FIN_DE_VIE.md`) :** Conservés pour traçabilité et évaluation des compétences MLOps (C12).

---

## 5. Bilan Financier et Preuve de Fin de Vie Totale

* **Coût mensuel évité :** ~20,00 € / mois.
* **Coût total réel du projet :** **0,06 €** (Validé sur la facture d'utilisation Azure).
* **Preuve de suppression (Preuve 8/8) :** La commande `az group delete` a été exécutée et confirmée, garantissant qu'aucune ressource facturable n'est restée active sur le sous-compte Azure.