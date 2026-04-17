# Documentation du programme Arduino DIY_PM

## Description générale
Ce programme Arduino permet de mesurer la qualité de l’air (particules fines PM2.5/PM10, température, humidité) et d’envoyer les données via LoRaWAN. Il gère les capteurs, la communication, la basse consommation et le cycle de mesure.

## Fonctionnalités principales
- Lecture des capteurs SDS011 (PM2.5/PM10) et HTU21DF (température/humidité)
- Moyennage des mesures sur une période définie
- Envoi des données via LoRaWAN (OTAA ou ABP)
- Gestion de l’alimentation (pulsation, sommeil, reset)
- Configuration initiale via port série

## Architecture générale

```mermaid
graph TD
    subgraph Capteurs
        SDS011["SDS011<br/>PM2.5/PM10"]
        HTU21DF["HTU21DF<br/>Temp/Hum"]
    end
    subgraph Microcontrôleur
        A["Arduino MKR WAN 1300/1310"]
        LORA["Module LoRaWAN"]
        RTC["RTCZero"]
        LOWPWR["ArduinoLowPower"]
    end
    subgraph Réseau
        GW["Passerelle LoRaWAN"]
        Serveur["Serveur distant"]
    end
    SDS011 -- UART --> A
    HTU21DF -- I2C --> A
    A -- SPI/UART --> LORA
    A -- Gestion sommeil --> LOWPWR
    A -- Horloge --> RTC
    LORA -- Radio --> GW
    GW -- Internet --> Serveur
```

## Diagramme de séquence principal

```mermaid
sequenceDiagram
    participant Utilisateur
    participant Arduino
    participant Capteurs
    participant LoRaWAN
    Utilisateur->>Arduino: Démarrage / Reset
    Arduino->>Capteurs: Initialisation
    loop Mesure périodique
        Arduino->>Capteurs: Lecture PM/Temp/Hum
        Capteurs-->>Arduino: Valeurs mesurées
        Arduino->>Arduino: Moyennage des mesures
    end
    Arduino->>LoRaWAN: Envoi des données
    LoRaWAN-->>Serveur: Transmission
    Arduino->>Arduino: Mode sommeil / Pulsation
    Arduino->>Utilisateur: Logs série
```

## Fichiers principaux
- `app.ino` : Code principal Arduino
- `arduino_secrets.h` : Identifiants LoRaWAN

## Variables et constantes clés
- `PULSEFREQUENCY`, `PULSEDURATION`, `MEASUREFREQUENCY`, `MEASUREDURATION` : Paramètres de mesure
- `ACTIVEPM`, `ACTIVETH`, `RESET`, `PULSE`, `FIRSTCONFIG` : Broches de contrôle

## Fonctions principales
- `setup()` : Initialisation
- `loop()` : Boucle principale
- `sendLoraMessage()` : Envoi LoRa
- `dopulse()` : Gestion du cycle de sommeil
- `firstConfiguration()` : Configuration LoRa
- `getDeviceInfos()` : Informations module
- `ledBlink()` : Indication activité

## Dépendances
- SdsDustSensor
- Adafruit_HTU21DF
- MKRWAN
- ArduinoLowPower
- RTCZero

## Remarques
- Le code est conçu pour fonctionner sur Arduino MKR WAN 1300/1310.
- La configuration initiale LoRaWAN peut être lancée via le bouton FIRSTCONFIG.
- Les identifiants LoRaWAN sont à renseigner dans `arduino_secrets.h`.
