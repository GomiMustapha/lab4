# Rapport d'analyse statique - OWASP UnCrackable Level 1

## Informations générales

* **Titre :** Analyse statique de OWASP UnCrackable Level 1
* **Date d'analyse :** 08/05/2026
* **Analyste :** Gomi Mustapha
* **APK analysé :** app-debug.apk
* **Version :** 1.0
* **VersionCode :** 1
* **Package principal :** owasp.mstg.uncrackable1
* **Provenance :** APK officiel OWASP MSTG Crackmes
* **Hash SHA256 :** 1DA8BF57D266109F9A07C01BF7111A1975CE01F190B9D914BCD3AE3DBEF96F21
* **Taille de l'APK :** 66651 octets
* **Outils utilisés :** JADX GUI, dex2jar v2.4, JD-GUI, apksigner

---

# Résumé exécutif

Cette analyse statique a révélé plusieurs mécanismes de protection intégrés dans l'application OWASP UnCrackable Level 1. Les principales préoccupations concernent la présence de mécanismes anti-root, anti-debugging et l'utilisation d'une configuration Android permettant la sauvegarde des données de l'application.

Le niveau de risque global est évalué comme **moyen** dans le contexte d'une application pédagogique. Aucun secret critique, token d'accès ou mot de passe codé en dur n'a été identifié lors de l'analyse.

## Actions prioritaires recommandées

1. Désactiver la sauvegarde Android via `android:allowBackup="false"`
2. Renforcer les mécanismes anti-tampering et anti-debugging
3. Éviter les messages explicites révélant les mécanismes de protection internes

---

# Constats détaillés

## Constat #1 : Détection de root intégrée

**Sévérité :** Moyenne

**Description :**
L'application intègre plusieurs mécanismes de détection de root via les méthodes `c.a()`, `c.b()` et `c.c()`. Si un appareil rooté est détecté, l'application affiche un message d'erreur puis termine son exécution.

**Localisation :**
`sg.vantagepoint.uncrackable1.MainActivity -> onCreate()`

**Code observé :**

```java
if (c.a() || c.b() || c.c()) {
    a("Root detected!");
}
```

**Impact potentiel :**
Ces mécanismes compliquent l'analyse dynamique de l'application et peuvent empêcher certains outils de sécurité ou de test de fonctionner correctement.

**Remédiation recommandée :**
Mettre en place des mécanismes de protection supplémentaires côté serveur et éviter de dépendre uniquement des contrôles locaux anti-root.

---

## Constat #2 : Détection du mode debug

**Sévérité :** Moyenne

**Description :**
L'application vérifie si elle est exécutée en mode débogage grâce à la méthode `b.a(getApplicationContext())`. En cas de détection, l'application affiche une alerte et s'arrête.

**Localisation :**
`sg.vantagepoint.uncrackable1.MainActivity -> onCreate()`

**Code observé :**

```java
if (b.a(getApplicationContext())) {
    a("App is debuggable!");
}
```

**Impact potentiel :**
La présence de contrôles anti-debugging indique une volonté de limiter l'analyse dynamique de l'application.

**Remédiation recommandée :**
Renforcer les protections via l'obfuscation, le contrôle d'intégrité et des vérifications natives supplémentaires.

---

## Constat #3 : Sauvegarde Android autorisée

**Sévérité :** Faible

**Description :**
Le manifeste Android contient l'attribut `android:allowBackup="true"`, ce qui autorise potentiellement la sauvegarde des données de l'application via ADB ou les mécanismes de backup Android.

**Localisation :**
`AndroidManifest.xml`

**Code observé :**

```xml
<application
    android:allowBackup="true">
```

**Impact potentiel :**
Des données locales potentiellement sensibles pourraient être récupérées via les mécanismes de sauvegarde Android.

**Remédiation recommandée :**
Désactiver les sauvegardes Android avec :

```xml
android:allowBackup="false"
```

---

## Constat #4 : Présence d'un mécanisme de vérification de secret

**Sévérité :** Faible

**Description :**
L'application contient une logique de validation d'un secret utilisateur via la méthode `a.a(string)`.

