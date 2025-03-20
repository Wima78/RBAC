# RBAC: Comment fonctionne le contrôle d'accès basé sur les rôles dans Kubernetes

Voici un guide sur le fonctionnement du contrôle d'accès basé sur les rôles (RBAC) dans Kubernetes:

## Étape 1: Création d'un namespace
Pour isoler les ressources, créez un nouveau namespace:
```
kubectl create namespace rbac
```

## Étape 2: Création d'un compte de service pour les tests
Nous utiliserons un compte de service Kubernetes pour simuler un utilisateur.

Créez un compte de service appelé dev-user dans le namespace rbac:
```
kubectl create serviceaccount dev-user --namespace rbac
```

Vous pouvez afficher le compte de service:
```
kubectl get serviceaccount dev-user --namespace rbac
```

## Étape 3: Création d'un rôle avec des permissions limitées
Créons un rôle qui permet au compte de service d'effectuer uniquement des opérations de base sur les Pods dans le namespace rbac (par exemple, visualiser les Pods, mais pas les créer ou les supprimer).

Enregistrez le YAML suivant sous view-pods-role.yaml et appliquez-le au cluster:
```
kubectl apply -f view-pods-role.yaml
```

Vous pouvez confirmer que le rôle a été créé:
```
kubectl get role view-pods --namespace rbac
```

## Étape 4: Lier le rôle au compte de service
Ensuite, créez un RoleBinding pour associer le rôle view-pods au compte de service dev-user.

Enregistrez le YAML sous view-pods-rolebinding.yaml et appliquez-le:
```
kubectl apply -f view-pods-rolebinding.yaml
```

Testez si le dev-user peut maintenant créer des Pods:
```
kubectl auth can-i create pods --namespace rbac --as=system:serviceaccount:rbac:dev-user
```

## Étape 5: Tester l'accès avec la commande can-i
Pour tester le contrôle d'accès, utilisez la commande kubectl auth can-i.

Vérifiez si dev-user peut lister les Pods:
```
kubectl auth can-i list pods --namespace rbac --as=system:serviceaccount:rbac:dev-user
```

Vous devriez voir une réponse "yes" puisque nous avons donné au compte de service dev-user la permission de lister les Pods.

Vérifiez si dev-user peut créer des Pods:
```
kubectl auth can-i create pods --namespace rbac --as=system:serviceaccount:rbac:dev-user
```

## Étape 6: Créer un Pod en tant que dev-user
Puisque l'utilisateur dev-user a maintenant la permission de créer des Pods (selon le contexte), essayons d'en créer un. Créez un fichier manifeste de Pod simple, puis essayez de créer le Pod en tant que dev-user:

```
kubectl create -f pod.yaml --as=system:serviceaccount:rbac:dev-user
```

Vérifiez que le Pod a été créé avec succès:
```
kubectl get pods --namespace rbac
```

## Étape 7: Nettoyage
Après avoir terminé, nettoyez les ressources pour éviter l'encombrement de votre cluster:
```
kubectl delete namespace rbac
```
