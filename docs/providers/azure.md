# Acquisition forensic Microsoft Azure

## Identification

Relever :

- Tenant ID ;
- Subscription ID ;
- Resource Group ;
- VM Resource ID ;
- région ;
- Managed Disks ;
- NIC, subnet et NSG ;
- identité managée ;
- extensions ;
- Defender for Cloud ;
- Log Analytics Workspace.

## Collecte

1. Exporter la définition complète de la VM.
2. Exporter Azure Activity Log, Entra ID sign-in/audit logs et Defender.
3. Utiliser Run Command ou Azure Arc seulement si le canal est déjà approuvé.
4. Acquérir RAM et état volatile.
5. Copier le temporary disk avant arrêt si nécessaire.
6. Créer un snapshot de chaque Managed Disk.
7. Copier vers une subscription forensic.
8. Stocker les exports dans un container Blob immuable.
9. Protéger les preuves avec une clé dédiée.
10. Isoler avec un NSG forensic après acquisition volatile.

## Commandes indicatives

```bash
az vm show -g "$RG" -n "$VM" -o json
az disk list -g "$RG" -o json
az snapshot create \
  -g "$FORENSIC_RG" \
  -n "$SNAPSHOT_NAME" \
  --source "$DISK_ID"
az monitor activity-log list \
  --resource-group "$RG" \
  --offset 7d -o json
```

## Particularités RH

- présence obligatoire du juridique/DPO et, selon le cas, de RH ;
- minimisation des données collectées ;
- accès limité aux analystes habilités ;
- pseudonymisation dans les rapports ;
- interdiction de diffuser des données personnelles dans les tickets généraux ;
- durée de rétention explicitement validée.

## Points de vigilance

- Le temporary disk est éphémère.
- Préserver les journaux Entra ID et les identités managées.
- L’isolement ne doit pas couper le seul canal d’acquisition.
- Documenter les changements de NSG dans la chaîne de custody.
