# Acquisition forensic AWS

## Identification

Relever :

- Organization ID, Account ID ;
- région ;
- Instance ID ;
- ARN des ressources ;
- VPC, subnet, ENI ;
- security groups ;
- rôle IAM de l’instance ;
- volumes EBS et instance store ;
- Auto Scaling Group ;
- cluster EKS éventuel.

## Collecte

1. Exporter `DescribeInstances`, volumes, ENI et security groups.
2. Exporter CloudTrail, GuardDuty, Config, VPC Flow Logs et logs S3 pertinents.
3. Utiliser SSM si déjà déployé et approuvé.
4. Acquérir RAM et état volatile.
5. Copier immédiatement les données de l’instance store.
6. Créer un snapshot de chaque volume EBS.
7. Copier les snapshots vers le compte forensic.
8. Chiffrer avec une clé KMS forensic.
9. Appliquer tags `CaseId`, `Evidence`, `Source`, `AcquiredUTC`.
10. Isoler avec un security group précréé après collecte volatile.

## Commandes indicatives

```bash
aws ec2 describe-instances --instance-ids "$INSTANCE_ID" --region "$REGION"
aws ec2 describe-volumes \
  --filters Name=attachment.instance-id,Values="$INSTANCE_ID" \
  --region "$REGION"
aws ec2 create-snapshot \
  --volume-id "$VOLUME_ID" \
  --description "Forensic $CASE_ID"
aws cloudtrail lookup-events \
  --start-time "$START" --end-time "$END"
```

## Points de vigilance

- L’instance store n’est pas préservé par un snapshot EBS.
- Modifier un security group change l’état de la preuve : documenter l’action.
- Désactiver toute remédiation automatique susceptible d’arrêter ou remplacer l’instance.
- Préserver les rôles, politiques et clés KMS nécessaires à la lecture.
- Ne pas lancer les volumes originaux dans une instance d’analyse.
