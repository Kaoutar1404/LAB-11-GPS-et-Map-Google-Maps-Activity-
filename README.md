 # 📍 Système de Localisation Mobile (Android + PHP + MySQL)
## 📌 Description
Ce projet est une application de localisation en temps réel basée sur une architecture client/serveur :
- 📱 Une application Android récupère la position GPS du smartphone
- 🌐 Un serveur PHP reçoit les données via HTTP (Volley)
- 🗄️ Une base de données MySQL stocke les positions
---
## 🏗️ Architecture du système
### 🔹 Partie serveur (PHP + MySQL)
- Base de données MySQL : localisation
- Script PHP : réception et traitement des données
- Classes PHP : modèle objet + accès aux données
### 🔹 Partie mobile (Android)
- Permissions Android (GPS, Internet, Téléphone)
- Récupération GPS via LocationManager
- Envoi H…
[00:52, 27/05/2026] ✨: b
[00:54, 27/05/2026] ✨: Partie 1 : Création du projet “Google Maps Activity”
Étape 1 — Créer un nouveau projet Android
Android Studio → New Project
Choisir un projet vide (ou directement Maps Activity selon version)
Uploaded Image
Étape 2 — Choisir “Google Maps Activity”
Dans les templates : Google Maps Activity
Cliquer Next puis Finish
Uploaded Image
Étape 3 — Comprendre la structure générée
Après génération, on obtient généralement :
MapsActivity.java
activity_maps.xml (contient le fragment map)
google_maps_api.xml (clé API)
dépendances Gradle (maps)
✅ Checkpoint
Le projet compile (même sans clé, mais la map ne s’affiche pas correctement)
Uploaded ImagePartie 2 : Clé Google Maps API (google_maps_key)
Étape 4 — Récupérer le lien de génération
Ouvrir res/values/google_maps_api.xml
…
[00:54, 27/05/2026] ✨: Voici la version README.md ajoutée (Partie Google Maps) que tu peux intégrer directement à ton projet 👇

⸻

# 🗺️ Partie Google Maps Android (GPS + Map SDK)
## 📌 Objectif
Cette partie ajoute une carte Google Maps à l’application Android afin de :
- 📍 Afficher la position en temps réel
- 🧭 Suivre les déplacements du smartphone
- 📌 Placer des markers sur la carte
- 🔄 Centrer la caméra automatiquement sur la position
---
# 🧱 Partie 1 : Création du projet Google Maps
## Étape 1 — Créer le projet
Dans Android Studio :
- New Project
- Choisir *Google Maps Activity*
---
## Étape 2 — Structure générée
Après création, le projet contient :
- MapsActivity.java
- activity_maps.xml
- google_maps_api.xml
- dépendances Google Maps SDK
---
## ✅ Checkpoint
Le projet compile même sans clé API, mais la carte ne s’affiche pas correctement.
---
# 🔑 Partie 2 : Google Maps API Key
## Étape 3 — Générer la clé
Dans google_maps_api.xml :
1. Ouvrir le lien fourni dans le fichier
2. Aller sur Google Cloud Console
3. Activer :
   - Maps SDK for Android
4. Créer une API Key
---
## Étape 4 — Ajouter la clé
Dans res/values/google_maps_api.xml :
```xml
<string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">
    VOTRE_CLE_ICI
</string>

⸻

✅ Checkpoint

Si la clé est correcte :

* la carte s’affiche
* la grille Google Maps apparaît

⸻

🔐 Partie 3 : Permissions Android

Étape 5 — AndroidManifest.xml

<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />

⸻

⚠️ Runtime Permission (important)

Même si déclarée dans le Manifest, Android 6+ exige une permission dynamique.

Sinon :

* ❌ crash SecurityException
* ❌ pas de GPS actif

⸻

Étape 6 — GPS Alert Dialog

private void buildAlertMessageNoGps() {
    final AlertDialog.Builder builder = new AlertDialog.Builder(this);
    builder.setMessage("Your GPS seems to be disabled, do you want to enable it?")
            .setCancelable(false)
            .setPositiveButton("Yes", (dialog, id) ->
                    startActivity(new Intent(android.provider.Settings.ACTION_LOCATION_SOURCE_SETTINGS)))
            .setNegativeButton("No", (dialog, id) -> dialog.cancel());
    builder.create().show();
}

⸻

📌 Explication

* ✔️ Yes → ouvre paramètres GPS
* ❌ No → ferme la boîte
* 🚫 setCancelable(false) → empêche fermeture accidentelle

⸻

🗺️ Partie 4 : onMapReady()

Étape 7 — Implémentation complète

@Override
public void onMapReady(GoogleMap googleMap) {
    mMap = googleMap;
    LocationManager locationManager =
            (LocationManager) getSystemService(Context.LOCATION_SERVICE);
    LatLng defaultPos = new LatLng(-34, 151);
    mMap.addMarker(new MarkerOptions().position(defaultPos).title("Default"));
    mMap.moveCamera(CameraUpdateFactory.newLatLng(defaultPos));
    if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
            == PackageManager.PERMISSION_GRANTED) {
        locationManager.requestLocationUpdates(
                LocationManager.NETWORK_PROVIDER,
                1000,
                50,
                new LocationListener() {
                    @Override
                    public void onLocationChanged(Location location) {
                        LatLng pos = new LatLng(
                                location.getLatitude(),
                                location.getLongitude()
                        );
                        mMap.addMarker(new MarkerOptions().position(pos).title("Position"));
                        mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(pos, 15f));
                    }
                    @Override
                    public void onProviderDisabled(String provider) {
                        buildAlertMessageNoGps();
                    }
                }
        );
    } else {
        ActivityCompat.requestPermissions(
                this,
                new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                200
        );
    }
}

⸻

📌 Explication

📍 LocationManager

Permet d’écouter GPS / réseau

📡 NETWORK_PROVIDER

* rapide
* fonctionne en intérieur
* moins précis

📡 GPS_PROVIDER

* très précis
* lent
* nécessite extérieur

⸻

⚡ Partie 5 : Permissions result

@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    if (requestCode == 200 && grantResults.length > 0
            && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        Toast.makeText(this, "Permission accordée", Toast.LENGTH_SHORT).show();
        onMapReady(mMap);
    }
}

⸻

📌 Partie 6 : Zoom & caméra

float zoomLevel = 15f;
mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(position, zoomLevel));

⸻

📊 Niveaux de zoom

Zoom	Niveau
10–12	Ville
15	Quartier
18–20	Rue / détails

⸻

🧠 Version optimisée (1 seul marker)

private Marker currentMarker;
@Override
public void onLocationChanged(Location location) {
    LatLng pos = new LatLng(location.getLatitude(), location.getLongitude());
    if (currentMarker == null) {
        currentMarker = mMap.addMarker(
                new MarkerOptions().position(pos).title("Position actuelle")
        );
    } else {
        currentMarker.setPosition(pos);
    }
    mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(pos, 15f));
}

⸻

🚀 Résultat final

✔️ Carte Google Maps affichée
✔️ Position GPS en temps réel
✔️ Marker dynamique
✔️ Zoom automatique
✔️ Gestion permissions Android

⸻

👨‍💻 Technologies utilisées

* Android Studio
* Google Maps SDK
* GPS (LocationManager)
* Java
