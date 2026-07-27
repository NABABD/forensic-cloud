# Playbook d’acquisition forensic — AWS, Azure, GCP et cloud privé

**Version :** 1.0
**Statut :** Projet pédagogique / à adapter avant production
**Propriétaire :** Équipe CSIRT / DFIR
**Classification :** Interne sensible

## 1. Objectif

Ce playbook standardise la première réponse et l’acquisition de preuves
numériques dans quatre environnements :

- AWS ;
- Microsoft Azure ;
- Google Cloud Platform ;
- cloud privé, principalement OpenStack, VMware vSphere ou Hyper-V.

Il vise à garantir une collecte :

- techniquement exploitable ;
- reproductible ;
- documentée ;
- intègre ;
- conforme aux exigences juridiques et contractuelles ;
- respectueuse de l’ordre de volatilité.

Ce document ne remplace ni une autorisation légale, ni les procédures internes,
ni les contrats conclus avec les fournisseurs cloud.

## 2. Périmètre

Le playbook couvre :

- machines virtuelles Linux et Windows ;
- disques système et données ;
- mémoire vive ;
- journaux du système invité ;
- journaux du plan de contrôle cloud ;
- métadonnées de ressources ;
- configuration IAM et réseau ;
- stockage objet ;
- conteneurs et orchestrateurs, en collecte initiale ;
- chaîne de conservation des preuves.

Hors périmètre sans annexe dédiée :

- acquisition physique dans les datacenters du fournisseur ;
- équipements mobiles ;
- SaaS tiers ;
- bases de données managées complexes ;
- déchiffrement ou contournement d’un mécanisme de sécurité ;
- suppression ou remédiation définitive.

## 3. Rôles et responsabilités

| Rôle | Responsabilités |
|---|---|
| Incident Commander | Autorise les étapes, arbitre continuité d’activité et confinement |
| Responsable DFIR | Définit la stratégie d’acquisition et valide les outils |
| Opérateur d’acquisition | Exécute les commandes et remplit la chaîne de custody |
| Responsable cloud | Fournit les accès, identifiants et informations d’architecture |
| Juridique / DPO | Valide base légale, données personnelles, transferts et rétention |
| RH | Obligatoire lorsque l’incident concerne un salarié ou des données RH |
| RSSI / CISO | Valide les mesures de confinement et les communications |
| Fournisseur cloud | Support contractuel, conservation de logs, escalade plateforme |
| Client | Pour GCP clients : autorise le périmètre et reçoit les livrables |

## 4. Conditions de déclenchement

Le playbook peut être déclenché après :

- alerte EDR ou SIEM critique ;
- compromission présumée d’un compte privilégié ;
- exfiltration de données ;
- exécution de malware ;
- activité réseau anormale ;
- modification non autorisée d’une ressource cloud ;
- demande du juridique, du DPO, de RH ou d’un client ;
- compromission d’une VM, d’un nœud Kubernetes ou d’un hyperviseur.

## 5. Principes fondamentaux

1. Ne jamais travailler directement sur l’unique copie de preuve.
2. Noter toutes les heures en UTC.
3. Acquérir les éléments les plus volatils en premier.
4. Éviter l’arrêt et le redémarrage avant la capture de la RAM.
5. Ne pas installer un outil à chaud sans validation.
6. Hacher chaque artefact immédiatement après acquisition.
7. Stocker les preuves dans un espace distinct, chiffré et immuable.
8. Utiliser une identité forensic temporaire et dédiée.
9. Séparer l’acquisition, l’analyse et la remédiation.
10. Documenter toute déviation.

## 6. Ordre de volatilité

L’ordre suivant doit être adapté au contexte, mais toute modification doit être
justifiée.

