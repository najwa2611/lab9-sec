# Audit de sécurité d'application Android – Analyse des composants exposés

## Objectifs de l’audit

- Maîtriser l’utilisation d’un outil d’analyse de sécurité pour applications Android  
- Identifier les composants Android exposés et leurs vulnérabilités potentielles  
- Évaluer les risques liés à des configurations inadéquates  
- Documenter méthodiquement les résultats d’un audit de sécurité  
- Proposer des remédiations conformes aux standards de l’industrie (OWASP MASVS)

## Périmètre et cadre

Cet audit s’inscrit dans une démarche défensive et autorisée. L’analyse est réalisée sur un environnement isolé (émulateur contrôlé) avec une application conçue à des fins pédagogiques.

**Règles applicables :**
- Aucune exploitation offensive des vulnérabilités  
- Aucune donnée réelle manipulée  
- Analyse strictement limitée à l’environnement de test

**Périmètre technique :**
- Émulateur Android  
- Application de test (non publiée)  
- Composants Android exposés  
- Configuration du manifeste

## Prérequis techniques

- Windows, macOS ou Linux  
- Émulateur Android configuré (API 28 ou ultérieur)  
- ADB fonctionnel  
- Outil d’analyse Drozer installé sur la machine hôte  
- Agent Drozer installé sur l’émulateur  
- Application de test fournie

## Vérification de l’environnement

```bash
adb version
adb devices
drozer
```

L’émulateur doit être détecté avec le statut `device`.  
La console Drozer doit afficher son aide.

## Architecture de l’environnement

```
Machine hôte (Drozer console) ←→ ADB ←→ Émulateur Android (Agent Drozer + Application cible)
```

## Déroulement de l’audit

### 1. Configuration initiale

- Lancement de l’émulateur  
- Installation de l’agent Drozer et de l’application cible  
- Activation du serveur intégré dans l’agent Drozer  
- Configuration du redirection de port ADB :  
  ```bash
  adb forward tcp:31415 tcp:31415
  ```

### 2. Connexion à Drozer

```bash
drozer console connect
dz> device
dz> list
dz> run information.device
```

### 3. Cartographie des composants exposés

Recherche de l’application cible :

```bash
dz> run app.package.list -f <motif>
dz> run app.package.info -a <package>
```

Inventaire des composants exportés :

```bash
dz> run app.activity.info -a <package>
dz> run app.service.info -a <package>
dz> run app.broadcast.info -a <package>
dz> run app.provider.info -a <package>
```

### 4. Vérification des protections

```bash
dz> run app.package.manifest <package>
dz> run app.provider.finduri <package>
dz> run scanner.provider.finduris -a <package>
```

### 5. Analyse des risques par type de composant

| Composant | Risque principal | Scénario potentiel |
|-----------|----------------|---------------------|
| Activité exportée | Accès non autorisé | Contournement d’authentification |
| Service exporté | Exécution de fonctions sensibles | Opérations privilégiées non autorisées |
| Broadcast Receiver | Déclenchement d’actions malveillantes | Intents forgés |
| Content Provider | Fuite ou modification de données | Accès direct aux données |
| Permission faible | Protection inadéquate | Élévation de privilèges |

## Triage des vulnérabilités

| ID | Composant | Vulnérabilité | Sévérité | Impact | Recommandation |
|----|------------|----------------|-----------|---------|----------------|
| V1 | Activité sans protection | Exportée sans restriction | Élevée | Contournement d’auth | `exported=false` |
| V2 | Content Provider | URI accessible sans permission | Critique | Fuite de données | Ajouter permission |
| V3 | Service exporté | Absence de validation | Moyenne | Exécution indue | Valider les intents |
| V4 | Broadcast Receiver | Exporté sans validation | Faible | Déclenchement intempestif | Valider l’intent |
| V5 | Activité avec permission faible | Protection insuffisante | Moyenne | Accès aux données | Permission renforcée |

## Correspondance avec OWASP MASVS

| Vulnérabilité | Référence MASVS |
|---------------|------------------|
| Activités exportées | MSTG-PLATFORM-1 |
| Content Provider non sécurisé | MSTG-STORAGE-2 |
| Service exporté | MSTG-PLATFORM-2 |
| Receiver non validé | MSTG-PLATFORM-3 |
| Permission insuffisante | MSTG-AUTH-1 |
