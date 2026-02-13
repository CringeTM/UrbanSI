# Projet E5 SAB - Urbanisation des SI

Projet E5 SAB - Urbanisation des SI - Documentation du projet

## Ressources

- [TP : Projet E5 SAB - Urbanisation des SI (PDF)](docs/pdf/Projet%20E5%20SAB%20-%20Urbanisation%20des%20SI.pdf)


## Les premières étapes avant de tout commencer

### Choisr un framwork basé sur kubernetes correspondant aux critères du mvp

```text
Minikube correspond à ce framwork, nous allons donc l'utiliser
```

### Lancer minikube

```bash
# On lance minikube
minikube start --listen-address=0.0.0.0 --memory=max --cpus=max --kubernetes-version=v1.35.0
```

![Résultat de la commande minikube](docs/images/lancer_minikube.png)

