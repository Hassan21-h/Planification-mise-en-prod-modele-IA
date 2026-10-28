# 📊 PLAN.md - Planification du Projet de Mise en Production

Ce document contient le tableau de planification prévisionnel et réel pour la mise en production du modèle de classification de films. Le projet s'étend sur **5 jours ouvrés** (du 21 au 27 octobre 2026), sur la base de **7 heures par jour** (charge totale : 35 heures).

---

## 1. Tableau de Planification des Tâches

| ID | Tâche | Dépend de | Charge Estimée | Charge Réelle | Ressources (H, Mat, Tech, Fin) | Date Prévue | Date Réelle |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **T1** | **Audit, nettoyage des données & préparation des synopsis** | Aucune | 3h30 | 3h30 | **H:** Data Engineer<br>**Mat:** PC portable<br>**Tech:** Python, Pandas, NLTK<br>**Fin:** 0,00 € | 21/10/2026 | 21/10/2026 |
| **T2** | **Entraînement du modèle Keras & TF-IDF** | T1 | 3h30 | 4h00 | **H:** Data Engineer<br>**Mat:** PC portable<br>**Tech:** TensorFlow/Keras, Scikit-learn<br>**Fin:** 0,00 € | 21/10/2026 | 21/10/2026 |
| **T3** | **Recherche d'hyperparamètres & calcul des métriques ($F1$-score)** | T2 | 3h30 | 3h30 | **H:** MLOps<br>**Mat:** PC portable<br>**Tech:** KerasTuner, Joblib<br>**Fin:** 0,00 € | 22/10/2026 | 22/10/2026 |
| **T4** | **Sauvegarde des artefacts (`model.keras`, `vectoriseur.joblib`) & `metrics.json`** | T3 | 3h30 | 3h00 | **H:** MLOps<br>**Mat:** PC portable, Git<br>**Tech:** Joblib, JSON<br>**Fin:** 0,00 € | 22/10/2026 | 22/10/2026 |
| **T5** | **Développement de l'API FastAPI (`/predict`, `/health`)** | T4 | 4h00 | 4h30 | **H:** Backend Dev<br>**Mat:** PC portable<br>**Tech:** FastAPI, Uvicorn, Pytest<br>**Fin:** 0,00 € | 23/10/2026 | 23/10/2026 |
| **T6** | **Conteneurisation Docker (Dockerfile & tests locaux)** | T5 | 3h00 | 3h00 | **H:** DevOps<br>**Mat:** PC portable<br>**Tech:** Docker Desktop, Debian slim<br>**Fin:** 0,00 € | 23/10/2026 | 23/10/2026 |
| **T7** | **Mise en place de la CI/CD GitHub Actions & Barrière de Qualité** | T6 | 4h00 | 4h30 | **H:** DevOps<br>**Mat:** GitHub Cloud<br>**Tech:** GitHub Actions, Pytest, YAML<br>**Fin:** 0,00 € | 26/10/2026 | 26/10/2026 |
| **T8** | **Test de la barrière de qualité (Chaîne rouge - Seuil impossible)** | T7 | 3h00 | 2h30 | **H:** DevOps<br>**Mat:** GitHub Cloud<br>**Tech:** Git commit, GitHub Actions<br>**Fin:** 0,00 € | 26/10/2026 | 26/10/2026 |
| **T9** | **Déploiement sur Azure Container Apps (ACA) & Validation des Preuves (1 à 7)** | T8 | 4h00 | 4h30 | **H:** Cloud Engineer<br>**Mat:** Cloud Azure<br>**Tech:** Azure ACR, ACA, Azure CLI<br>**Fin:** ~0,06 € | 27/10/2026 | 27/10/2026 |
| **T10**| **Exécution du Plan de Fin de vie (Preuve 8), Rédaction & Synthèse C12** | T9 | 3h00 | 3h00 | **H:** Cloud Engineer / MLOps<br>**Mat:** Azure CLI, Word<br>**Tech:** Azure CLI, Markdown<br>**Fin:** 0,00 € | 27/10/2026 | 27/10/2026 |
| **TOTAL** | | | **35h00** | **36h00** | | | **~0,06 €** | |

---

## 2. Synthèse et Analyse des Écarts (Prévu vs Réel)

### Analyse de la Charge
* **Charge globale prévue :** 35 heures (5 jours ouvrés $\times$ 7h)
* **Charge réelle totale :** 36 heures (+1 heure d'écart, soit +2.8%)
* **Principaux écarts :**
  1. **T2 - Entraînement Keras (+30 min) :** Temps de convergence légèrement plus long que prévu sur le réseau de neurones avec la matrice TF-IDF.
  2. **T7 - CI/CD & Quality Gate (+30 min) :** Ajustements des chemins relatifs et du script `quality_gate.py` dans les runners Ubuntu de GitHub Actions.
  3. **T9 - Déploiement Cloud Azure (+30 min) :** Configuration initiale des ports d'écoute (8000) et liaison du registre ACR vers Container Apps.

### Bilan Financier
* **Budget prévisionnel :** $< 1{,}00\text{ \euro}$
* **Coût réel facturé :** **0,06 €** (Facturation minimale Azure limitée au temps de présence des ressources de test avant destruction).

### Retours d'expérience (Ce qu'il faudrait faire différemment)
* **Utiliser un DevContainer dès le premier jour :** Pour éviter tout écart d'environnement d'exécution entre la machine locale et le runner GitHub Actions.
* **Infrastructure as Code (IaC) :** Automatiser le provisionnement d'ACR et d'ACA avec Bicep ou Terraform au lieu de passer par des commandes Azure CLI manuelles.