| Priorité | Élément | Exemples |
|---:|---|---|
| 1 | État temps réel | heure, écran, console, sessions actives |
| 2 | Mémoire vive | RAM, clés en mémoire, injections, processus |
| 3 | État système volatile | processus, handles, services, modules, tâches |
| 4 | Réseau volatile | connexions, sockets, ARP/neighbor, routes, DNS cache |
| 5 | Conteneurs | pods, namespaces, processus, volumes éphémères |
| 6 | Journaux locaux non centralisés | journal, Event Logs, auditd, shell history |
| 7 | Stockage éphémère | instance store, temp disk, local SSD |
| 8 | Disques persistants | EBS, Managed Disks, Persistent Disk, Cinder/VMDK |
| 9 | Plan de contrôle | audit logs, IAM, API calls, activité administrative |
| 10 | Sauvegardes et archives | snapshots, sauvegardes, objets versionnés |

> Un snapshot de disque ne capture pas la RAM. La mémoire doit être acquise
> avant un arrêt ou un redémarrage.

## 7. Préparation avant incident

### 7.1 Comptes et accès

Préparer :

- un compte ou projet forensic séparé ;
- des rôles IAM temporaires ;
- une authentification multifacteur ;
- un coffre de secrets ;
- un stockage immuable ;
- une clé de chiffrement dédiée ;
- une station d’acquisition durcie ;
- des outils signés et hachés.

### 7.2 Outils validés

Exemples à faire homologuer :

- Linux : AVML, LiME, `ss`, `lsof`, `journalctl`, `auditctl` ;
- Windows : WinPmem, KAPE, Sysinternals, `wevtutil`, `netstat` ;
- réseau : `tcpdump`, `pktmon`, VPC Flow Logs, NSG Flow Logs ;
- disque : snapshots natifs, `dd`, E01 lorsque nécessaire ;
- analyse : Volatility, Plaso, Timesketch, Autopsy.

### 7.3 Stockage de preuve

Le stockage doit avoir :

- chiffrement côté serveur ;
- versioning ;
- rétention ou verrouillage ;
- journalisation des accès ;
- refus de suppression pour l’opérateur ;
- séparation par dossier d’incident ;
- contrôle d’accès nominatif.

Arborescence recommandée :

```text
CASE-YYYY-NNN/
├── 00-administration/
├── 01-control-plane/
├── 02-volatile/
├── 03-memory/
├── 04-disks/
├── 05-logs/
├── 06-network/
├── 07-analysis-copies/
├── 08-reports/
└── manifests/
```

## 8. Procédure générale

### Phase A — Autorisation

- créer le numéro d’incident ;
- identifier le demandeur ;
- confirmer le propriétaire des données ;
- confirmer les pays de stockage et de traitement ;
- obtenir l’accord de l’Incident Commander ;
- obtenir l’accord juridique/DPO si données personnelles ;
- obtenir l’accord RH si salarié concerné ;
- confirmer les obligations de notification ;
- ouvrir un canal de communication restreint.

**Point d’arrêt :** aucune collecte sans autorisation documentée, sauf procédure
d’urgence préautorisée.

### Phase B — Stabilisation documentaire

Noter immédiatement :

- date et heure UTC ;
- identité de l’opérateur ;
- source de l’alerte ;
- identifiants exacts de la ressource ;
- compte, subscription, projet ou tenant ;
- région et zone ;
- adresse IP ;
- OS présumé ;
- criticité métier ;
- état de la ressource ;
- actions déjà réalisées.

### Phase C — Capture du plan de contrôle

Avant toute modification, exporter :

- description complète de la VM ;
- tags et labels ;
- disques attachés ;
- interfaces réseau ;
- groupes de sécurité, NSG ou firewall rules ;
- rôles et identités attachés ;
- historique d’activité ;
- alertes sécurité ;
- configuration de journalisation ;
- clés de chiffrement utilisées ;
- politiques de rétention ;
- état de l’horloge.

### Phase D — Acquisition volatile

Lorsque le risque et l’autorisation le permettent :

1. capturer l’heure UTC de la cible ;
2. capturer la RAM ;
3. calculer le hachage du dump ;
4. copier le dump vers le coffre ;
5. collecter processus, services et sessions ;
6. collecter connexions, routes et cache réseau ;
7. collecter modules/pilotes ;
8. collecter conteneurs et namespaces ;
9. collecter journaux locaux récents ;
10. enregistrer chaque commande.

