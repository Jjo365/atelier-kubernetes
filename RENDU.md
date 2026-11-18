# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :JOSEPH NATHAN OUETHY SITCHI

**Groupe :RS2

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >`kubectl get nodes`   affiche trois nœuds. `k3d-lab-server-0` est le nœud du plan de contrôle, qui gère le cluster et décide où placer les pods. `k3d-lab-agent-0` et `k3d-lab-agent-1` sont les deux workers, qui exécutent les pods.
   
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >`docker ps` affiche quatre conteneurs en cours d’exécution. Trois exécutent K3s, avec un conteneur Docker par nœud Kubernetes. Le quatrième, `k3d-lab-serverlb`, est le proxy qui transmet le trafic vers le cluster.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > kubectl create deployment web --image=nginx a créé le Deployment web. Celui-ci a créé le ReplicaSet web-68d995574f, qui a démarré le pod Nginx web-68d995574f-54j6w. La commande kubectl expose a ensuite créé le Service web, de type ClusterIP, sur le port 80.
   
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Après la suppression du pod web-68d995574f-54j6w, Kubernetes a créé automatiquement le pod web-68d995574f-rkbck. Le ReplicaSet du Deployment web a remplacé le pod supprimé pour maintenir le nombre de réplicas demandé, soit un.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non. J’avais modifié `index.html` directement dans le conteneur. Quand le pod a été supprimé, cette modification a disparu. Le nouveau pod est reparti de l’image Nginx d’origine et affiche « Welcome to nginx! ».
   
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Même lancé vers le Service `web`, le `port-forward` se connecte à un pod précis. La suppression de ce pod a coupé le tunnel. En relançant la commande, le tunnel s’est connecté au nouveau pod.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > kubectl create crée une nouvelle ressource et échoue si elle existe déjà. kubectl apply -f lit l’état souhaité dans un fichier YAML et crée ou met à jour la ressource.
   
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector indique au Service vers quels pods envoyer le trafic, en se basant sur leurs labels. Dans mon TP, le Service site sélectionne les pods avec le label app: site. Si aucun pod ne correspond, le Service existe toujours, mais il n’a aucun pod vers lequel envoyer les requêtes.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.

   Mes trois réplicas sont répartis à raison d’un pod par nœud sur `k3d-lab-agent-0`, `k3d-lab-agent-1` et `k3d-lab-server-0`.

   ```text
   NAME                   READY   STATUS    RESTARTS   AGE    IP          NODE               NOMINATED NODE   READINESS GATES
   site-65449cbcd8-7sggr   1/1     Running   0          7m2s   10.42.2.8   k3d-lab-agent-0    <none>           <none>
   site-65449cbcd8-99lvb   1/1     Running   0          6s     10.42.1.5   k3d-lab-server-0   <none>           <none>
   site-65449cbcd8-s7skd   1/1     Running   0          6s     10.42.0.6   k3d-lab-agent-1    <none>           <none>
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le nouveau pod est passé à ErrImagePull, puis à ImagePullBackOff. La commande kubectl describe pod site-5dcd6d449f-df22t a donné la cause dans la section Events. L’image docker.io/library/nginx:9.99 était introuvable.

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui. Pendant la panne, curl affichait encore « Bienvenue sur mon site ». Les trois pods Nginx 1.28 étaient toujours prêts, donc le Service site continuait à leur envoyer le trafic.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://solid-winner-pj79gv9vxp6426qjx-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > La commande kubectl get deployment mario -n default affiche le Deployment mario créé depuis Portainer, avec 1/1 réplica prêt. Portainer et kubectl communiquent avec le même composant du plan de contrôle, l’API Server Kubernetes (kube-apiserver).

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >J’ai réalisé les bonus B, C et D. J’ai ajouté une readinessProbe HTTP sur / ainsi que des demandes et des limites de CPU et de mémoire. Lors du drain de k3d-lab-agent-0, le pod du site évacué a été recréé sur k3d-lab-agent-1. Le uncordon a rendu le worker de nouveau disponible pour accueillir des pods. Enfin, j’ai déployé une deuxième copie du site dans le namespace prod. Dans Portainer, les deux Deployments site, dans default et prod, affichaient chacun 3/3 réplicas prêts.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
