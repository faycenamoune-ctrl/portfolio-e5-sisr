# 🛡️ Portfolio E5 — BTS SIO SISR
## Apprenti Technicien Systèmes, Réseaux & Support

> **Fondation Saint Jean de Dieu** — Secteur Médico-Social, Échelle Nationale  
> *Alternance BTS SIO — Option SISR — 1ère Année*

---

## 👤 Présentation

| Informations | Détail |
|---|---|
| **Formation** | BTS Services Informatiques aux Organisations — Option SISR |
| **Entreprise** | Fondation Saint Jean de Dieu |
| **Secteur** | Médico-Social (établissements nationaux) |
| **Poste** | Apprenti Technicien Systèmes, Réseaux & Support |
| **Environnement** | Windows Server · Active Directory · M365 · Azure · WithSecure · GLPI |

---

## 🗂️ Sommaire des Situations Professionnelles

| # | Mission | Compétences clés |
|---|---|---|
| [SP1](#sp1) | Migration nationale vers Office 365 & gestion Active Directory | SISR1, SISR4 |
| [SP2](#sp2) | Administration réseau, VPN et réponse à incident de sécurité | SISR2, SISR3 |
| [SP3](#sp3) | Automatisation PowerShell & Support N1/N2 via GLPI | SISR1, SISR5 |
| [SP4](#sp4) | Déploiement applicatif métier Netvie/Netfactu | SISR4, SISR5 |

---

## 🔧 Environnement Technique Global

```
Infrastructure
├── Serveurs       : Windows Server 2019/2022, Active Directory DS
├── Cloud          : Microsoft 365, Azure AD (Entra ID), Exchange Online
├── Sécurité       : WithSecure Elements, Pare-feu, VPN site-à-site
├── ITSM           : GLPI (ticketing, inventaire, CMDB)
├── Métier         : Octime (RH), CEGI (Paie), Netvie, Netfactu
└── Réseau         : Segmentation VLAN, DNS, DHCP, GPO
```

---

<a id="sp1"></a>
## 📋 SP1 — Migration Nationale vers Office 365 & Gestion de l'Active Directory

### 1. Titre de la Mission
**Industrialisation de la migration M365 et harmonisation de l'annuaire Active Directory sur l'ensemble des établissements de la Fondation Saint Jean de Dieu**

---

### 2. Contexte et Enjeux

La **Fondation Saint Jean de Dieu** est un opérateur national du secteur médico-social regroupant plusieurs dizaines d'établissements (EHPAD, IME, ESAT, etc.) répartis sur l'ensemble du territoire français. Chaque site dispose de son propre environnement informatique historique, avec des versions hétérogènes d'Office (2013, 2016, 2019) et des annuaires Active Directory cloisonnés.

**Enjeux identifiés :**

- **Continuité de service médicale** : les soignants et personnels administratifs doivent accéder à leurs outils à tout moment — une interruption de service, même brève, a un impact direct sur la prise en charge des patients.
- **Conformité réglementaire** : le secteur médical est soumis au **RGPD** et aux recommandations de l'**ANSSI** (Agence Nationale de la Sécurité des Systèmes d'Information) concernant la protection des données de santé.
- **Harmonisation nationale** : standardiser les pratiques sur l'ensemble des établissements afin de réduire les coûts de maintenance et de simplifier le support.
- **Gain de temps estimé** : réduction de **60 à 70 %** du temps de provisionnement d'un nouveau compte utilisateur grâce à l'automatisation via PowerShell et les modèles GPO.

---

### 3. Compétences Visées (Référentiel BTS SIO — SISR)

- **SISR1** — Gérer le patrimoine informatique *(gestion des comptes, droits, licences M365)*
- **SISR4** — Travailler en mode projet *(migration planifiée par vagues d'établissements)*
- **SISR5** — Mettre à disposition des utilisateurs un service informatique *(déploiement M365, formation)*
- **B1.1** — Recenser et identifier les ressources numériques
- **B1.4** — Déployer et configurer les équipements et logiciels

---

### 4. Solutions Techniques Mises en Œuvre

#### 4.1 — Audit et Cartographie de l'existant

Avant toute migration, une phase d'inventaire a été réalisée à l'aide de **GLPI** et d'un script PowerShell d'export de l'Active Directory :

```powershell
# Export des comptes utilisateurs Active Directory vers CSV
Import-Module ActiveDirectory

Get-ADUser -Filter * -Properties DisplayName, EmailAddress, Department, `
    Enabled, LastLogonDate, DistinguishedName |
Select-Object DisplayName, SamAccountName, EmailAddress, `
    Department, Enabled, LastLogonDate |
Export-Csv -Path "C:\Audit\Export_AD_$(Get-Date -Format 'yyyyMMdd').csv" `
    -Encoding UTF8 -NoTypeInformation

Write-Host "[OK] Export termine : $((Get-ADUser -Filter *).Count) comptes traites."
```

Ce script a permis de générer un **tableau de bord Excel** recensant l'ensemble des comptes (actifs, inactifs, orphelins) sur tous les établissements, facilitant le nettoyage préalable à la migration.

#### 4.2 — Structure de l'Active Directory et Unités Organisationnelles

La refonte de l'AD a suivi une **arborescence OU normalisée** par établissement et par fonction :

```
DC=fondation-sjdd,DC=local
├── OU=Siege
│   ├── OU=Informatique
│   ├── OU=Direction
│   └── OU=RH
├── OU=Etablissements
│   ├── OU=EHPAD-Lyon
│   │   ├── OU=Soignants
│   │   ├── OU=Administratif
│   │   └── OU=Stagiaires
│   ├── OU=IME-Paris
│   └── OU=ESAT-Bordeaux
└── OU=Comptes_Service
```

#### 4.3 — Gestion des Droits NTFS et Partages Réseau

Application du **principe du moindre privilège** sur les partages de fichiers :

| Groupe AD | Dossier | Droits NTFS | Justification |
|---|---|---|---|
| `GG_Soignants_Lyon` | `\\SRV-FICHIERS\Soins` | Lecture/Écriture | Accès aux dossiers patients |
| `GG_Direction` | `\\SRV-FICHIERS\Direction` | Contrôle Total | Documents stratégiques |
| `GG_Stagiaires` | `\\SRV-FICHIERS\Public` | Lecture seule | Limitation des risques |
| `GG_Informatique` | `\\SRV-FICHIERS\IT` | Contrôle Total | Administration système |

#### 4.4 — Synchronisation Azure AD Connect et Migration M365

- Configuration d'**Azure AD Connect** (aujourd'hui *Microsoft Entra Connect*) pour la synchronisation des identités on-premise vers le cloud.
- Activation de la **synchronisation de hachage de mot de passe** (PHS) pour assurer la continuité d'accès.
- Attribution des **licences M365 Business Premium** via des groupes dynamiques Azure AD, permettant une gestion automatisée par département.
- Déploiement de **Microsoft 365 Apps** via **Intune** avec des profils de configuration prédéfinis par rôle métier.

---

### 5. Analyse Cybersécurité

| Risque identifié | Mesure mise en œuvre | Outil / Méthode |
|---|---|---|
| Comptes orphelins post-migration | Audit AD + désactivation automatique | Script PowerShell + tâche planifiée |
| Accès non autorisé aux données patients | Droits NTFS granulaires + groupes AD | GPO + Active Directory |
| Compromission de compte cloud | Authentification Multi-Facteurs (MFA) | Azure AD / Entra ID |
| Fuite de données lors de la migration | Chiffrement en transit (TLS 1.2/1.3) | M365 + protocoles sécurisés |
| Élévation de privilèges | Séparation comptes utilisateurs / admin | Modèle Tier Active Directory |

> **Point de conformité RGPD** : les données de santé des patients ne transitent jamais hors du périmètre souverain. Les boîtes mail soignants ont été configurées avec des étiquettes de **sensibilité Microsoft Purview** pour éviter tout partage externe non autorisé.

---

<a id="sp2"></a>
## 📋 SP2 — Administration Réseau, Sécurité et Réponse à Incident

### 1. Titre de la Mission
**Renforcement de l'architecture réseau par segmentation, administration des VPN inter-sites et gestion d'un incident de compromission d'un poste utilisateur**

---

### 2. Contexte et Enjeux

Dans un réseau médical multi-sites, la **segmentation** et la **cloisonnement des flux** sont des impératifs de sécurité. Un établissement médical est une cible privilégiée des cyberattaques (ransomwares, phishing) en raison de la criticité des données et de la pression opérationnelle qui pousse parfois à contourner les règles de sécurité.

Au cours de mon alternance, un **incident de sécurité réel** a été détecté sur le poste d'un salarié : une alerte **WithSecure Elements** a signalé l'exécution d'un processus suspect, potentiellement lié à une tentative d'infection par un ransomware via une pièce jointe malveillante.

**Enjeux critiques :**
- Protéger **les données médicales** (dossiers patients, données RH sensibles) d'une propagation latérale.
- Maintenir la **disponibilité des applications métier** (Octime, Netvie) durant l'incident.
- Respecter les obligations de **notification CNIL** en cas de violation de données de santé (72h).

---

### 3. Compétences Visées

- **SISR2** — Gérer les incidents et les problèmes *(isolation, investigation, remédiation)*
- **SISR3** — Mettre en place, assurer et tester la sécurité d'un réseau *(VPN, segmentation)*
- **B2.2** — Gérer des indicateurs et tableaux de bord
- **B2.3** — Assurer la cybersécurité d'une solution informatique

---

### 4. Solutions Techniques Mises en Œuvre

#### 4.1 — Architecture de Segmentation Réseau

```
Internet
    │
    ▼
[Pare-feu UTM]
    │
    ├── VLAN 10 — Administration IT (10.10.10.0/24)
    │       └── Accès complet + console d'admin
    │
    ├── VLAN 20 — Postes Utilisateurs (10.10.20.0/24)
    │       └── Accès Internet filtré + ressources métier
    │
    ├── VLAN 30 — Serveurs (10.10.30.0/24)
    │       └── Accès restreint, pas d'accès Internet direct
    │
    ├── VLAN 40 — Imprimantes / IoT Médical (10.10.40.0/24)
    │       └── Isolation totale, flux contrôlés
    │
    └── VLAN 50 — Invités / WiFi Public (10.10.50.0/24)
            └── Internet uniquement, DMZ
```

#### 4.2 — Gestion des VPN Inter-Sites

- Configuration de **VPN site-à-site IPsec/IKEv2** entre le siège et les établissements régionaux pour sécuriser les flux de données des applications métier.
- Déploiement du client **VPN Always-On (Always On VPN)** via Intune pour les postes nomades du personnel administratif, garantissant un tunnel permanent vers le SI de la Fondation.
- Paramétrage des **règles de routage et de filtrage** pour que seul le trafic applicatif nécessaire emprunte le tunnel VPN (split tunneling contrôlé).

#### 4.3 — Gestion de l'Incident de Sécurité (Poste Compromis)

**Chronologie de la réponse à incident :**

```
J+0 / 09h14  → Alerte WithSecure : détection comportementale (processus PowerShell 
               encodé lancé depuis Outlook)
J+0 / 09h20  → Escalade vers le responsable SSI — ouverture ticket GLPI priorité CRITIQUE
J+0 / 09h25  → Isolation réseau immédiate du poste (désactivation port switch + 
               quarantaine WithSecure)
J+0 / 09h30  → Sauvegarde forensique de la RAM et des logs événements Windows
J+0 / 10h00  → Analyse des logs : identification du fichier malveillant (macro Excel)
J+0 / 11h30  → Désinfection complète + réinstallation OS depuis image gold
J+0 / 14h00  → Contrôle des accès AD du compte compromis + réinitialisation mdp
J+1 / 09h00  → Rapport d'incident + recommandations préventives transmis à la Direction
```

**Actions techniques réalisées :**

```powershell
# Désactivation immédiate du compte AD compromis
Disable-ADAccount -Identity "jdupont"
Write-EventLog -LogName "Security" -Source "Incident-SSI" `
    -EventId 9999 -Message "Compte jdupont desactive suite incident securite J+0"

# Réinitialisation forcée du mot de passe
Set-ADAccountPassword -Identity "jdupont" `
    -NewPassword (ConvertTo-SecureString "TempP@ss2024!" -AsPlainText -Force) `
    -Reset
Set-ADUser -Identity "jdupont" -ChangePasswordAtLogon $true
```

---

### 5. Analyse Cybersécurité

**Vecteur d'attaque identifié :** Macro VBA malveillante embarquée dans un fichier `.xlsm` reçu par email, simulant une facture fournisseur (technique de *spear phishing*).

| Phase MITRE ATT&CK | Technique | Contre-mesure appliquée |
|---|---|---|
| **Initial Access** | Phishing (T1566) | Filtrage anti-spam Exchange Online, formation utilisateurs |
| **Execution** | Macro Office malveillante (T1203) | Désactivation macros non signées via GPO |
| **Persistence** | Clé de registre Run (T1547) | Surveillance WithSecure + Wazuh |
| **Lateral Movement** | Pass-the-Hash (tentative) | Isolation réseau VLAN + désactivation compte |
| **Exfiltration** | Bloquée | Pare-feu sortant + règles DLP M365 |

> **Bilan** : L'incident a été **contenu en moins de 20 minutes** grâce à la segmentation VLAN et à la réactivité de la solution WithSecure Elements. Aucune donnée patient n'a été compromise. Un rapport a été transmis à la CNIL à titre de précaution, conformément à l'article 33 du RGPD.

---

<a id="sp3"></a>
## 📋 SP3 — Automatisation PowerShell & Support Utilisateurs N1/N2

### 1. Titre de la Mission
**Industrialisation du support technique via des scripts d'automatisation et gestion du ticketing GLPI pour l'ensemble des établissements**

---

### 2. Contexte et Enjeux

Avec plusieurs dizaines d'établissements à gérer depuis un support centralisé, la **répétabilité** et l'**automatisation** des tâches récurrentes constituent un enjeu majeur de productivité. Les demandes récurrentes (création de comptes, montage de lecteurs réseau, inventaire matériel) représentaient auparavant plusieurs heures de travail manuel par semaine.

**Objectif** : Réduire le temps de traitement des demandes récurrentes de **80 %** et homogénéiser le niveau de service sur l'ensemble du parc.

---

### 3. Compétences Visées

- **SISR1** — Gérer le patrimoine informatique *(inventaire, CMDB GLPI)*
- **SISR5** — Mettre à disposition des utilisateurs un service informatique *(SLA, base de connaissance)*
- **B1.5** — Gérer les identités et les habilitations
- **B2.1** — Recueillir, suivre et orienter des demandes

---

### 4. Solutions Techniques Mises en Œuvre

#### 4.1 — Script d'Inventaire Automatique et Export Excel

```powershell
# ============================================================
# Script : Inventaire_Parc_SJDD.ps1
# Objet  : Génère un rapport Excel du parc informatique
# Auteur : Apprenti IT - Fondation Saint Jean de Dieu
# ============================================================

$computers = Get-ADComputer -Filter * -Properties *
$results   = @()

foreach ($pc in $computers) {
    try {
        $os  = Get-WmiObject -Class Win32_OperatingSystem `
               -ComputerName $pc.Name -ErrorAction Stop
        $cpu = Get-WmiObject -Class Win32_Processor `
               -ComputerName $pc.Name -ErrorAction Stop
        $ram = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)

        $results += [PSCustomObject]@{
            Nom          = $pc.Name
            OS           = $os.Caption
            Version      = $os.Version
            RAM_Go       = $ram
            Processeur   = $cpu.Name
            Statut       = "OK"
            DerniereMAJ  = $os.LastBootUpTime
            Etablissement= ($pc.DistinguishedName -split ',OU=')[2]
        }
    } catch {
        $results += [PSCustomObject]@{
            Nom          = $pc.Name
            Statut       = "INACCESSIBLE"
            Etablissement= ($pc.DistinguishedName -split ',OU=')[2]
        }
    }
}

# Export CSV + rapport de synthèse
$date = Get-Date -Format "yyyy-MM-dd"
$results | Export-Csv "C:\Rapports\Inventaire_$date.csv" -Encoding UTF8 -NoTypeInformation
Write-Host "[RAPPORT] $($results.Count) machines traitees - Fichier : Inventaire_$date.csv"
```

#### 4.2 — Script de Montage Automatique des Lecteurs Réseau (GPO + Logon Script)

```powershell
# ============================================================
# Script : Montage_Lecteurs.ps1 — Déploiement via GPO Logon
# Logique : Attribution dynamique selon groupe AD de l'utilisateur
# ============================================================

$user    = $env:USERNAME
$groups  = (New-Object System.DirectoryServices.DirectorySearcher(
              "(&(sAMAccountName=$user))")).FindOne().Properties['memberof']

# Suppression des lecteurs existants pour repartir d'un état propre
Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -like "\\*" } |
    ForEach-Object { Remove-PSDrive $_.Name -Force -ErrorAction SilentlyContinue }

# Montage conditionnel selon appartenance groupe
if ($groups -match "GG_Soignants") {
    New-PSDrive -Name "S" -PSProvider FileSystem `
        -Root "\\SRV-FICHIERS\Soins" -Persist -Scope Global
    Write-Host "[OK] Lecteur S: Monte -> \\SRV-FICHIERS\Soins"
}

if ($groups -match "GG_Direction") {
    New-PSDrive -Name "D" -PSProvider FileSystem `
        -Root "\\SRV-FICHIERS\Direction" -Persist -Scope Global
    Write-Host "[OK] Lecteur D: Monte -> \\SRV-FICHIERS\Direction"
}

# Lecteur commun pour tous les utilisateurs
New-PSDrive -Name "P" -PSProvider FileSystem `
    -Root "\\SRV-FICHIERS\Public" -Persist -Scope Global
Write-Host "[OK] Lecteur P: Monte -> \\SRV-FICHIERS\Public"
```

#### 4.3 — Organisation du Ticketing GLPI

**Catégories de tickets configurées :**

| Catégorie GLPI | Priorité par défaut | SLA | Volume mensuel estimé |
|---|---|---|---|
| Incident — Poste inaccessible | Haute | 4h | ~15 tickets |
| Demande — Création compte AD | Normale | 24h | ~20 tickets |
| Incident — Application métier | Haute | 2h | ~8 tickets |
| Demande — Accès / Droits | Normale | 48h | ~12 tickets |
| Incident — Sécurité | Critique | 1h | <5 tickets |

**Processus de traitement N1/N2 :**

```
Utilisateur → Ticket GLPI
                │
                ▼
          [N1 - Support]
          Diagnostic initial
          Résolution si connue ──→ Fermeture ticket + MaJ base de connaissance
                │
                │ (si non résolu)
                ▼
          [N2 - Technicien]
          Investigation approfondie
          Intervention sur site/à distance (TeamViewer / RDP)
                │
                │ (si problème systémique)
                ▼
          [N3 - Administrateur]
          Modification infrastructure
```

---

### 5. Analyse Cybersécurité

- **Journalisation des actions scripts** : chaque exécution est loguée dans l'Observateur d'Événements Windows (`Write-EventLog`) pour assurer la **traçabilité** conformément aux bonnes pratiques ANSSI.
- **Sécurisation des scripts GPO** : les scripts de logon sont stockés dans `SYSVOL` avec des **permissions NTFS restrictives** (lecture seule pour les utilisateurs, modification réservée aux admins IT).
- **Gestion des secrets** : les mots de passe de service ne sont jamais écrits en clair dans les scripts — utilisation de **Windows Credential Manager** ou des **secrets Azure Key Vault** pour les automatisations cloud.
- **Audit des droits GLPI** : revue trimestrielle des comptes GLPI avec suppression des comptes inactifs — principe de moindre privilège appliqué aux rôles techniciens.

---

<a id="sp4"></a>
## 📋 SP4 — Déploiement d'Applications Métier Netvie / Netfactu

### 1. Titre de la Mission
**Déploiement, paramétrage et support des applications de gestion comptable et médicale Netvie et Netfactu dans un contexte de conformité réglementaire**

---

### 2. Contexte et Enjeux

**Netvie** et **Netfactu** sont des logiciels spécialisés dans la gestion des **résidents d'EHPAD** (admissions, facturation, suivi médico-social) et la **comptabilité analytique** des établissements. Leur déploiement sur plusieurs sites implique des contraintes fortes :

- **Criticité métier haute** : la facturation des séjours résidents ne peut souffrir aucune interruption — les erreurs impactent directement la trésorerie de la Fondation.
- **Conformité comptable** : intégration avec **CEGI** (logiciel de paie/comptabilité) et respect des **normes M14/M22** applicables aux établissements médico-sociaux publics.
- **Multi-sites** : le déploiement doit être reproductible à l'identique sur chaque établissement tout en préservant la **spécificité des paramètres locaux** (tarifs journaliers, conventions, codes analytiques).

---

### 3. Compétences Visées

- **SISR4** — Travailler en mode projet *(planification, recette, déploiement)*
- **SISR5** — Mettre à disposition des utilisateurs un service informatique *(formation, documentation)*
- **B1.4** — Déployer et configurer les équipements et logiciels
- **B1.6** — Organiser son développement professionnel

---

### 4. Solutions Techniques Mises en Œuvre

#### 4.1 — Analyse Préalable et Plan de Déploiement

**Prérequis techniques vérifiés avant installation :**

```powershell
# Vérification des prérequis système pour Netvie/Netfactu
$checks = @{}

# .NET Framework requis : 4.8 minimum
$dotnet = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full").Release
$checks["DotNet_4.8"] = if ($dotnet -ge 528040) { "OK ✓" } else { "KO - Mise a jour requise" }

# SQL Server Express / Full requis
$sql = Get-Service -Name "MSSQLSERVER" -ErrorAction SilentlyContinue
$checks["SQL_Server"] = if ($sql.Status -eq "Running") { "OK ✓" } else { "KO - Service arrete" }

# Espace disque (minimum 20 Go pour base de données)
$disk = Get-PSDrive C
$freeGB = [math]::Round($disk.Free / 1GB, 1)
$checks["Disque_Libre"] = if ($freeGB -ge 20) { "OK - $freeGB Go libres" } else { "KO - $freeGB Go" }

# Affichage du rapport
$checks.GetEnumerator() | ForEach-Object {
    Write-Host "[$($_.Value)] $($_.Key)"
}
```

#### 4.2 — Procédure de Déploiement Standardisée

**Étapes du déploiement type (reproductible sur chaque établissement) :**

1. **Préparation** — Snapshot VM pré-installation, vérification prérequis (script ci-dessus), sauvegarde base SQL existante.
2. **Installation** — Exécution du setup Netvie/Netfactu en mode administrateur, sélection du profil établissement.
3. **Paramétrage base** — Création de la base SQL dédiée, configuration des connexions ODBC, paramétrage des **codes analytiques** propres à l'établissement.
4. **Intégration CEGI** — Configuration des exports automatiques pour la **comptabilité analytique** (flux XML/CSV vers CEGI), test de bout en bout.
5. **Recette fonctionnelle** — Validation avec le référent métier (comptable ou directeur administratif) via une grille de tests prédéfinie.
6. **Formation utilisateurs** — Session de 2h avec les utilisateurs clés, remise d'un guide de démarrage rapide.
7. **Passage en production** — Activation en production, surveillance pendant 48h, hotline dédiée.

#### 4.3 — Gestion des Droits d'Accès Applicatifs

| Profil utilisateur | Droits Netvie | Droits Netfactu | Justification |
|---|---|---|---|
| Directeur administratif | Administration totale | Administration totale | Responsabilité comptable |
| Comptable | Saisie + validation | Saisie + édition factures | Mission fonctionnelle |
| Soignant référent | Consultation dossiers | Aucun | Besoin métier restreint |
| Support IT | Accès technique DB | Accès technique DB | Maintenance uniquement |

#### 4.4 — Supervision et Maintenance

- Mise en place de **tâches planifiées** pour les sauvegardes quotidiennes des bases SQL vers un NAS sécurisé.
- Configuration des **alertes mail** (via SQL Server Agent) en cas d'échec de sauvegarde.
- Intégration dans **GLPI** d'une fiche inventaire pour chaque installation Netvie/Netfactu (version, établissement, contacts référents).

---

### 5. Analyse Cybersécurité

| Risque | Impact | Mesure préventive |
|---|---|---|
| Accès non autorisé aux données de facturation | Fraude, violation RGPD | Authentification applicative + cloisonnement VLAN serveurs |
| Corruption de la base SQL | Perte de données comptables | Sauvegardes quotidiennes automatisées + test de restauration mensuel |
| Interception des flux comptables | Fuite données financières | Chiffrement TLS des flux entre application et base de données |
| Mise à jour non maîtrisée | Régression fonctionnelle | Environnement de recette dédié + gestion des versions |
| Accès root à la base SQL | Élévation de privilèges | Comptes SQL dédiés par application, pas de compte `sa` en production |

> **Conformité M14/M22** : Les exports vers CEGI ont été configurés avec des **contrôles de cohérence** (rapprochement des totaux) pour garantir l'intégrité des données comptables transmises. Un journal d'audit des opérations comptables est conservé sur 10 ans, conformément aux obligations légales.

---

## 📊 Bilan des Compétences SISR

```
Compétences du Référentiel BTS SIO — SISR

SISR1 — Gérer le patrimoine IT          ████████████████████  SP1 / SP3
SISR2 — Gérer les incidents             ████████████████████  SP2
SISR3 — Sécurité réseau                 ████████████████░░░░  SP2
SISR4 — Mode projet                     ████████████████░░░░  SP1 / SP4
SISR5 — Service utilisateurs            ████████████████████  SP3 / SP4

Légende : ░ = En cours de maîtrise  █ = Maîtrisé
```

| Domaine | Technologies maîtrisées |
|---|---|
| **Administration système** | Windows Server, Active Directory, GPO, DHCP, DNS |
| **Cloud & Identité** | Microsoft 365, Azure AD / Entra ID, Intune, Exchange Online |
| **Sécurité** | WithSecure Elements, VPN IPsec, MFA, NTFS, Analyse d'incident |
| **Scripting** | PowerShell (administration AD, inventaire, automatisation) |
| **ITSM** | GLPI (ticketing, CMDB, SLA, base de connaissance) |
| **Applications métier** | Netvie, Netfactu, Octime, CEGI |

---

## 📚 Veille Technologique

| Date | Sujet | Source |
|---|---|---|
| 2024-Q1 | Évolutions Microsoft Entra ID (remplace Azure AD) | Microsoft Learn |
| 2024-Q2 | Recommandations ANSSI — Sécurisation Active Directory | ANSSI.fr |
| 2024-Q3 | Ransomwares ciblant le secteur médical (Conti, LockBit) | CERT-FR |
| 2024-Q4 | Windows Server 2025 — Nouvelles fonctionnalités SMB | Microsoft Docs |

---

## 📞 Contact

> *Document rédigé dans le cadre du Portfolio E5 — BTS SIO SISR*  
> *Apprenti Technicien Systèmes, Réseaux & Support*  
> *Fondation Saint Jean de Dieu — France*

---

*© Portfolio BTS SIO SISR — Fondation Saint Jean de Dieu — Tous droits réservés*
