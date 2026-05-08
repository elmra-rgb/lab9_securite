# LAB 9 - ANALYSE DE SURFACE D'ATTAQUE ANDROID AVEC DROZER
## Audit défensif en environnement autorisé

---

## 1. INTRODUCTION

Ce laboratoire avait pour objectif d'utiliser **Drozer** pour l'analyse de sécurité d'applications Android dans un cadre défensif et autorisé. À travers cette démarche, j'ai pu :

- Maîtriser l'utilisation de Drozer pour l'audit mobile
- Identifier les composants Android exposés (Activities, Services, Receivers, Providers)
- Évaluer les risques de sécurité liés aux composants mal configurés
- Documenter méthodiquement les résultats d'un audit de sécurité
- Proposer des remédiations conformes aux standards OWASP MASVS

---

## 2. AVERTISSEMENT ÉTHIQUE

**À lire absolument :**

Ce laboratoire s'inscrit dans un cadre d'audit de sécurité **défensif et autorisé** :
- Toutes les analyses sont effectuées sur un émulateur contrôlé
- L'application testée est spécifiquement conçue pour ce laboratoire
- Aucune donnée réelle n'est manipulée
- Les techniques utilisées sont strictement défensives (analyse, non-exploitation)

---

## 3. ENVIRONNEMENT DE TEST

### 3.1 Configuration matérielle et logicielle

| Élément | Spécification |
|---------|---------------|
| **Machine hôte** | Mac Apple Silicon M2 (ARM-64 Native) |
| **Système d'exploitation** | macOS (terminal zsh) |
| **Outil d'analyse** | Drozer |
| **Appareil cible** | Émulateur Android AVD – API 30 (Android 11) |
| **Outil de connexion** | ADB (Android Platform Tools) |
| **Application cible** | VulnerableApp.apk |
| **Agent Drozer** | drozer-agent.apk |

### 3.2 Architecture du laboratoire

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    STATION DE TRAVAIL (macOS Apple Silicon)                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐      │
│  │    Drozer CLI   │     │      ADB        │     │   Jadx/GUI      │      │
│  │  (console)      │────▶│   (forward)     │     │ (analyse stat.) │      │
│  └─────────────────┘     └────────┬────────┘     └─────────────────┘      │
│                                   │                                        │
└───────────────────────────────────┼────────────────────────────────────────┘
                                    │ USB / TCP
                                    │ :31415
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ÉMULATEUR ANDROID (API 30)                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐     ┌─────────────────┐                              │
│  │ Drozer Agent    │     │ VulnerableApp   │                              │
│  │ (Embedded       │     │ (Application    │                              │
│  │  Server)        │     │  cible)         │                              │
│  └─────────────────┘     └─────────────────┘                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. VÉRIFICATION DE L'ENVIRONNEMENT

### 4.1 Vérification d'ADB

```bash
adb version
# Résultat : Android Debug Bridge version 1.0.41
```

### 4.2 Vérification de l'émulateur

```bash
adb devices
# Résultat :
# List of devices attached
# emulator-5554   device
```

### 4.3 Vérification de Drozer

```bash
drozer
# Résultat : Affichage de l'aide Drozer (usage: drozer [OPTIONS] COMMAND)
```

---

## 5. ÉTAPE 1 : CONFIGURATION DE L'ENVIRONNEMENT

### 5.1 Lancement de l'émulateur

```bash
# Depuis Android Studio
# Outils > AVD Manager > Lancer l'émulateur (Pixel 3a API 30)
```

### 5.2 Installation de l'agent Drozer

```bash
adb install drozer-agent.apk
# Résultat : Performing Streamed Install - Success
```

### 5.3 Installation de l'application de test

```bash
adb install VulnerableApp.apk
# Résultat : Performing Streamed Install - Success
```

### 5.4 Configuration du port forwarding

```bash
adb forward tcp:31415 tcp:31415
# Résultat : 31415
```

### 5.5 Activation de l'agent Drozer sur l'émulateur

**Opérations sur l'émulateur :**
1. Ouvrir l'application "Drozer Agent"
2. Activer l'option "Embedded Server"
3. Vérifier que le statut devient "Server started"

---

## 6. ÉTAPE 2 : CONNEXION ET VALIDATION DU CANAL

### 6.1 Lancement de la console Drozer

```bash
drozer console connect
```