Exemples Linux :

```bash
date -u
ps auxwwf
ss -plantue
ip address
ip route
ip neigh
lsof -nP
mount
lsmod
who -a
last -F
journalctl --no-pager --since "-24 hours"
```

Exemples Windows PowerShell :

```powershell
Get-Date -AsUTC
Get-Process
Get-Service
Get-NetTCPConnection
Get-NetRoute
Get-NetNeighbor
Get-CimInstance Win32_LoggedOnUser
Get-WinEvent -LogName System -MaxEvents 5000
Get-WinEvent -LogName Security -MaxEvents 5000
```

### Phase E — Stockage éphémère

Les volumes temporaires peuvent disparaître lors d’un arrêt.

- AWS : instance store ;
- Azure : temporary disk ;
- GCP : Local SSD ;
- privé : disque local hyperviseur ou ephemeral disk OpenStack.

Les copier avant arrêt lorsque cela est juridiquement et techniquement possible.

### Phase F — Acquisition disque

- identifier tous les volumes ;
- créer un snapshot natif ;
- noter l’identifiant du snapshot ;
- copier vers le compte forensic si possible ;
- utiliser une clé forensic ;
- appliquer tags, rétention et legal hold ;
- créer une copie de travail ;
- monter uniquement la copie de travail en lecture seule ;
- ne jamais démarrer l’image originale comme une VM normale.

### Phase G — Journaux

Collecter au minimum :

- logs d’audit du plan de contrôle ;
- journaux IAM et authentification ;
- logs réseau ;
- DNS ;
- logs stockage objet ;
- logs du système invité ;
- EDR ;
- pare-feu et proxy ;
- Kubernetes audit logs ;
- logs applicatifs pertinents.

Exporter les données brutes avant normalisation.

### Phase H — Confinement

Le confinement intervient après les acquisitions volatiles, sauf danger
immédiat.

Options :

- remplacer les règles réseau par un groupe d’isolement ;
- conserver uniquement le canal forensic ;
- révoquer les sessions ou credentials compromis ;
- bloquer l’egress ;
- désactiver l’auto-scaling ;
- empêcher la suppression ;
- empêcher les actions automatiques de remédiation.

Toute mesure doit être approuvée et journalisée.

### Phase I — Scellement

Pour chaque artefact :

```bash
sha256sum artefact > artefact.sha256
sha512sum artefact > artefact.sha512
```

Enregistrer :

- nom logique ;
- chemin ;
- taille ;
- hachage ;
- heure d’acquisition ;
- outil et version ;
- opérateur ;
- ressource source ;
- stockage destination ;
- événements de transfert.

### Phase J — Validation

- vérifier les hachages après transfert ;
- vérifier la lecture de la preuve ;
- vérifier la rétention ;
- vérifier les journaux d’accès ;
- produire une copie d’analyse ;
- faire relire la fiche de custody ;
- fermer les accès temporaires.

## 9. Matrice synthétique par fournisseur

| Fonction | AWS | Azure | GCP | Cloud privé |
|---|---|---|---|---|
| VM | EC2 | Virtual Machines | Compute Engine | Nova / vSphere / Hyper-V |
| Disque | EBS Snapshot | Managed Disk Snapshot | Persistent Disk Snapshot | Cinder snapshot / VMDK |
| Audit | CloudTrail | Activity Log / Entra logs | Cloud Audit Logs | Keystone/Nova/vCenter logs |
| Réseau | VPC Flow Logs | NSG Flow Logs | VPC Flow Logs | Neutron / firewall / vSwitch |
| Canal distant | SSM | Run Command / Arc | OS Config / IAP / SSH | Bastion / console / agent |
| Isolation | Security Group | NSG | Firewall tag/rule | Security group / port group |
| Coffre | S3 Object Lock | Immutable Blob | Bucket retention lock | S3 privé / WORM / coffre |
| Clés | KMS | Key Vault | Cloud KMS | HSM / Vault / KMS interne |

