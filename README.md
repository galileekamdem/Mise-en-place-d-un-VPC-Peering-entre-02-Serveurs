# Mise-en-place-d-un-VPC-Peering-entre-02-Serveurs
Mise en place d'un VPC-Peering entre 02 Serveurs dans 02 zones de disponibilités

VPC Peering entre 2 VPC AWS
Connexion sécurisée entre environnements réseau isolés
Mise en place d'une communication privée entre deux VPCs dans la même région AWS

📌 Objectifs du projet
Établir une connexion privée entre deux VPCs distincts

Permettre la communication inter-VPC sans passer par Internet

Maintenir l'isolation réseau tout en autorisant des échanges contrôlés

Configurer des routes spécifiques pour le trafic peering

🔧 Stack technique
Composant	Description
VPCA	VPC principal (12.0.0.0/16)
VPCB	VPC secondaire (13.0.0.0/16)
VPC Peering	Connexion directe entre VPCA et VPCB
EC2	Instances Ubuntu avec Apache
Route Tables	Tables de routage personnalisées
🏗️ Architecture

Diagram(in file)
Détails clés :

Plages IP non chevauchantes : 12.0.0.0/16 et 13.0.0.0/16

Sous-réseaux publics dans chaque VPC

Tables de routage mises à jour pour le trafic peering

Sécurité : Communication directe sans exposition publique

🛠️ Implémentation étape par étape

1. Création des VPCs
bash
# VPCA
aws ec2 create-vpc --cidr-block 12.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=VPCA}]'

# VPCB
aws ec2 create-vpc --cidr-block 13.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=VPCB}]'

2. Configuration des sous-réseaux
VPC	Sous-réseau	CIDR	AZ
VPCA	VPCA-subnet	12.0.0.0/24	us-east-1a
VPCB	VPCB-subnet	13.0.1.0/24	us-east-1a

3. Mise en place du Peering
bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-12345 \ # VPCA
  --peer-vpc-id vpc-67890 \ # VPCB
  --peer-region us-east-1 \
  --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=My-PC}]'

4. Configuration des routes
Table de routage VPCA :

Destination: 13.0.0.0/16 → Target: peering-connection-id

Table de routage VPCB :

Destination: 12.0.0.0/16 → Target: peering-connection-id

5. Déploiement des instances EC2
VPCA : Instance Ubuntu avec Apache (User Data)

VPCB : Instance Ubuntu avec Apache (User Data)

🔐 Bonnes pratiques implémentées
Séparation claire des plages IP

Contrôle granulaire via les tables de routage

Nommage cohérent des ressources

Documentation complète des flux autorisés

🧪 Validation du peering
Se connecter à l'instance VPCA via SSH

Tester la connectivité vers VPCB :

bash
ping 13.0.1.10 # IP privée de l'instance VPCB
curl http://13.0.1.10 # Test du serveur Apache
📚 Ressources
Guide complet du lab

Documentation AWS VPC Peering

🚀 Améliorations possibles
Ajouter un schéma détaillé des flux réseau

Automatiser avec Terraform/CloudFormation

Implémenter des règles de sécurité supplémentaires

Configurer le monitoring avec VPC Flow Logs

Exemple de script de test (scripts/test-connectivity.sh) :

bash
#!/bin/bash
# Test de connectivité entre instances
INSTANCE_A_PRIVATE_IP="12.0.0.10"
INSTANCE_B_PRIVATE_IP="13.0.1.10"

echo "Testing connectivity from VPCA to VPCB..."
ssh -i VPCAKey.pem ubuntu@$INSTANCE_A_PRIVATE_IP "ping -c 4 $INSTANCE_B_PRIVATE_IP"

echo "Testing web server access..."
curl "http://$INSTANCE_B_PRIVATE_IP"
Documentation des décisions techniques (docs/technical-decisions.md) :

markdown
## Choix des plages IP
- Sélection de 12.0.0.0/16 et 13.0.0.0/16 pour :
  - Éviter tout chevauchement
  - Permettre une croissance future
  - Respecter les bonnes pratiques AWS

## Pourquoi le peering plutôt qu'un VPN?
- Latence réduite (communication interne AWS)
- Coût moindre (pas de frais de transfert de données)
- Simplicité de configuration
