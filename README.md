# M.-A.-Priscille
# Déploiement d'une infrastructure AWS avec CloudFormation

le projet propose un modèle AWS CloudFormation permettant de déployer automatiquement un systeme simple pour traiter de fichiers stockés sur Amazon S3. L’infrastructure inclut un bucket S3, une fonction Lambda pour le traitement des événements, une table DynamoDB pour enregistrer les métadonnées des fichiers, une instance EC2 avec son groupe de sécurité, ainsi que les autorisations nécessaires pour que les composants interagissent entre eux.

1- Objectifs du syteme de traitement 

L'objectif de cette infrastructure est de déclencher automatiquement une fonction Lambda uen fois qu'un fichier est déposé dans le bucket S3. Cette fonction collecte alors les métadonnées du fichier et les enregistre dans une table DynamoDB. Une instance EC2 est également déployée dans un VPC existant, avec un groupe de sécurité personnalisé autorisant les connexions SSH depuis une IP spécifique.

2- Présentation des composants

    * Paramètres du modèle
Le modèle prend deux paramètres :
- EnvName : le nom de l’environnement utilisé pour personnaliser les noms des ressources.
- VPcId : l'identifiant du VPC dans lequel sera lancée l’instance EC2.

    * Bucket S3
Le bucket S3 est est configuré pour déclencher un événement à chaque ajout de fichier. Cet événement appelle une fonction Lambda, grâce à une autorisation spécifique définie par la ressource Lambda::Permission.

    * Fonction Lambda
La fonction Lambda est écrite en Python 3.9. Elle est déclenchée à chaque ajout de fichier dans le bucket S3. À chaque exécution, elle lit les métadonnées du fichier et les insère dans la table DynamoDB. Le nom de la table est passé à Lambda via une variable d’environnement. La fonction utilise les SDK boto3 pour interagir avec S3 et DynamoDB.

    * Table DynamoDB
La table DynamoDB est utilisée pour stocker les métadonnées des fichiers. Elle est nommée dynamiquement selon l’environnement. Elle utilise un seul attribut comme clé primaire : FileName de type String. Le débit est provisionné avec 5 unités de lecture et 5 unités d’écriture.

    * Instance EC2
Une instance EC2 de type t2.micro est également déployée dans un sous-réseau spécifique (subnet-0a230d78bf307974b) appartenant au VPC indiqué en paramètre. L’instance est configurée avec deux volumes EBS. Elle est associée à un groupe de sécurité personnalisé qui autorise uniquement les connexions SSH depuis une adresse IP bien précise (196.169.11.120/24). La clé SSH utilisée est iabdkey.

    * Groupe de sécurité
Un groupe de sécurité nommé selon l’environnement est créé pour contrôler les accès à l’instance EC2. Il autorise uniquement les connexions entrantes sur le port SSH depuis une plage IP spécifique.

    * Autorisation Lambda-S3
Pour permettre au bucket S3 d’appeler la fonction Lambda, une autorisation est définie par la ressource AWS::Lambda::Permission. Cette autorisation indique que le service S3 est autorisé à appeler la fonction lorsque des événements se produisent dans le bucket concerné.


3- Déploiement

Pour déployer cette infrastructure, vous devez disposer de l’AWS CLI configurée, d’un rôle IAM pour Lambda (lambda-s3-trigger-role) avec les bonnes permissions, d’un VPC actif, et d’une clé EC2. Une fois ces prérequis, vous pouvez déployer le modèle avec la commande suivante dans le terminale:

" aws cloudformation deploy --template-file template.yaml --stack-name devoir-iabd-priscillegogoyiia-stack --capabilities CAPABILITY_NAMED_IAM --no-fail-on-empty-changeset --parameter-overrides EnvName=priscillegogoyiia "


4- Fonctionnement global

Une fois le systeme déployé, voici le cycle d’utilisation :
1. Un fichier est déposé dans le bucket S3.
2. Cela déclenche un événement qui appelle la fonction Lambda.
3. La fonction Lambda lit les informations du fichier et les écrit dans DynamoDB.
4. Les données stockées peuvent ensuite être utilisées après.
