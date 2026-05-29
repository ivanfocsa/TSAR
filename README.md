# TSAR - Toolbox for Security & Automated Reporting

TSAR (Toolbox for Security & Automated Reporting) est une plateforme de test d'intrusion web, modulaire et automatisée. Elle est conçue pour centraliser, exécuter et rapporter les résultats d'une large gamme d'outils de sécurité, le tout via une interface web moderne et réactive.

 <!-- Pensez à remplacer par une vraie capture d'écran -->

## ✨ Fonctionnalités

-   **Architecture Modulaire :** Ajoutez facilement de nouveaux outils en créant un simple fichier Python dans le dossier `modules/`.
-   **Exécution Asynchrone :** Les scans longs sont gérés en arrière-plan par Celery et Redis, sans jamais bloquer l'interface.
-   **Streaming en Temps Réel :** Suivez la sortie de vos scans en direct depuis votre navigateur grâce aux Server-Sent Events.
-   **Conteneur d'Outils Isolé :** Tous les outils de pentest sont exécutés dans un conteneur Docker Kali Linux (`toolbox`), gardant le système hôte propre et sécurisé.
-   **Gestion de Projets :** Organisez vos missions en projets. Les scans sont automatiquement liés aux projets en fonction de leur cible.
-   **Rapports Chiffrés :** Chaque scan génère un rapport PDF détaillé, chiffré au repos dans la base de données.
-   **Pivot VPN Facilité :** Générez des configurations WireGuard client/serveur et un "one-liner" d'installation pour pivoter facilement dans un réseau cible.
-   **Tâches Planifiées :** Automatisez la surveillance en planifiant des scans récurrents (quotidiens, hebdomadaires, mensuels).
-   **Analyse de Vulnérabilités :** Obtenez des informations détaillées sur une CVE ou analysez un rapport pour trouver les CVEs liées aux technologies détectées.
-   **Authentification Sécurisée :** Gestion des utilisateurs via Auth0.

## 🚀 Installation

### Prérequis

-   Docker
-   Docker Compose

### Étapes

1.  **Clonez le dépôt :**
    ```bash
    git clone https://github.com/ivanfocsa/TSAR.git
    cd TSAR
    ```

2.  **Configurez les variables d'environnement :**
    Copiez le fichier d'exemple. Sur macOS ou Linux, le fichier `.env` sera caché. C'est normal.
    ```bash
    cp .env.example .env
    ```
    Modifiez ensuite le fichier `.env` avec vos propres clés.
    ```bash
    nano .env
    ```
    Vous devrez remplir les clés pour **Auth0** et générer une clé de chiffrement pour les PDF.

3.  **Lancez l'application :**
    Utilisez Docker Compose pour construire les images et démarrer tous les services.
    ```bash
    docker-compose up -d --build
    ```

### 4. Accès à l'Application et Authentification (via Ngrok)

Pour utiliser pleinement TSAR, notamment la gestion des utilisateurs via Auth0, votre application web doit être accessible depuis Internet. C'est ici que Ngrok devient essentiel.

#### Option A : Accès Local (Fonctionnalités Limitées)

Si vous souhaitez simplement lancer des scans et consulter les rapports **sans authentification**, vous pouvez accéder à l'application directement.

-   **URL :** Ouvrez votre navigateur et allez sur `http://localhost:5373`.

**Point important :** Cette méthode ne permet **pas** l'authentification. Les services externes comme Auth0, qui fonctionnent sur Internet, ne peuvent pas rediriger les utilisateurs vers une adresse `http://localhost:5373` car cette adresse n'est connue que de votre machine locale. L'authentification échouera.

#### Option B : Accès Public avec Ngrok (Requis pour Auth0 et Toutes les Fonctionnalités)

Pour que l'authentification Auth0 fonctionne correctement, TSAR doit être accessible publiquement via une URL internet. Ngrok est l'outil idéal pour cela : il crée un **tunnel sécurisé** de votre machine locale vers Internet, exposant votre application web à une URL publique temporaire (souvent en `https`).

