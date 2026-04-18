# 🌿 Serre Connectée — Smart Greenhouse IoT System

> Projet de Fin d'Année — Licence en Ingénierie des Systèmes Informatiques — 2025/2026

**Réalisé par :** Asma BELLALAH · Wala OUAILI · Zeineb HENCHIRI · Ale ALOUI · Tasnim ABID  
**Encadrant :** Pr. Faouzi MOUSSA

---

## 📖 Description

Une serre agricole intelligente basée sur l'ESP32, permettant la 
**surveillance en temps réel**
**automatisation de l'arrosage et de la ventilation**,
 **contrôle d'accès RFID** 
 **pilotage à distance** via une application mobile Flutter connectée à Firebase.

---

## ✨ Fonctionnalités

| Catégorie | Détail |
|-------------------------|----------------------------------------------------------------------------------|
| 🌡️ Capteurs            | Température & humidité air (DHT11), humidité sol (capacitif), accès RFID (PN532) |
| ⚙️ Automatisation      | Ventilateur si temp > 30 °C — Pompe si sol sec          |
| 📟 Affichage local     | LCD I²C 16×2 (temp / hum / sol)                         |
| 🔐 Contrôle d'accès    | Badge RFID → servo-moteur → porte + journal Firebase    |
| 📱 Application mobile  | Dashboard live · Commandes manuelles · Alertes push FCM |
| ☁️ Cloud               | Firebase Realtime DB · Firestore · Auth · FCM · Storage |

---

## 🏗️ Architecture du système

```
[DHT11] [Sol capacitif] [PN532 RFID]
         ↓         ↓          ↓
      ┌──────────────────────────┐
      │   ESP32 WROOM 32D        │  ← Cerveau · WiFi · Logique auto
      └──────────────────────────┘
         ↓         ↓          ↓
   [Pompe] [Ventilateur] [Servo/LED]   (via relais 4 canaux)
         ↓
   Firebase (Google Cloud)
   Realtime DB · Firestore · Auth · FCM
         ↓
   Application Flutter (mobile)
   Dashboard · Commandes · Alertes push
```

---

## 🧰 Matériel requis

| Composant | Référence |
|---|---|
| Microcontrôleur | ESP32 WROOM 32D |
| Capteur temp/hum | DHT11 |
| Capteur humidité sol | Capacitif (sortie analogique) |
| Module RFID | PN532 NFC/RFID V3 |
| Affichage | LCD I²C 16×2 |
| Actionneur porte | Servo-moteur SG90 |
| Relais | Module 4 canaux 5V |
| Pompe | Pompe à eau 5V |
| Ventilation | Ventilateur mini 5V |
| Voyant | LED |

---

## 📌 Câblage (Tableau des pins)

| Composant | Pin module | Pin ESP32 | Remarque |
|---|---|---|---|
| DHT11 | DATA | GPIO 4 | |
| DHT11 | VCC | 3.3V | |
| LCD I²C | SDA | GPIO 21 | Bus I²C |
| LCD I²C | SCL | GPIO 22 | Bus I²C |
| LCD I²C | VCC | 5V | |
| Capteur sol | AO | GPIO 34 | Entrée analogique |
| Capteur sol | VCC | 3.3V | |
| Relais (Pompe) | IN1 | GPIO 23 | |
| Relais (Ventilateur) | IN2 | GPIO 19 | |
| PN532 RFID | SDA | GPIO 21 | Partagé avec LCD |
| PN532 RFID | SCL | GPIO 22 | Partagé avec LCD |
| PN532 RFID | VCC | 5V (ou 3.3V) | |
| Servo-moteur | SIGNAL | GPIO 18 | |
| Servo-moteur | VCC | **5V externe** | ⚠️ Important |

---

## 🚀 Installation & déploiement

### 1. Firmware ESP32

**Prérequis :** Arduino IDE avec le support ESP32

**Bibliothèques à installer (Gestionnaire de bibliothèques) :**
```
DHT sensor library          — Adafruit
LiquidCrystal I2C           — Frank de Brabander
Adafruit PN532              — Adafruit
ESP32Servo                  — Kevin Harrington
```

**Étapes :**
1. Ouvrir `pfa-final.ino` dans Arduino IDE
2. Sélectionner la carte : `ESP32 Dev Module`
3. Modifier le UID autorisé dans le code :
   ```cpp
   uint8_t allowedUID[] = {0x04, 0xA1, 0xB2, 0xC3}; // ← Ton badge RFID
   ```
4. (Optionnel) Ajuster les seuils :
   ```cpp
   float TEMP_LIMIT = 30.0;   // °C → déclenche le ventilateur
   int   SOIL_LIMIT = 2000;   // valeur brute → déclenche la pompe
   ```
5. Téléverser sur l'ESP32

### 2. Firebase

1. Créer un projet sur [Firebase Console](https://console.firebase.google.com)
2. Activer : **Realtime Database** · **Firestore** · **Authentication** · **FCM**
3. Télécharger `google-services.json` et le placer dans `/android/app/`
4. Ajouter vos clés Firebase dans le firmware ESP32 (WiFi SSID / password / URL Firebase)

### 3. Application Flutter

```bash
git clone https://github.com/<votre-username>/serre-connectee.git
cd serre-connectee/flutter_app
flutter pub get
flutter run
```

---

## 📂 Structure du dépôt

```
serre-connectee/
├── firmware/
│   └── pfa-final.ino          # Code Arduino ESP32
├── flutter_app/               # Application mobile Flutter
│   ├── lib/
│   ├── android/
│   └── pubspec.yaml
├── docs/
│   ├── Block_diagram.png      # Diagramme de blocs
│   ├── cas_utilisation.png    # Diagramme de cas d'utilisation
│   └── rapport.docx           # Rapport complet du projet
└── README.md
```

---

## ⚙️ Logique d'automatisation

```
loop() toutes les 500 ms :
  ├─ Lire DHT11 → temp, hum
  ├─ Lire sol analogique → valeur sol
  ├─ temp > 30°C  → Activer ventilateur (RELAY_FAN  LOW)
  ├─ sol  > 2000  → Activer pompe       (RELAY_PUMP LOW)
  ├─ Scan RFID    → badge valide ? ouvrir porte + log Firebase
  └─ Afficher sur LCD : T / H / Sol
```

---

## 📱 Application mobile (Flutter)

- **Architecture :** MVVM
- **Écrans :** Login · Dashboard live · Contrôle manuel · Journal RFID · Alertes
- **Temps réel :** Firebase Realtime Database stream
- **Notifications :** Firebase Cloud Messaging (FCM)
- **Modèle de données :** `SensorData` (temperature, humidity, soilMoisture, timestamp)

---

## 🔮 Améliorations futures

- [ ] Intégration WiFi complète dans le firmware (envoi Firebase via HTTPs)
- [ ] Support multi-badges RFID avec gestion des rôles
- [ ] Prédiction IA des besoins en eau (ML sur Firebase)
- [ ] Panneau solaire pour alimentation autonome
- [ ] Caméra embarquée pour surveillance visuelle

---

## 📄 Licence

Ce projet est réalisé dans un cadre académique. Libre d'utilisation à des fins éducatives.

---

> *"L'agriculture intelligente commence par des données intelligentes."*
