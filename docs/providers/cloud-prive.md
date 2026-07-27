# Acquisition forensic cloud privé

## Technologies couvertes

- OpenStack ;
- VMware vSphere / ESXi ;
- Microsoft Hyper-V ;
- stockage SAN, NAS ou objet interne.

## Identification

Relever :

- cluster et hyperviseur ;
- identifiant de la VM ;
- datastore ;
- volumes ;
- réseaux virtuels ;
- snapshots existants ;
- sauvegardes ;
- comptes administrateurs ;
- journaux de management ;
- horodatage des hôtes.

## OpenStack

Collecter :

- Keystone audit ;
- Nova API et compute logs ;
- Neutron logs et security groups ;
- Cinder volumes et snapshots ;
- Glance images ;
- logs du load balancer ;
- Ceph si utilisé.

Commandes indicatives :

```bash
openstack server show "$SERVER_ID" -f json
openstack server volume list "$SERVER_ID" -f json
openstack volume snapshot create \
  --force \
  --property CaseId="$CASE_ID" \
  "$VOLUME_ID"
openstack security group list -f json
```

## VMware vSphere

Collecter :

- événements vCenter ;
- tâches ;
- configuration VMX ;
- VMDK et descripteurs ;
- snapshots ;
- logs ESXi ;
- configuration vSwitch/dvSwitch ;
- sessions administratives.

Précautions :

- éviter de consolider ou supprimer des snapshots ;
- préserver la chaîne des VMDK ;
- utiliser une copie de datastore ;
- documenter tout snapshot suspendu incluant éventuellement la mémoire.

## Hyper-V

Collecter :

- configuration de la VM ;
- VHD/VHDX ;
- checkpoints ;
- journaux Hyper-V ;
- configuration des vSwitch ;
- état de réplication ;
- Event Logs de l’hôte.

## Sécurité de l’hyperviseur

Si l’hyperviseur est suspect :

- ne pas lui faire confiance pour calculer seul les hachages ;
- collecter depuis une console d’administration indépendante ;
- préserver les logs du stockage et du réseau ;
- envisager l’acquisition des hôtes physiques ;
- contacter l’exploitant et le fournisseur matériel.

## Isolation

- OpenStack : security group forensic ;
- VMware : port group de quarantaine ;
- Hyper-V : vSwitch de quarantaine ;
- conserver un canal d’administration forensic distinct.
