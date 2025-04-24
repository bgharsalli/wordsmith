# Wordsmith App
Wordsmith est le projet de démonstration présenté à l'origine à la DockerCon EU 2017 et 2018.

L'application de démonstration fonctionne dans trois conteneurs :

api - une API Java REST qui sert les mots lus dans la base de données
web - une application web Go qui appelle l'API et construit des mots en phrases
db - une base de données Postgres qui stocke les mots.

- **[api](api/Dockerfile)** - une API Java REST qui sert les mots lus dans la base de données
- **[web](web/Dockerfile)** - une application web Go qui appelle l'API et construit des mots en phrases
- **db** - une base de données Postgres qui stocke les mots.

## Architecture

![Architecture diagram](architecture.excalidraw.png)

## Build and run in Docker Compose

construire et exécuter dans Docker Compose
La seule exigence pour construire et exécuter l'application à partir des sources est Docker. Clonez ce repo et utilisez Docker Compose pour construire toutes les images. Vous pouvez utiliser la nouvelle V2 Compose avec docker compose ou le CLI classique docker-compose :

```shell
docker compose up --build
```
Vous pouvez également récupérer des images préconstruites depuis Docker Hub en utilisant docker compose pull.


## Deploy using Kubernetes manifests

Vous pouvez déployer la même application sur Kubernetes à l'aide de l'application [Kustomize configuration](./kustomization.yaml). Il définira tous les objets de déploiement et de service nécessaires ainsi qu'un ConfigMap pour fournir le schéma de la base de données.

Appliquer le manifeste en utilisant `kubectl` à la racine du projet :

```shell
kubectl apply -k .
```

Une fois que les pods fonctionnent, naviguez jusqu'à  http://localhost:8080 et vous verrez le site.

Docker Desktop inclut Kubernetes et  la ligne de commande [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) , soafin que vous puissiez travailler directement avec le cluster. Vérifiez que les services sont en place, et vous devriez obtenir une sortie comme celle-ci :
```text
kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
db           ClusterIP      None             <none>        55555/TCP        2m
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          38d
web          LoadBalancer   10.107.215.211   <pending>     8080:30220/TCP   2m
words        ClusterIP      None             <none>        55555/TCP        2m
```
Vérifiez que les pods sont en cours d'exécution et vous devriez voir un pod pour la base de données et les composants Web et cinq pods pour l'API de mots :

```text
kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
db-8678676c79-h2d99    1/1       Running   0          1m
web-5d6bfbbd8b-6zbl8   1/1       Running   0          1m
api-858f6678-6c8kk     1/1       Running   0          1m
api-858f6678-7bqbv     1/1       Running   0          1m
api-858f6678-fjdws     1/1       Running   0          1m
api-858f6678-rrr8c     1/1       Running   0          1m
api-858f6678-x9zqh     1/1       Running   0          1m
```
