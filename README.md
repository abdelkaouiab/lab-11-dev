# 📍 Lab 11 — Application de Géolocalisation (Android + PHP + MySQL)

---

## 🎯 Objectifs pédagogiques

* Récupérer la latitude et la longitude d’un smartphone
* Comprendre les permissions Android liées à la localisation
* Envoyer des données vers un serveur PHP
* Enregistrer les coordonnées dans MySQL
* Structurer un projet mobile connecté à un backend

---

## ✅ Résultat attendu

L’application doit :

* Détecter la position géographique
* Afficher les coordonnées
* Envoyer latitude, longitude, date et IMEI
* Enregistrer les données dans MySQL

---

## 🏗️ Architecture du projet

### Partie serveur

* Base de données MySQL
* Classes PHP
* Script de réception HTTP

### Partie mobile

* Permissions Android
* GPS (LocationManager)
* Envoi HTTP (Volley)

---

# 🗄️ Partie Serveur

## 📌 Création de la base de données

```sql
CREATE DATABASE IF NOT EXISTS localisation
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

USE localisation;

CREATE TABLE position (
    id INT AUTO_INCREMENT PRIMARY KEY,
    latitude DOUBLE NOT NULL,
    longitude DOUBLE NOT NULL,
    date_position DATETIME NOT NULL,
    imei VARCHAR(50) NOT NULL
);
```

---

## 📌 Classe Position.php

```php
<?php
class Position {
    private $id;
    private $latitude;
    private $longitude;
    private $datePosition;
    private $imei;

    public function __construct($id, $latitude, $longitude, $datePosition, $imei) {
        $this->id = $id;
        $this->latitude = $latitude;
        $this->longitude = $longitude;
        $this->datePosition = $datePosition;
        $this->imei = $imei;
    }

    public function getLatitude() { return $this->latitude; }
    public function getLongitude() { return $this->longitude; }
    public function getDatePosition() { return $this->datePosition; }
    public function getImei() { return $this->imei; }
}
?>
```

---

## 📌 Connexion.php

```php
<?php
class Connexion {
    private $connexion;

    public function __construct() {
        $this->connexion = new PDO("mysql:host=localhost;dbname=localisation;charset=utf8mb4", "root", "");
    }

    public function getConnexion() {
        return $this->connexion;
    }
}
?>
```

---

## 📌 PositionService.php

```php
<?php
include_once 'connexion/Connexion.php';
include_once 'classe/Position.php';

class PositionService {
    private $connexion;

    public function __construct() {
        $this->connexion = new Connexion();
    }

    public function create($position) {
        $sql = "INSERT INTO position(latitude, longitude, date_position, imei)
                VALUES(:latitude, :longitude, :date_position, :imei)";

        $stmt = $this->connexion->getConnexion()->prepare($sql);
        $stmt->execute([
            ':latitude' => $position->getLatitude(),
            ':longitude' => $position->getLongitude(),
            ':date_position' => $position->getDatePosition(),
            ':imei' => $position->getImei()
        ]);
    }
}
?>
```

---

## 📌 createPosition.php

```php
<?php
include_once 'service/PositionService.php';

$latitude = $_POST['latitude'];
$longitude = $_POST['longitude'];
$datePosition = $_POST['date_position'];
$imei = $_POST['imei'];

$service = new PositionService();
$position = new Position(null, $latitude, $longitude, $datePosition, $imei);
$service->create($position);

echo "Position enregistrée";
?>
```

---

# 📱 Partie Mobile Android

## 📌 Permissions (AndroidManifest.xml)

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

---

## 📌 Dépendance Volley

```gradle
implementation 'com.android.volley:volley:1.2.1'
```

---

## 📌 Interface activity_main.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvInfo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Aucune position détectée"
        android:textSize="18sp" />
</LinearLayout>
```

---

## 📌 MainActivity.java

```java
LocationManager locationManager =
        (LocationManager) getSystemService(Context.LOCATION_SERVICE);

locationManager.requestLocationUpdates(
        LocationManager.GPS_PROVIDER,
        60000,
        150,
        new LocationListener() {
            @Override
            public void onLocationChanged(Location location) {
                double lat = location.getLatitude();
                double lon = location.getLongitude();
                tvInfo.setText("Lat: " + lat + " Lon: " + lon);
                addPosition(lat, lon);
            }
        }
);
```

---

## 📌 Envoi des données avec Volley

```java
private void addPosition(final double lat, final double lon) {
    StringRequest request = new StringRequest(Request.Method.POST, insertUrl,
        response -> Toast.makeText(this, response, Toast.LENGTH_SHORT).show(),
        error -> Toast.makeText(this, "Erreur", Toast.LENGTH_SHORT).show()) {

        @Override
        protected Map<String, String> getParams() {
            HashMap<String, String> params = new HashMap<>();
            params.put("latitude", String.valueOf(lat));
            params.put("longitude", String.valueOf(lon));
            params.put("date_position", "2026-01-01 00:00:00");
            params.put("imei", "test_device");
            return params;
        }
    };

    Volley.newRequestQueue(this).add(request);
}
```

---

# 🧪 Tests

* Lancer Apache + MySQL
* Vérifier URL PHP
* Activer GPS
* Exécuter l’application
* Vérifier insertion dans phpMyAdmin

---

# 🧠 Synthèse

Ce TP permet de connecter une application Android à un backend PHP/MySQL.

Compétences acquises :

* GPS Android
* Permissions
* Requêtes HTTP (Volley)
* PHP + PDO
* MySQL

➡️ Résultat : un système complet de géolocalisation connecté.