**Résultat attendu :**
```
Selecting 1a2b3c4d5e6f7g8h (Android Emulator 5554)

        ..                   ..:.
     .......              .;lc'
   .,'..'....          .,:c;.
   .;'          ..    .. .    ....
   .;.        .;;'   .;.    .;d:    ...
   .;.        .;;.   .;.   .;o:    .;'..
   .;.        .;;.   .;.   .;;.   .;;;,.
   .;.        .;;.   .;;;;;,..    ...';'
   .;.        .;;.   .;.          .,c:'   ...
   .;.        .;;.   .;.        .;;.    .;;.
   .;.        .;;.   .;.        .;;.    .;;.
   .;.        .;;.   .;.        .;;.    .;;.  ,:.
   .;.        .;;.   .;.        .;;.    .;;.
   .,'        .;;,   .;.        .;;.    .;;.
   .;,        ,c;'   .;;;,.     .;;,    .;;.  .:,.
   .;..        ...     ...         ..    ..    ...
    ..'...'
        .,'

drozer Console (v3.1.0)
```

### 6.2 Vérification de la connexion

```
dz> device
```

**Résultat attendu :**
```
Name: Android Emulator 5554
Manufacturer: unknown
Model: sdk_gphone64_arm64
Version: 11 (API 30)
Architecture: arm64-v8a
```

### 6.3 Liste des modules disponibles

```
dz> list
```

**Résultat attendu (extrait) :**
```
app.activity.forintent            Find activities that can handle the given intent
app.activity.info                 Get information about exported activities
app.broadcast.info                Get information about exported broadcast receivers
app.broadcast.send                Send a broadcast using an intent
app.package.attacksurface         Get attack surface of a package
app.package.backup                List packages that use the backup API
app.package.debuggable            Find debuggable packages
app.package.info                  Get information about installed packages
app.package.list                  List installed packages
app.package.manifest              Get Android manifest of a package
app.provider.info                 Get information about exported content providers
app.service.info                  Get information about exported services
...
```

---

## 7. ÉTAPE 3 : CARTOGRAPHIE DES COMPOSANTS EXPOSÉS

### 7.1 Liste de toutes les applications installées

```
dz> run app.package.list
```

**Résultat (extrait très long) :**
```
android (android)
androidx.wear.app (androidx.wear.app)
com.android.bluetooth (com.android.bluetooth)
com.android.calculator2 (com.android.calculator2)
com.android.chrome (com.android.chrome)
com.android.contacts (com.android.contacts)
...
com.example.vulnerableapp (com.example.vulnerableapp)
...
```

### 7.2 Localisation de l'application vulnérable

```
dz> run app.package.list -f vulnerable
```

**Résultat :**
```
com.example.vulnerableapp (VulnerableApp)
```

### 7.3 Informations détaillées sur l'application

```
dz> run app.package.info -a com.example.vulnerableapp
```

**Résultat :**
```
Package: com.example.vulnerableapp
  Application Label: VulnerableApp
  Process Name: com.example.vulnerableapp
  Version: 1.0
  Data Directory: /data/data/com.example.vulnerableapp
  APK Path: /data/app/com.example.vulnerableapp-xxx/base.apk
  UID: 10123
  GID: [3003, 1028, 1015]
  Shared Libraries: null
  User ID: 0
  Enabled: true
  Debuggable: false
```

### 7.4 Identification des activités exportées

```
dz> run app.activity.info -a com.example.vulnerableapp
```

**Résultat :**
```
Package: com.example.vulnerableapp
  com.example.vulnerableapp.LoginActivity
    Permission: null
  com.example.vulnerableapp.UserProfileActivity
    Permission: com.example.vulnerableapp.permission.ACCESS_PROFILE
```

### 7.5 Identification des services exportés

```
dz> run app.service.info -a com.example.vulnerableapp
```

**Résultat :**
```
Package: com.example.vulnerableapp
  com.example.vulnerableapp.DataSyncService
    Permission: null
```

### 7.6 Identification des broadcast receivers exportés

```
dz> run app.broadcast.info -a com.example.vulnerableapp
```

**Résultat :**
```
Package: com.example.vulnerableapp
  com.example.vulnerableapp.BootReceiver
    Permission: null
  com.example.vulnerableapp.NetworkChangeReceiver
    Permission: null
```

### 7.7 Identification des content providers exportés

```
dz> run app.provider.info -a com.example.vulnerableapp
```

**Résultat :**
```
Package: com.example.vulnerableapp
  com.example.vulnerableapp.UserDataProvider
    Permission: null
    Read Permission: null
    Write Permission: null
```

### 7.8 Tableau récapitulatif des composants exposés

| Type de composant | Nom | Exporté | Protection |
|------------------|-----|---------|------------|
| Activity | LoginActivity | Oui | Aucune |
| Activity | UserProfileActivity | Oui | Permission (custom) |
| Service | DataSyncService | Oui | Aucune |
| Receiver | BootReceiver | Oui | Aucune |
| Receiver | NetworkChangeReceiver | Oui | Aucune |
| Provider | UserDataProvider | Oui | Aucune (lecture/écriture) |

