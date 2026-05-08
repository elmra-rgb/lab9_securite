# Risques - Activities
**Composant :** com.example.vulnerableapp.LoginActivity
- **État d'exportation :** Exporté (Oui)
- **Protections en place :** Aucune
- **Risques identifiés :** Accès non autorisé à des écrans sensibles. Un attaquant pourrait lancer directement cette activité interne, contournant l'authentification.

**Composant :** com.example.vulnerableapp.UserProfileActivity
- **État d'exportation :** Exporté (Oui)
- **Protections en place :** Permission (faible)
- **Risques identifiés :** Protection inadéquate. Des composants protégés par des permissions trop faibles pourraient être accessibles.