## 10. Procédures par fournisseur

Les procédures détaillées sont disponibles dans :

- [AWS](docs/providers/aws.md)
- [Azure](docs/providers/azure.md)
- [GCP](docs/providers/gcp.md)
- [Cloud privé](docs/providers/cloud-prive.md)

## 11. Contacts légaux et contractuels

Avant incident, tenir à jour le registre suivant :

| Domaine | Contact primaire | Contact secondaire | Référence |
|---|---|---|---|
| Incident Commander | À compléter | À compléter | Astreinte CSIRT |
| Juridique | À compléter | À compléter | Procédure contentieux |
| DPO | À compléter | À compléter | Registre RGPD |
| RH | À compléter | À compléter | Procédure interne |
| Assurance cyber | À compléter | À compléter | Police / sinistre |
| AWS Support | À compléter | À compléter | Account ID / plan |
| Microsoft Support | À compléter | À compléter | Tenant / contrat |
| Google Cloud Support | À compléter | À compléter | Organization ID |
| Cloud privé | À compléter | À compléter | Exploitant / hébergeur |
| Client concerné | À compléter | À compléter | Contrat / SLA |
| Autorité compétente | À compléter | À compléter | Selon juridiction |

Questions obligatoires :

- Qui est propriétaire des données ?
- Existe-t-il une obligation de secret ou de confidentialité renforcée ?
- Les données peuvent-elles sortir du compte ou du pays ?
- Quelle est la durée de rétention autorisée ?
- Une notification au client, au DPO, à une autorité ou à l’assureur est-elle
  requise ?
- Le contrat permet-il une acquisition par snapshot ?
- Le fournisseur doit-il préserver des journaux supplémentaires ?
- Une demande de legal hold doit-elle être envoyée ?

## 12. Critères de réussite

Une acquisition est considérée complète lorsque :

- les autorisations sont jointes au dossier ;
- l’ordre de volatilité est respecté ou la déviation justifiée ;
- les quatre environnements disposent d’une procédure ;
- les éléments volatils pertinents ont été collectés ;
- tous les disques ont été identifiés ;
- les journaux du plan de contrôle sont exportés ;
- les artefacts sont hachés ;
- la chaîne de custody est complète ;
- les preuves sont chiffrées et verrouillées ;
- une copie de travail distincte est créée ;
- les accès temporaires sont révoqués ;
- le rapport final liste les limites.

## 13. Automatisation Ansible

Le dossier `ansible/` fournit une structure d’orchestration. L’automatisation ne
doit jamais remplacer la décision humaine pour :

- acquisition mémoire ;
- arrêt d’une VM ;
- confinement réseau ;
- révocation d’identités ;
- modification de règles de rétention.

Exécution de validation :

```bash
ansible-playbook ansible/playbooks/forensic.yml \
  -i ansible/inventory/localhost.yml \
  -e @ansible/vars/example-case.yml \
  --check
```

## 14. Limites et précautions

- L’acquisition live modifie nécessairement une partie du système.
- Un snapshot est une copie logique à un instant donné, pas automatiquement une
  image forensique bit à bit.
- Les disques chiffrés nécessitent de préserver les accès aux clés sans exporter
  de secrets inutilement.
- Les journaux peuvent être absents si leur collecte n’était pas configurée.
- Un attaquant disposant de privilèges cloud peut altérer les preuves avant le
  déclenchement.
- Une preuve copiée entre régions ou pays peut soulever un enjeu juridique.
- Les services managés exigent des procédures spécifiques.

## 15. Gestion de version

Convention :

- `MAJOR` : changement de procédure ou de responsabilité ;
- `MINOR` : ajout de fournisseur ou d’artefact ;
- `PATCH` : correction documentaire.

Toute modification doit être revue par le responsable DFIR et le juridique
lorsque le périmètre légal change.
