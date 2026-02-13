# Projet E5 SAB - Urbanisation des SI

Ce document présente la démarche et les étapes réalisées dans le cadre du projet d’urbanisation des systèmes d’information (SI).

## Ressources

- [TP : Projet E5 SAB - Urbanisation des SI (PDF)](docs/pdf/Projet%20E5%20SAB%20-%20Urbanisation%20des%20SI.pdf)

---

## Partie 1 : Mise en place de l’environnement

### Choix du framework Kubernetes

Le framework sélectionné pour répondre aux critères du MVP est Minikube.

```text
Minikube correspond à ce framework, il sera donc utilisé pour la suite du projet.
```

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

---

## Accès à l’application via CURL (interne et externe)

### Accès interne via le LoadBalancer

Le fichier de configuration du LoadBalancer est disponible ici : [prod-deployment.yml](docs/files/prod-deployment.yml)

```bash
curl -I 192.168.49.2:31491
```

Réponse :

```text
HTTP/1.1 200 OK
Server: gunicorn
Date: Fri, 13 Feb 2026 09:53:51 GMT
Connection: close
Content-Type: text/html; charset=utf-8
X-Frame-Options: DENY
Content-Length: 20823
Vary: Cookie
X-Content-Type-Options: nosniff
Referrer-Policy: same-origin
Cross-Origin-Opener-Policy: same-origin
```

### Accès externe via le LoadBalancer

```bash
minikube tunnel
```

Retour :

```text
Status:
        machine: minikube
        pid: 42043
        route: 10.96.0.0/12 -> 192.168.49.2
        minikube: Running
        services: [ecommerce-front-service]
    errors: 
                minikube: no errors
                router: no errors
                loadbalancer emulator: no errors
```

---

## Partie 2 : Déploiement multi-environnements

Pour permettre l’accès aux applications à plusieurs équipes, des environnements isolés doivent être créés pour :

- mlops
- preprod
- prod

L’infrastructure as code doit être déployée de façon identique dans ces trois environnements.

---

## Partie 3 (facultative) : Intégration d’une base de données

Une base de données autre que SQLite doit être déployée et connectée à l’application Stripe.

### Exemple de configuration MariaDB dans `prod-deployment.yml`

La documentation [MariaDB sur Kubernetes de IONOS](https://www.ionos.fr/digitalguide/hebergement/aspects-techniques/mariadb-kubernetes/) a été utilisée pour intégrer MariaDB au projet. Le fichier a été adapté pour répondre aux besoins spécifiques.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
spec:
  replicas: 1
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
          value: '@pr€ttyN1cePA$$W0RD!'
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
spec:
    ports:
    - port: 3306
      targetPort: 3306
    selector:
      app: mariadb
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
