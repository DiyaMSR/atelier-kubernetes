# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : MAISURIA Diya
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >kubectl get nodes affiche 3 nœuds au total :
   1 nœud control-plane (qui fait office de master, exécutant l'API Server, le planificateur et le gestionnaire de contrôle pour administrer le cluster).
   2 nœuds workers (agent-0 et agent-1), qui hébergent et exécutent les conteneurs (pods) applicatifs demandés par les utilisateurs.

2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >docker ps montre la liste des conteneurs Docker en cours d'exécution sur la machine hôte. On y retrouve les conteneurs correspondant aux nœuds du cluster (k3d-lab-server-0, k3d-lab-agent-0, k3d-lab-agent-1) ainsi qu'un conteneur de load-balancer (k3d-lab-serverlb). Concrètement, dans k3d, un « nœud » Kubernetes est un conteneur Docker à l'intérieur duquel tourne une instance légère de Kubernetes (k3s).

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >La commande kubectl create deployment crée principalement un Deployment, qui génère à son tour un ReplicaSet, lequel gère la création effective d'un ou plusieurs Pods. Si on expose le déploiement (kubectl expose), un objet de type Service est également créé pour router le trafic réseau vers ces pods.
   
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Lorsque l'on supprime manuellement le pod, Kubernetes en recrée immédiatement un nouveau sous un autre nom. Cela est dû au ReplicaSet (géré par le Deployment) dont le rôle est de veiller en permanence à ce que l'état réel du système corresponde à l'état désiré (ici, maintenir 1 réplica en vie, c'est la boucle de réconciliation).

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >Non, le message a disparu. Le système de fichiers d'un conteneur est éphémère. Lorsque le pod a été supprimé puis recréé par le ReplicaSet, un nouveau conteneur propre a été instancié à partir de l'image Docker d'origine (nginx), effaçant la modification manuelle faite précédemment via kubectl exec.
   
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Un port-forward établi vers un pod spécifique (ou même via un Service qui cible les pods) est lié à l'instance précise du pod actif. Lorsque ce pod meurt et est remplacé, la redirection vers l'ancien conteneur est rompue et le tunnel s'arrête (Exit 1).

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >kubectl create est une commande impérative qui demande de créer une ressource à partir d'un fichier ; si la ressource existe déjà, elle génère une erreur.
   >kubectl apply est une approche déclarative qui applique l'état décrit dans les fichiers YAML : si la ressource n'existe pas, elle est créée, et si elle existe déjà, elle est mise à jour (application d'un patch intelligent basé sur l'état désiré).
   
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le selector d'un Service permet de cibler dynamiquement les pods portant des labels spécifiques afin de leur router le trafic réseau. S'il ne correspond à aucun pod, le Service est créé mais ses Endpoints restent vides, rendant l'accès impossible ou provoquant des erreurs de routage (le trafic ne trouve aucune cible).

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
 > Les 3 réplicas sont répartis sur chacun des 3 nœuds du cluster : `k3d-lab-agent-0`, `k3d-lab-server-0` et `k3d-lab-agent-1`.
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
   > Le nouveau pod créé s'est retrouvé bloqué dans le statut d'erreur **`ErrImagePull`** puis **`ImagePullBackOff`**. La commande **`kubectl describe pod -l app=site`** (via la section `Events` en bas de la sortie) a permis de voir la cause exacte : l'image `nginx:9.99` est introuvable sur Docker Hub (`not found`).

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
   > Oui, le site est resté parfaitement accessible. Lors d'une mise à jour en *rolling update*, Kubernetes conserve les anciens pods actifs (`Running`) tant que le nouveau pod de remplacement n'est pas démarré et prêt. Les 3 anciens réplicas ont donc continué à servir le trafic normalement.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
