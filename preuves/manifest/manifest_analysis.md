# Analyse du Manifeste
- **Permissions requises** : L'application définit des composants exposés (`android:exported="true"`) sans imposer de permissions de niveau `signature` adéquates.
- **Intent-filters** : Les activités, services et receivers possèdent des intent-filters qui les rendent par défaut exportés ou accessibles publiquement.
- **Vulnérabilité principale** : L'absence de restriction stricte sur les composants expose l'application à des attaques de type "Privilege Escalation" et "Data Leakage".