**4.1. Installez Ngrok**
Si Ngrok n'est pas déjà installé sur votre système, suivez les instructions officielles sur [ngrok.com](https://ngrok.com/download) ou utilisez un gestionnaire de paquets comme Homebrew sur macOS :
```bash
brew install ngrok/ngrok/ngrok
```
Pour des sessions Ngrok plus longues et stables, il est **fortement recommandé** de créer un compte gratuit sur [ngrok.com](https://ngrok.com) et de lier votre `authtoken` (disponible dans votre tableau de bord Ngrok après inscription). Cela se fait généralement avec `ngrok config add-authtoken <votre_authtoken>`.

**4.2. Lancez Ngrok et obtenez votre URL publique**
Ouvrez une **nouvelle fenêtre de terminal** (laissez `docker-compose up -d` tourner dans la première !) et exécutez la commande suivante pour créer un tunnel HTTP vers le port 5373 de votre application TSAR :
```bash
ngrok http 5373
```
Après quelques instants, Ngrok affichera une interface dans votre terminal avec plusieurs informations, dont votre **URL de redirection publique**. Elle ressemblera à `https://xxxxxxxx.ngrok-free.app`. Copiez cette URL, elle est essentielle pour la suite.

**4.3. Mettez à jour la configuration de TSAR et d'Auth0**
Cette étape est **cruciale** pour que l'authentification fonctionne.

*   **Modifiez votre fichier `.env` de TSAR** : Ouvrez le fichier `.env` que vous avez créé à l'étape 2. Vous devez mettre à jour les variables `APP_BASE_URL` et `AUTH0_CALLBACK_URL` avec l'URL `https` que Ngrok vient de vous fournir.
    ```dotenv
    # Exemple avec une URL Ngrok
    APP_BASE_URL=https://xxxxxxxx.ngrok-free.app
    AUTH0_CALLBACK_URL=https://xxxxxxxx.ngrok-free.app/auth/callback
    ```
    Ces URLs indiquent à TSAR quelle est son adresse publique et où Auth0 doit le rediriger après l'authentification.

*   **Mettez à jour votre tableau de bord Auth0** : Connectez-vous à votre compte Auth0 (où vous avez configuré votre application TSAR). Dans les paramètres de votre application (Applications -> Applications -> Votre application TSAR), trouvez la section "Allowed Callback URLs" et **ajoutez-y exactement la même URL de callback** que celle que vous avez mise dans votre fichier `.env` (par exemple, `https://xxxxxxxx.ngrok-free.app/auth/callback`). Sans cette étape, Auth0 ne permettra pas la redirection vers votre URL Ngrok pour des raisons de sécurité.

**4.4. Redémarrez TSAR pour appliquer les changements**
Pour que TSAR prenne en compte les modifications effectuées dans le fichier `.env`, vous devez redémarrer le conteneur `web`.
```bash
docker-compose up -d --build
```
Ceci reconstruit et redémarre les services nécessaires.

Félicitations ! Vous pouvez maintenant accéder à TSAR via votre URL Ngrok (celle en `https://xxxxxxxx.ngrok-free.app`) et l'authentification via Auth0 fonctionnera parfaitement, vous donnant accès à toutes les fonctionnalités de la plateforme.

## 🔧 Variables d'Environnement (`.env`)

-   `SECRET_KEY`: Clé secrète pour Flask (ex: `openssl rand -hex 32`).
-   `PDF_ENC_KEY`: Clé de chiffrement Fernet pour les rapports PDF (générez-en une avec le script Python ci-dessous).
-   `AUTH0_DOMAIN`, `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`: Vos identifiants Auth0.
-   `AUTH0_CALLBACK_URL`: URL de callback pour Auth0. Doit correspondre à votre URL d'accès (ex: `https://votre-url.ngrok-free.app/auth/callback`). **Ceci doit être l'URL fournie par Ngrok et également configurée dans votre tableau de bord Auth0.**
-   `APP_BASE_URL`: URL de base de votre application. Doit correspondre à votre URL d'accès (ex: `https://votre-url.ngrok-free.app`). **Ceci doit être l'URL fournie par Ngrok.**
-   `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`: Identifiants pour la base de données PostgreSQL (peuvent être laissés par défaut).

**Pour générer `PDF_ENC_KEY` :**
```python
from cryptography.fernet import Fernet
key = Fernet.generate_key()
print(key.decode())
```

## 🧩 Ajouter un Nouveau Module

L'un des plus grands atouts de TSAR est sa facilité d'extension.

1.  Créez un nouveau fichier Python dans le dossier `modules/` (ex: `modules/my_new_scanner.py`).
2.  Dans ce fichier, définissez un dictionnaire nommé `MODULE`.

**Structure du dictionnaire `MODULE` :**

```python
# modules/my_new_scanner.py
import shlex

MODULE = {
    # Nom affiché dans l'interface
    "name": "Mon Nouveau Scanner",
    # Description affichée dans l'interface
    "description": "Description de ce que fait ce scanner.",
    # Catégorie pour le regroupement dans la page des modules
    "category": "Scan Réseau",
    # (Optionnel) Masquer le module de la liste principale
    "hidden_from_list": False,
    # Schéma du formulaire pour les paramètres
    "schema": [
        {
            "group_name": "Paramètres du Scan",
            "fields": [
                {"name": "target", "type": "string", "placeholder": "exemple.com", "required": True},
                {"name": "scan_type", "type": "select", "choices": ["rapide", "complet"], "default": "rapide"},
            ]
        }
    ],
    # Fonction lambda qui construit la commande à exécuter dans le conteneur 'toolbox'
    "cmd": lambda p: [
        "nmap",
        "-sV" if p.get("scan_type") == "complet" else "-F",
        shlex.quote(p["target"])
    ],
}
```

3.  Redémarrez les conteneurs (`docker-compose restart worker web`) et votre nouveau module apparaîtra automatiquement dans l'interface !

## Documentation technique

- [Architecture](docs/ARCHITECTURE.md) : composants, flux d'execution et posture securite.
- [Contrat des modules](docs/MODULE_CONTRACT.md) : schema attendu pour ajouter un outil proprement.

## 🏗️ Structure du Projet

`docker-compose.yaml` : Le chef d'orchestre de TSAR, il définit et lance tous les services Docker (web, workers, db, redis, toolbox).
`modules/` : La boîte à outils magique de TSAR, contenant toutes les définitions des outils de pentest.
  - `aggressive_audit.py` : Module d'audit complet (Nmap, Nuclei, Metasploit).
  - `code_secrets_scanner.py` : Module de scan de secrets GitHub (Gitleaks).
  - `iot_pivot.py` : Module de gestion de pivot VPN WireGuard et d'audits réseau interne.
  - `metasploit_scan.py` : Module de scan de vulnérabilités Metasploit auxiliaire.
  - `osint_global.py` : Module de recherche OSINT de pseudo (GoSearch).
  - `reporting_global.py` : Module générateur de rapports de synthèse multi-outils.
  - `subdomain_takeover.py` : Module de détection de "subdomain takeover".
  - `vuln_global.py` : Module d'évaluation de vulnérabilités web complet.
`toolbox/` : Le laboratoire de pentest portable, un conteneur Docker avec tous les outils.
  - `Dockerfile` : La recette pour construire l'image Kali Linux des outils nécessaires.
`web/` : Le cerveau de TSAR, c'est l'application Flask, ses templates et sa logique métier.
  - `Dockerfile` : La recette pour construire l'image Docker de l'application Flask et des workers.
  - `requirements.txt` : La liste des bibliothèques Python nécessaires à l'application.
  - `app/` : Le cœur de l'application Flask.
    - `__init__.py` : Lance l'app Flask, connecte la BDD, l'authentification et prépare Celery.
    - `celery_app.py` : Le point d'entrée de Celery pour les workers.
    - `config.py` : Gère les réglages de l'application Flask.
    - `models.py` : Les plans de la base de données (Projets, Rapports, Utilisateurs, etc.).
    - `modules.py` : Le bibliothécaire des modules, il les charge et les rend accessibles.
    - `pdf.py` : La machine à imprimer, contient les fonctions de génération de rapports PDF.
    - `pdf_crypto.py` : Le coffre-fort des rapports, gère leur chiffrement/déchiffrement.
    - `routes.py` : La carte routière de l'application, définit toutes les pages et interactions.
    - `tasks.py` : Le centre de commande des opérations lourdes (scans, rapports, CVE) exécutées en arrière-plan.
  - `static/` : Contient les fichiers qui donnent le "look" à TSAR.
    - `css/` : Les styles CSS.
      - `tsar.css` : Le fichier de style principal de l'interface.
  - `templates/` : La galerie d'art, contient tous les modèles HTML des pages web.
    - `_form_field.html` : Un petit modèle réutilisable pour afficher un champ de formulaire.
    - `_macros.html` : Des "morceaux" de code HTML réutilisables (sélecteur de cibles, etc.).
    - `base.html` : Le squelette de toutes les pages (navigation, fond animé, scripts globaux).
    - `cve_analysis.html` : La page pour analyser les CVEs (par ID ou depuis un rapport).
    - `dashboard.html` : Votre page d'accueil avec les métriques et l'historique.
    - `guide.html` : Le mode d'emploi détaillé de TSAR.
    - `module_launcher.html` : La page de lancement d'un module spécifique.
    - `modules.html` : La vitrine de tous les modules disponibles.
    - `profile.html` : La page de votre compte utilisateur (nom, avatar, stats).
    - `project_detail.html` : La page détaillée d'un projet (vulnérabilités, rapports, documents, synthèse).
    - `projects.html` : La liste et la gestion de tous vos projets.
    - `scheduled.html` : La page pour planifier des tâches automatiques.
    - `stdout_report.html` : Le modèle pour générer le contenu des rapports PDF.
    - `veille.html` : La page d'information cyber (flux RSS).
