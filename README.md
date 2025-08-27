# WAZUH
Automatisation de la Détection des Alertes MITRE ATT&amp;CK via Wazuh SIEM
# 🔐 Automatisation de la Détection des Alertes MITRE ATT&CK via Wazuh SIEM

Ce projet permet de récupérer automatiquement les alertes liées au framework MITRE ATT&CK générées par Wazuh, via l’API RESTful, de les convertir en format CSV, et de les analyser afin de les comparer à une base de données d’attaque (VERIS).
Il inclut également une méthode manuelle pour récupérer ces alertes via l’interface web de Wazuh.

---

## 🧩 Prérequis

- Accès à un serveur Wazuh fonctionnel (Manager + Dashboard)
- Droits administrateur sur le Wazuh Manager
- Python 3 installé sur le serveur Wazuh
- Accès au terminal du Wazuh Manager

---

## 🧑‍💻 1. Création d’un utilisateur API sur Wazuh

Connectez-vous à l'interface web de Wazuh avec un compte administrateur.

1. Accédez à **"Security" → "API configuration" → "Users"**.
2. Cliquez sur **"Add user"**.
3. Remplissez les champs :
   - **Username** : `test`
   - **Password** : `jW6fyEU?+KNiuGe2GDxyUtzsB.pWLAQK`
   - **Role** : `read` ou `administrator`
4. Cliquez sur **"Save"**.

---

## 🔑 2. Authentification via l’API Wazuh & Récupération du Token

Dans le terminal du Wazuh Manager :

image1 

 

 
Réponse attendue :

 
{
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}

Copiez le token pour les appels suivants à l’API.

 ## 🐍 3. Script Python pour Récolter les Alertes MITRE ATT&CK
Ce script se connecte à l’API Wazuh, récupère les alertes contenant des références MITRE ATT&CK, et les exporte dans un fichier .csv. Il est prêt à l’emploi.
📌 À exécuter sur le serveur Wazuh (ou toute machine ayant accès à l’API de Wazuh).

Créez un fichier get_mitre_alerts.py :

import requests
import csv
import datetime
import urllib3

# Désactiver les avertissements SSL (si certificat non valide)
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# === CONFIGURATION ===
WAZUH_API_URL = "https://localhost:55000"
USERNAME = "test"
PASSWORD = " jW6fyEU?+KNiuGe2GDxyUtzsB.pWLAQK "

# === AUTHENTIFICATION ===
auth_response = requests.post(
    f"{WAZUH_API_URL}/security/user/authenticate",
    headers={"Content-Type": "application/json"},
    json={"username": USERNAME, "password": PASSWORD},
    verify=False
)

if auth_response.status_code != 200:
    print("[✘] Échec de l’authentification")
    exit()

token = auth_response.json().get("data", {}).get("token")
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {token}"
}

# === RECHERCHE D’ALERTES MITRE ===
params = {
    "q": "mitre",        # Filtre simple (recherche texte)
    "limit": 1000        # Modifier si besoin
}

alert_response = requests.get(
    f"{WAZUH_API_URL}/alerts",
    headers=headers,
    params=params,
    verify=False
)

if alert_response.status_code != 200:
    print(f"[✘] Erreur lors de la récupération des alertes : {alert_response.status_code}")
    exit()

alerts = alert_response.json().get("data", {}).get("alerts", [])

# === EXPORT EN CSV ===
filename = f"mitre_alerts_{datetime.datetime.now().strftime('%Y%m%d_%H%M%S')}.csv"

with open(filename, mode='w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    writer.writerow(["timestamp", "rule.id", "rule.level", "rule.description", "agent.name"])
    
    for alert in alerts:
        rule = alert.get("rule", {})
        agent = alert.get("agent", {})
        writer.writerow([
            alert.get("timestamp", ""),
            rule.get("id", ""),
            rule.get("level", ""),
            rule.get("description", ""),
            agent.get("name", "")
        ])

print(f"[✔] {len(alerts)} alertes MITRE exportées dans : {filename}")


## 🌐 4. Récupération des alertes 
             ## 4.1 Récupération Manuelle via l'Interface Web Wazuh
1.	Connectez-vous à l'interface web de Wazuh.
2.	Naviguez vers "Security events" → "Events".
3.	Utilisez la barre de recherche et tapez :

rule.groups:mitre

4.	Filtrez les résultats selon la période, le niveau de criticité ou l’agent.
5.	Cliquez sur l’icône d’export CSV (en haut à droite de la liste des événements) pour télécharger les alertes filtrées.
## 4.2 Récupération depuis le terminal
💡 Exécution :

python3 get_mitre_alerts.py



## 5📁 Résultat
Les alertes MITRE ATT&CK sont extraites et enregistrées dans un fichier .csv contenant :
•	Date & Heure de l’alerte
•	ID de la règle
•	Niveau de gravité
•	Description de la règle
•	Nom de l’agent

Exemple de Fichier CSV Généré
Voici un aperçu du fichier .csv généré par le script, nommé par exemple mitre_alerts_20250823_153400.csv :

 

Tu peux ensuite ouvrir ce fichier dans Excel, LibreOffice ou l’importer dans un outil d’analyse.


                     ## Annexes
 
 Fig : Dashboard de wazuh avec les résultats des attaques 

 
Fig : Attaques éffectuées depuis Kali Linux
 
Fig : Installation et activation illustrant le fonctionnement de wazuh agent sur la machine cliente
 
Fig : Vue analytique des alertes détectés depuis l’interface web 
 
Fig : Installation et activation illustrant le fonctionnement du service ssh sur la machine client


