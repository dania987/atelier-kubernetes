# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :Dania jichi
**Groupe :

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
1 nœud control-plane (qui fait office de master, exécutant l'API Server, le planificateur et le gestionnaire de contrôle pour administrer le cluster).

2 nœuds workers (agent-0 et agent-1), qui hébergent et exécutent les conteneurs (pods) applicatifs demandés par les utilisateurs.

  
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   docker ps montre la liste des conteneurs Docker en cours d'exécution sur la machine hôte. On y retrouve les conteneurs correspondant aux nœuds du cluster (k3d-lab-server-0, k3d-lab-agent-0, k3d-lab-agent-1) ainsi qu'un conteneur de load-balancer (k3d-lab-serverlb). Concrètement, dans k3d, un « nœud » Kubernetes est un conteneur Docker à l'intérieur duquel tourne une instance légère de Kubernetes (k3s).

   

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   La commande `kubectl create deployment` permet de créer un **Deployment**, qui met ensuite en place un **ReplicaSet** chargé de gérer la création et le nombre de **Pods** nécessaires. Lorsqu'on utilise `kubectl expose`, un **Service** peut également être créé afin de permettre l'accès aux Pods et de rediriger le trafic réseau vers eux.

   
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Lorsqu'un Pod est supprimé manuellement, Kubernetes en crée automatiquement un nouveau avec un nom différent. Ce mécanisme est assuré par le **ReplicaSet**, contrôlé par le Deployment, qui surveille en permanence le nombre de Pods actifs afin de maintenir le nombre de réplicas souhaité (dans ce cas, un seul Pod).

   

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   Non, le message n'est plus présent. En effet, le système de fichiers d'un conteneur est temporaire. Lorsque le Pod a été supprimé puis recréé par le ReplicaSet, un nouveau conteneur a été lancé à partir de l'image Docker d'origine (`nginx`). La modification effectuée manuellement avec `kubectl exec` a donc été perdue.

   
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >La commande `port-forward` établit une connexion directe avec un Pod précis, même lorsqu'elle passe par un Service. Si ce Pod est supprimé ou tombe en panne, la connexion est interrompue, car elle ne peut plus communiquer avec l'ancien conteneur. Le tunnel s'arrête alors et renvoie une erreur (Exit 1).

   

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   1. `kubectl create` est une commande **impérative** qui permet de créer une ressource à partir d'un fichier. Si cette ressource existe déjà, Kubernetes renvoie une erreur.

  2. `kubectl apply` utilise une méthode **déclarative** qui consiste à appliquer la configuration définie dans un fichier YAML. Si la ressource n'existe pas, elle est créée ; si elle existe déjà, elle est mise à jour selon les modifications apportées au fichier.

     
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le `selector` d'un Service sert à identifier automatiquement les Pods qui possèdent les labels correspondants afin de leur transmettre le trafic réseau. Si aucun Pod ne correspond à ces labels, le Service est quand même créé, mais ses **Endpoints** restent vides. Le trafic ne peut donc pas être acheminé vers les Pods, ce qui empêche l'accès à l'application.

   

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   kubectl get pods -l app=site -o wide

   
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
Le Pod est resté bloqué à cause d'une erreur de récupération de l'image, avec un statut comme `ImagePullBackOff` ou `ErrImagePull`. En utilisant `kubectl describe pod ...` ou `kubectl get pods`, nous avons pu consulter les événements et constater que le tag `9.99` n'était pas disponible sur le registre distant.

    
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui, au moins en partie, grâce aux autres réplicas encore actifs. Lors d'une mise à jour progressive (*Rolling Update*), Kubernetes conserve les anciens Pods tant que les nouveaux ne sont pas correctement démarrés et prêts à fonctionner (vérification par la **Readiness Probe**). Cela permet de maintenir le service disponible et d'éviter une interruption pendant la mise à jour.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://shiny-invention-pj5jjpx7vp4w26wjj-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >Portainer et `kubectl` utilisent tous les deux le même composant central de Kubernetes : l'**API Server**. La différence est que Portainer propose une interface graphique, tandis que `kubectl` fonctionne en ligne de commande. Les deux permettent donc de gérer le cluster en envoyant leurs requêtes à l'API Server.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