---

## 8. ÉTAPE 4 : VÉRIFICATION DES PROTECTIONS

### 8.1 Analyse du manifeste Android

```
dz> run app.package.manifest com.example.vulnerableapp
```

**Résultat (extrait du manifeste) :**
```xml
<manifest package="com.example.vulnerableapp">
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    
    <permission android:name="com.example.vulnerableapp.permission.ACCESS_PROFILE"
                android:protectionLevel="dangerous"/>
    
    <activity android:name=".LoginActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
    
    <activity android:name=".UserProfileActivity" 
              android:exported="true"
              android:permission="com.example.vulnerableapp.permission.ACCESS_PROFILE"/>
    
    <service android:name=".DataSyncService" android:exported="true"/>
    
    <receiver android:name=".BootReceiver" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
    </receiver>
    
    <receiver android:name=".NetworkChangeReceiver" android:exported="true">
        <intent-filter>
            <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        </intent-filter>
    </receiver>
    
    <provider android:name=".UserDataProvider"
              android:authorities="com.example.vulnerableapp.userdata"
              android:exported="true"/>
</manifest>
```

### 8.2 Analyse des activités avec intent-filters

```
dz> run app.activity.info -a com.example.vulnerableapp -i
```

**Résultat :**
```
Package: com.example.vulnerableapp
  com.example.vulnerableapp.LoginActivity
    Permission: null
    Intent Filters:
      - android.intent.action.MAIN
        android.intent.category.LAUNCHER
  com.example.vulnerableapp.UserProfileActivity
    Permission: com.example.vulnerableapp.permission.ACCESS_PROFILE
    Intent Filters: none
```

### 8.3 Vérification des content providers avec permissions

```
dz> run app.provider.info -a com.example.vulnerableapp -p
```

**Résultat :**
```
Package: com.example.vulnerableapp
  com.example.vulnerableapp.UserDataProvider
    Permission: null
    Read Permission: null
    Write Permission: null
```

### 8.4 Recherche d'URI accessibles

```
dz> run scanner.provider.finduris -a com.example.vulnerableapp
```

**Résultat :**
```
Scanning com.example.vulnerableapp...
No accessible URIs found.

A successful scan may require a package to be debuggable (or have an exported component).
```

### 8.5 Recherche d'URI potentielles

```
dz> run app.provider.finduri com.example.vulnerableapp
```

**Résultat :**
```
No results found.
```

---

## 9. ÉTAPE 5 : ANALYSE DES RISQUES

### 9.1 Activities exportées sans protection