**Localisation :**
`sg.vantagepoint.uncrackable1.MainActivity -> verify()`

**Code observé :**

```java
if (a.a(string)) {
    alertDialogCreate.setTitle("Success!");
}
```

**Impact potentiel :**
Le mécanisme de vérification pourrait être étudié ou contourné lors d'une analyse plus avancée du bytecode.

**Remédiation recommandée :**
Déplacer les vérifications sensibles côté serveur lorsque cela est possible.

---

## Constat #5 : Absence d'API ou de secrets hardcodés

**Sévérité :** Faible

**Description :**
Les recherches globales effectuées dans JADX GUI n'ont révélé aucun token, mot de passe, API key ou endpoint HTTP/HTTPS codé en dur.

**Localisation :**
Recherche globale dans l'ensemble de l'APK

**Impact potentiel :**
Aucun impact direct identifié.

**Remédiation recommandée :**
Continuer à éviter le stockage de secrets directement dans le code de l'application.

---

# Analyse comparative JADX GUI vs JD-GUI

| Aspect             | JADX GUI                                      | JD-GUI                                |
| ------------------ | --------------------------------------------- | ------------------------------------- |
| Navigation         | Affiche manifeste, ressources et code Android | Affiche uniquement le code Java       |
| Ressources Android | Accès direct aux XML Android                  | Pas d'accès aux ressources            |
| Analyse Android    | Très adapté aux APK Android                   | Plus adapté aux JAR Java              |
| Lisibilité         | Code souvent mieux reconstruit                | Code parfois plus brut                |
| Obfuscation        | Reconstruction partielle possible             | Conserve davantage les noms obfusqués |
| Vue globale        | Structure complète de l'application           | Vue limitée au bytecode Java          |

## Conclusion comparative

JADX GUI est l'outil le plus adapté pour l'analyse complète d'applications Android car il permet d'accéder simultanément au manifeste Android, aux ressources XML et au code décompilé. JD-GUI reste néanmoins utile pour l'analyse complémentaire du bytecode Java après conversion DEX vers JAR.

---

# Annexes

## Informations du manifeste Android

* **Package principal :** `owasp.mstg.uncrackable1`
* **VersionName :** `1.0`
* **VersionCode :** `1`
* **minSdkVersion :** `19`
* **targetSdkVersion :** `28`

---

## Permissions demandées

Aucune permission Android dangereuse ou spécifique n'a été identifiée dans le manifeste.

---

## Composants exportés

### Activity principale

```xml
sg.vantagepoint.uncrackable1.MainActivity
```

Cette activité contient un `intent-filter` avec :

```xml
<action android:name="android.intent.action.MAIN"/>
<category android:name="android.intent.category.LAUNCHER"/>
```

Elle agit comme point d'entrée principal de l'application.

---

## Vérification de signature APK

Résultat de `apksigner verify --verbose` :

* Verified using v1 scheme : true
* Verified using v2 scheme : true
* Verified using v3 scheme : false
* Number of signers : 1

L'APK possède une signature valide compatible Android.

---

## Fichiers importants identifiés

### AndroidManifest.xml

Contient :

* configuration principale Android
* version de l'application
* composants Android
* paramètres de sécurité

### classes.dex

Contient le bytecode Dalvik principal de l'application.

### strings.xml

Chaînes importantes identifiées :

```text
Root detected!
App is debuggable!
Success!
Nope...
```

---

# Conclusion générale

Cette analyse statique de l'application OWASP UnCrackable Level 1 a permis d'identifier plusieurs mécanismes de protection couramment rencontrés dans les applications Android sensibles, notamment des contrôles anti-root et anti-debugging.

L'application ne contient pas de secrets critiques codés en dur ni d'API exposées, ce qui limite les risques immédiats. Toutefois, certaines configurations comme `allowBackup="true"` peuvent représenter des points d'amélioration.

L'utilisation combinée de JADX GUI, dex2jar et JD-GUI a permis d'obtenir une vision complète de la structure interne de l'application et de comparer différentes approches de décompilation Android.
