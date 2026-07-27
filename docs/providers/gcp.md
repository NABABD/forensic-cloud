# Acquisition forensic Google Cloud Platform

## Identification

Relever :

- Organization ID ;
- Folder et Project ID ;
- zone et région ;
- nom et ID de l’instance ;
- service account ;
- Persistent Disks ;
- Local SSD ;
- VPC, subnet, tags et firewall rules ;
- instance groups ;
- GKE éventuel.

## Collecte

1. Exporter la description de l’instance et des disques.
2. Exporter Cloud Audit Logs, VPC Flow Logs, DNS, Security Command Center.
3. Utiliser IAP, SSH ou OS Config seulement si le canal est approuvé.
4. Acquérir RAM et état volatile.
5. Copier les Local SSD avant arrêt.
6. Créer un snapshot de chaque Persistent Disk.
7. Copier ou créer les snapshots dans le projet forensic autorisé.
8. Appliquer labels et politique de rétention.
9. Isoler par tag/règle firewall après acquisition volatile.
10. Créer une copie de travail séparée.

## Commandes indicatives

```bash
gcloud compute instances describe "$VM" \
  --zone "$ZONE" --project "$PROJECT" --format=json

gcloud compute disks snapshot "$DISK" \
  --snapshot-names "$SNAPSHOT" \
  --zone "$ZONE" \
  --project "$PROJECT"

gcloud logging read \
  'resource.type="gce_instance"' \
  --project "$PROJECT" \
  --freshness=7d \
  --format=json
```

## Séparation clients

Pour chaque client :

- projet forensic distinct si possible ;
- bucket distinct ;
- clé KMS distincte ;
- identité d’acquisition distincte ;
- rétention contractuelle propre ;
- rapport et chaîne de custody propres ;
- aucune mutualisation des artefacts entre clients.

## Points de vigilance

- Le Local SSD est éphémère.
- Vérifier les VPC Service Controls.
- Vérifier les CMEK et droits de déchiffrement.
- Préserver les changements IAM et activités de service accounts.