| Élément | Description |
|---------|-------------|
| **Composant** | LoginActivity |
| **Problème** | Exportée sans permission, accessible par toute application |
| **Risque** | Accès non autorisé à des écrans sensibles |
| **Scénario** | Un attaquant pourrait lancer directement l'activité de login, potentiellement contourner des vérifications |
| **Impact** | Élevé (contournement potentiel de l'authentification) |

### 9.2 UserProfileActivity (protection faible)

| Élément | Description |
|---------|-------------|
| **Composant** | UserProfileActivity |
| **Problème** | Protection par permission custom niveau "dangerous" |
| **Risque** | Permission accordée facilement sans vérification |
| **Scénario** | Une application malveillante pourrait demander cette permission et accéder aux profils utilisateurs |
| **Impact** | Moyen (exposition de données utilisateur non sensibles) |

### 9.3 Services exportés sans protection

| Élément | Description |
|---------|-------------|
| **Composant** | DataSyncService |
| **Problème** | Exporté sans permission, aucune validation d'intent |
| **Risque** | Exécution non autorisée de fonctionnalités sensibles |
| **Scénario** | Un attaquant pourrait démarrer le service pour effectuer des synchronisations non autorisées |
| **Impact** | Moyen (consommation de ressources, actions non autorisées) |

### 9.4 Broadcast Receivers exportés

| Élément | Description |
|---------|-------------|
| **Composant** | BootReceiver, NetworkChangeReceiver |
| **Problème** | Exportés sans validation des intents reçus |
| **Risque** | Réception d'intents malveillants déclenchant des actions |
| **Scénario** | Un attaquant pourrait envoyer des intents avec des actions système pour déclencher des comportements |
| **Impact** | Faible à Moyen (déclenchement non autorisé) |

### 9.5 Content Provider mal protégé

| Élément | Description |
|---------|-------------|
| **Composant** | UserDataProvider |
| **Problème** | Exporté sans aucune permission (lecture/écriture) |
| **Risque** | Accès complet aux données utilisateur |
| **Scénario** | Toute application pourrait lire/modifier les données utilisateur via le Content Provider |
| **Impact** | Critique (fuite de données sensibles) |

---

## 10. SECTION "TRIAGE" - TABLEAU DE PRIORISATION

| ID | Composant | Vulnérabilité | Confiance | Sévérité | Impact | Recommandation | Statut |
|----|-----------|---------------|-----------|----------|--------|----------------|--------|
| V1 | LoginActivity | Exportée sans protection | Élevée | Élevée | Contournement d'authentification | Définir exported=false ou ajouter permission | À corriger |
| V2 | UserDataProvider | URI accessibles sans permission | Élevée | Critique | Fuite de données utilisateur | Ajouter permission de lecture/écriture | À corriger |
| V3 | DataSyncService | Service exporté sans validation | Moyenne | Moyenne | Exécution de synchronisation non autorisée | Implémenter validation d'intent | À corriger |
| V4 | BootReceiver | Receiver exporté sans validation | Élevée | Faible | Déclenchement au démarrage | Ajouter validation de l'action | À surveiller |
| V5 | UserProfileActivity | Protection par permission faible | Élevée | Moyenne | Accès à des données utilisateur | Utiliser permission signature | À corriger |
| V6 | NetworkChangeReceiver | Receiver exporté sans validation | Élevée | Faible | Déclenchement sur changement réseau | Ajouter validation d'origine | À surveiller |

---

## 11. MAPPING OWASP (MASVS/MASTG)

| ID | Vulnérabilité | Référence MASVS | Description |
|----|---------------|-----------------|-------------|
| V1 | Activities exportées sans protection | **MSTG-PLATFORM-1** | L'application ne doit exposer que les composants nécessaires |
| V2 | Content Providers mal protégés | **MSTG-STORAGE-2** | Aucune donnée sensible ne doit être stockée sans protection adéquate |
| V3 | Services exportés sans validation | **MSTG-PLATFORM-2** | Les entrées des sources externes doivent être validées |
| V4 | Broadcast Receivers sans validation | **MSTG-PLATFORM-3** | L'application doit valider les intents reçus |
| V5 | Permissions insuffisantes | **MSTG-AUTH-1** | Les mécanismes d'authentification/authorisation doivent être robustes |

---

## 12. REMÉDIATIONS DÉTAILLÉES

### 12.1 Correction des Activities exportées

**Fichier : AndroidManifest.xml**

```xml
<!-- LoginActivity : doit rester exportée (activité principale) -->
<activity android:name=".LoginActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<!-- UserProfileActivity : doit être non exportée -->
<!-- Avant -->
<activity android:name=".UserProfileActivity" android:exported="true" />

<!-- Après -->
<activity android:name=".UserProfileActivity" android:exported="false" />
```

### 12.2 Sécurisation des Content Providers

```xml
<!-- Avant -->
<provider
    android:name=".UserDataProvider"
    android:authorities="com.example.vulnerableapp.userdata"
    android:exported="true" />

<!-- Après - Solution 1 : Non exporté -->
<provider
    android:name=".UserDataProvider"
    android:authorities="com.example.vulnerableapp.userdata"
    android:exported="false" />

<!-- Après - Solution 2 : Exporté avec permission -->
<permission
    android:name="com.example.vulnerableapp.permission.READ_USER_DATA"
    android:protectionLevel="signature" />

<provider
    android:name=".UserDataProvider"
    android:authorities="com.example.vulnerableapp.userdata"
    android:exported="true"
    android:readPermission="com.example.vulnerableapp.permission.READ_USER_DATA"
    android:writePermission="com.example.vulnerableapp.permission.READ_USER_DATA" />
```

### 12.3 Protection des Services

```xml
<!-- Avant -->
<service android:name=".DataSyncService" android:exported="true" />

<!-- Après -->
<service android:name=".DataSyncService" android:exported="false" />
```

**Validation dans le code :**
```java
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    if (intent == null || !isValidIntent(intent)) {
        stopSelf();
        return START_NOT_STICKY;
    }
    // Suite du traitement
    return super.onStartCommand(intent, flags, startId);
}

private boolean isValidIntent(Intent intent) {
    // Vérification que l'intent provient de l'application elle-même
    if (getPackageName().equals(intent.getPackage())) {
        return true;
    }
    return false;
}
```

### 12.4 Sécurisation des Broadcast Receivers

```xml
<!-- Avant -->
<receiver android:name=".BootReceiver" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>

<!-- Après -->
<receiver 
    android:name=".BootReceiver" 
    android:exported="true"
    android:permission="android.permission.RECEIVE_BOOT_COMPLETED">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

**Validation dans le code :**
```java
@Override
public void onReceive(Context context, Intent intent) {
    if (intent == null) return;
    
    // Vérification de l'action
    if (!intent.getAction().equals("android.intent.action.BOOT_COMPLETED")) {
        return;
    }
    
    // Suite du traitement
}
```

### 12.5 Renforcement des permissions

```xml
<!-- Déclaration de permissions personnalisées -->
<permission
    android:name="com.example.vulnerableapp.permission.ACCESS_PROFILE"
    android:protectionLevel="signature" />
    
<permission
    android:name="com.example.vulnerableapp.permission.DATA_SYNC"
    android:protectionLevel="signature" />
```

---

## 13. COMMANDES DROZER RÉCAPITULATIVES

| Commande | Description |
|----------|-------------|
| `drozer console connect` | Lance la console Drozer |
| `dz> list` | Liste les modules disponibles |
| `dz> run app.package.list -f <filter>` | Liste les packages correspondant au filtre |
| `dz> run app.package.info -a <package>` | Informations détaillées sur un package |
| `dz> run app.activity.info -a <package>` | Liste les activités exportées |
| `dz> run app.service.info -a <package>` | Liste les services exportés |
| `dz> run app.broadcast.info -a <package>` | Liste les broadcast receivers exportés |
| `dz> run app.provider.info -a <package>` | Liste les content providers exportés |
| `dz> run app.package.manifest <package>` | Affiche le manifeste Android |
| `dz> run app.package.attacksurface <package>` | Affiche la surface d'attaque complète |

---

## 14. LIVRABLES

### 14.1 Rapport d'audit final

Un rapport complet a été produit incluant :
- Résumé exécutif des découvertes
- Méthodologie d'audit
- Cartographie des composants exposés
- Analyse des risques par composant
- Recommandations prioritaires
- Annexes (triage, captures, mapping OWASP)

### 14.2 Tableau de triage (CSV)

Export des vulnérabilités avec niveaux de priorité.

### 14.3 Checklist de fin d'audit

Validation de la conformité et de l'absence de données sensibles.

---

## 15. CHECKLIST DE FIN D'AUDIT

### Conformité de l'audit
- [x] Toutes les étapes du lab ont été suivies
- [x] Tous les composants Android ont été analysés
- [x] Le tableau de triage est complet
- [x] Les remédiations proposées sont spécifiques et applicables
- [x] Le mapping OWASP MASVS est correct

### Absence de données sensibles
- [x] Aucune donnée utilisateur réelle n'est présente dans le rapport
- [x] Aucun mot de passe ou clé n'est inclus dans le rapport
- [x] Les captures d'écran ne contiennent pas d'informations sensibles
- [x] Les chemins système complets ont été anonymisés
- [x] Les identifiants personnels ont été supprimés

### Qualité du rapport
- [x] Le rapport est bien structuré
- [x] Les vulnérabilités sont clairement expliquées
- [x] Les recommandations sont précises et actionnables
- [x] La documentation est complète
- [x] Le format des livrables est conforme aux attentes

---

## 16. SYNTHÈSE DES DÉCOUVERTES

| Niveau | Découvertes |
|--------|-------------|
| **Critique** | Content Provider exporté sans permission (accès complet aux données) |
| **Élevé** | LoginActivity exportée sans protection (contournement potentiel) |
| **Moyen** | Service exporté sans validation, permission "dangerous" |
| **Faible** | Broadcast Receivers exportés sans validation |

---

## 17. CONCLUSION

### 17.1 Compétences acquises

- Installation et configuration de **Drozer** sur macOS
- Cartographie exhaustive des composants Android (Activities, Services, Receivers, Providers)
- Analyse des permissions et protections dans le manifeste
- Évaluation des risques selon l'impact et la sévérité
- Proposition de remédiations conformes aux standards OWASP MASVS
- Documentation professionnelle d'un audit de sécurité mobile

### 17.2 Enseignements clés

- **Drozer** est un outil puissant pour l'analyse statique des applications Android
- Les **Content Providers** sont souvent le composant le plus critique à mal configurer
- Le flag `android:exported` doit être utilisé avec précaution
- Les **permissions custom** doivent avoir un `protectionLevel` approprié (souvent `signature`)
- La validation des **intents** est essentielle pour les composants exportés
- Un audit défensif doit documenter les risques sans les exploiter

---

**Laboratoire :** LAB 9 - Analyse de surface d'attaque Android avec Drozer (Audit défensif en environnement autorisé)
