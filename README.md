# Projet E5 SAB - Urbanisation des SI

Ce document présente la démarche et les étapes réalisées dans le cadre du projet d’urbanisation des systèmes d’information (SI).
## Groupe et Références

- **Membres du groupe :**
    - BILLON Anthony
    - BLAIZOT Éloïse
    - CADET Lucie
    - DUPRESSOIR Amaury
    - LALANDE Mathieu

- [Dépôt GitHub du projet](https://github.com/CringeTM/UrbanSI?tab=readme-ov-file)

- [TP : Projet E5 SAB - Urbanisation des SI (PDF)](docs/pdf/Projet%20E5%20SAB%20-%20Urbanisation%20des%20SI.pdf)

- Ressource fournié dans le devoir

## Partie 0 : Applications sélectionnées
- Application Rocket Flask Stripe
- MariaDB
- PHPMyAdmin

## Partie 1 : Mise en place de l’environnement

### Choix du framework Kubernetes

Le framework sélectionné pour répondre aux critères du MVP est Minikube.

**Minikube correspond à ce framework, il sera donc utilisé pour la suite du projet.**

### Lancement de Minikube

```bash
minikube start --listen-address=0.0.0.0 --memory=max --cpus=max --kubernetes-version=v1.35.0
```

![Résultat de la commande minikube](docs/images/lancer_minikube.png)

### Chargement de l’image Docker dans Minikube

L’image Docker n’était pas présente dans l’environnement Docker de Minikube, elle avait été construite uniquement dans le Docker local. Il a donc été nécessaire de charger l’image locale dans Minikube pour l’utiliser dans le cluster.

![Avant de charger l’image](docs/images/docker_image_1.png)

![Avant de charger l’image](docs/images/docker_image_2.png)

![Avant de charger l’image](docs/images/docker_image_3.png)

```bash
minikube image load rocket:local
```

![Après le chargement de l’image](docs/images/docker_image_4_rocket.png)

Après avoir fait fonctionner l’image en local, nous l’avons publiée sur Docker Hub afin que les images des applications soient accessibles depuis un repository Docker Hub.

---
### Publication de l’image sur Docker Hub

Pour permettre le déploiement sur différents environnements et garantir l’accessibilité de l’image, celle-ci a été publiée sur un repository Docker Hub public : [warpprod/rocket-ecommerce](https://hub.docker.com/r/warpprod/rocket-ecommerce).

L’image à été envoyée dans DockerHub via la commande suivante :

```bash
docker build -t warpprod/rocket-ecommerce . && docker push warpprod/rocket-ecommerce
```

```text
[+] Building 14.4s (8/16)                                                                                                                                          docker:default
 => [internal] load build definition from Dockerfile                                                                                                                         0.0s
 => => transferring dockerfile: 1.09kB                                                                                                                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.11.5                                                                                                             0.7s
 => [auth] library/python:pull token for registry-1.docker.io                                                                                                                0.0s
 => [internal] load .dockerignore                                                                                                                                            0.0s
 => => transferring context: 2B                                                                                                                                              0.0s
 => CACHED [ 1/11] FROM docker.io/library/python:3.11.5@sha256:2e376990a11f1c1e03796d08db0e99c36eadb4bb6491372b227f1e53c3482914                                              0.0s
 => => resolve docker.io/library/python:3.11.5@sha256:2e376990a11f1c1e03796d08db0e99c36eadb4bb6491372b227f1e53c3482914                                                       0.0s
 => [internal] load build context                                                                                                                                            0.0s
 => => transferring context: 24.75kB                                                                                                                                         0.0s
 => [ 2/11] COPY requirements.txt .                                                                                                                                          0.3s
 => [ 3/11] RUN pip install --upgrade pip                                                                                                                                    5.2s
 => [ 4/11] RUN pip install --no-cache-dir -r requirements.txt                                                                                                               7.8s 
 => => #   Downloading mysqlclient-2.2.0.tar.gz (89 kB)                                                                                                                           
 => => #   Installing build dependencies: started                                                                                                                                 
 => => #   Installing build dependencies: finished with status 'done'                                                                                                             
 => => #   Getting requirements to build wheel: started                                                                                                                           
 => => #   Getting requirements to build wheel: finished with status 'done'                                                                                                       
 => => #   Preparing metadata (pyproject.toml): started   

 ... etc
```

Après avoir construit l’image, vous pouvez vérifier qu’elle est bien disponible avec la commande suivante :

```bash
docker images | grep warpprod/rocket-ecommerce
```

On spécifie donc l’image dans nos manifestes Kubernetes comme ceci :

```bash
image: warpprod/rocket-ecommerce:latest
```

Cela assure que toutes les équipes et environnements peuvent accéder à la même version de l’application.

### Configuration de l'application pour MariaDB

L'application n'avait pas le connecteur de MariaDB présent dans les requirements de l'aplication, nous avons dû l'ajouter

1. **Installer le connecteur** :  
    Modifier le fichier `requirements.txt` pour ajouter la dépendance suivante :  

    ```python
    PyMySQL==1.1.0
    ```

2. **Migration de la database** :
    Nous nous sommes connecté à l'un des conteurs via les commandes suivantes :
    ```bash
    minikube ssh
    docker ps
    docker exec -ti container_name /bin/bash
    ```
    Nous avons ensuite suivi la documentation de l'application ecommerce et tapé les commandes suivantes :
    ```bash
    python manage.py makemigrations
    python manage.py migrate
    ```
    ![Initialisation de la base de donnée](docs/images/initiation_bdd.png)
   

Nous avons ensuite ajouté un produit **Mug** dans Stripe, qui apparaît bien dans l’interface du site e-commerce :

| Produit ajouté dans Stripe | Affichage du produit sur le site |
|---------------------------|----------------------------------|
| ![Mug dans Stripe](docs/images/mug_stripe.png) | ![Mug sur le site](docs/images/mug_stripe_site.png) |

Pour valider le parcours d’achat, une transaction a été réalisée depuis le site pour acheter ce **Mug**. La réussite de la transaction est visible à la fois dans Stripe et sur le site e-commerce :

| Transaction réussie dans Stripe | Confirmation sur le site e-commerce |
|---------------------------------|-------------------------------------|
| ![Transaction Stripe](docs/images/mug_stripe_transaction.png) | ![Page d'achat](docs/images/mug_stripe_site_transaction_page.png) <br> ![Achat réussi](docs/images/mug_stripe_site_transaction.png) |

Enfin, les informations de la commande et du produit acheté sont bien enregistrées dans la base de données, consultables via phpMyAdmin :

| Commandes dans phpMyAdmin | Produit acheté |
|--------------------------|----------------|
| ![Commandes](docs/images/phpmyadmin_order.png) | ![Produit](docs/images/phpmyadmin_product.png) |

### Vérification du nombre de réplicas

Pour garantir la haute disponibilité de l’application, le nombre de réplicas a été défini à 3 dans le manifeste Kubernetes :

```yaml
spec:
    replicas: 3
```

Cela permet d’assurer que trois pods de l’application sont toujours déployés et disponibles dans le cluster.

## Partie 2 : Déploiement multi-environnements

Pour permettre l’accès aux applications à plusieurs équipes, des environnements isolés doivent être créés pour :

- mlops
- preprod
- prod

L’infrastructure as code doit être déployée de façon identique dans ces trois environnements.

### Déploiement multi-environnements

Trois fichiers de déploiement Kubernetes ont été créés pour isoler les environnements :

- [`prod-deployment.yaml`](files/prod-deployment.yaml) : environnement de production
- [`preprod-deployment.yaml`](files/preprod-deployment.yaml) : environnement de préproduction
- [`mlops-deployment.yaml`](files/mlops-deployment.yaml) : environnement MLOps

Chaque fichier contient la même structure d’infrastructure (namespace, déploiements, services, volumes, etc.), adaptée à l’environnement cible.  
Cela permet à chaque équipe de travailler dans un environnement isolé, tout en garantissant la cohérence des ressources déployées.

#### Commande pour appliquer les environnements

```bash
k apply -f prod-deployment.yaml
k apply -f preprod-deployment.yaml
k apply -f mlops-deployment.yaml
```

Commande pour appliquer les modifications :

- ![Apply](docs/images/apply-f.png)

Vérification de l'état des déploiements : 

- ![Commandes de vérification](docs/images/get_all_infos.png)

Chaque environnement dispose ainsi de ses propres ressources, assurant l’isolation et la reproductibilité des déploiements.

---

## Partie 3 : Intégration d’une base de données

Une base de données autre que SQLite doit être déployée et connectée à l’application Stripe.

### Exemple de configuration MariaDB dans `prod-deployment.yaml`

La documentation [MariaDB sur Kubernetes de IONOS](https://www.ionos.fr/digitalguide/hebergement/aspects-techniques/mariadb-kubernetes/) a été utilisée pour intégrer MariaDB au projet. Le fichier a été adapté pour répondre aux besoins spécifiques.

Nous avons ajouter les namespaces "prod" sur notre fichier pour faire fonctionner MariaDB et PhpMyAdmin (qui permettra à un adaministrateur de maintenir la base de données du site).

Pour voir les infos de mariadb, vous pouvez lire `prod-deployment.yaml`.

```yaml
# Principales informations de mariadb
apiVersion: v1
kind: PersistentVolume
metadata:
    name: mariadb-pv
spec:
    capacity:
        storage: 10Gi
    accessModes:
        - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    hostPath:
        path: /mnt/data/mariadb
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mariadb-pvc
    namespace: prod
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
            storage: 10Gi
---
apiVersion: v1
kind: ConfigMap
metadata:
    name: mariadb-config
    namespace: prod
    labels:
        app: mariadb
data:
    my.cnf: |
        [mysqld]
        bind-address=0.0.0.0
        default_storage_engine=InnoDB
        innodb_file_per_table=1
        max_connections=1000
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
  namespace: prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mariadb
  serviceName: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - env:
        - name: MYSQL_ROOT_PASSWORD
          value: "SuperSecurePass123"
        - name: MYSQL_DATABASE
          value: "ecommerce"
        image: mariadb:latest
        name: mariadb
        ports:
        - containerPort: 3306
          name: mariadb
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mariadb-storage
        - mountPath: /etc/mysql/conf.d
          name: config-volume
      volumes:
      - configMap:
          name: mariadb-config
        name: config-volume
  volumeClaimTemplates:
  - metadata:
      name: mariadb-storage
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
    name: mariadb
    namespace: prod
spec:
    ports:
    - port: 3306
      targetPort: 3306
    selector:
      app: mariadb
```

```yaml
# Partie pour phpmyadmin
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin
  namespace: prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
      - name: phpmyadmin
        image: phpmyadmin/phpmyadmin:latest
        env:
        - name: PMA_HOST
          value: mariadb
        - name: PMA_PORT
          value: "3306"
        - name: PMA_ARBITRARY
          value: "1"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
  namespace: prod
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    nodePort: 30080
  selector:
    app: phpmyadmin
```

```bash
# Afficher toutes les ressources principales Kubernetes dans le namespace courant
k get all
```

Retour :

```text
NAME                       READY   AGE
statefulset.apps/mariadb   1/1     22s
```

---

## Accès aux services depuis l'extérieur
Nous avions rencontré des difficultés d'usage avec mikicube port forward qui était extrêmement lent, nous l'avons donc remplacé par socat qui nous a permis de publier nos NodePorts sur l'IP Publique de notre VM avec de bien meilleures performances.
```bash
# Commande pour le site e-commerce de Prod
sudo socat TCP-LISTEN:5005,fork,reuseaddr TCP:$(minikube ip):30005
```
```bash
# Commande pour PhpMyAdmin de Prod
sudo socat TCP-LISTEN:8080,fork,reuseaddr TCP:$(minikube ip):30080
```

---

## Conclusion

Ce projet nous a permis de mettre en pratique les principes fondamentaux de Kubernetes à travers le déploiement d'une application e-commerce et d'une stack complète. Nous avons réussi à containeriser nos services (Flask, MariaDB et PHPMyAdmin), publier une "application métier" sur Docker Hub et les orchestrer via Minikube.