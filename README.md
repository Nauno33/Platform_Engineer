# Plateform Engineer

Ce projet me permet de réaliser les travaux pratiques du Bootcamp Platform Engineer Kubernetes de EazyTraining

## Labs
La totalité des enoncés est situé dans ./src

```bash
git clone https://github.com/Nauno33/platform_engineer.git
cd platform_engineer/src
```

### lab-00
Ce premmier lab permet le provisionnement de l'infra du projet: déploiement d'un cluster kubernetes vanilla en utilisant virtualbox à l'aide de vagrant. Il est à noté que l'infra peut également être provisionner chez des cloud provider.

```bash
git clone https://github.com/eazytraining/kubernetes-certification-stack
cd kubernetes-certification-stack
vagrant up
```
Attendre plusieurs minutes afin que l'installation se termine:

```bash
Vagrant global-status

id       name    provider   state   directory
-------------------------------------------------------------------------------------------------------
71301e8  master  virtualbox running D:/Documents/data/platform_engineer/kubernetes-certification-stack 
8300698  worker1 virtualbox running D:/Documents/data/platform_engineer/kubernetes-certification-stack 
```

Une fois déployé, il est maintenant possible d'accéder à son cluster kubernetes en ssh sur le node master

```bash
vagrant ssh master

Last login: Sat May  4 06:04:46 2024 from 10.0.2.2
[vagrant@master ~]$ 
```

Ne pas oublier de passer en root
```bash
sudo su -
```

Afin de s'assurer que le cluster kubernetes est bien opérationnel, il est possible de réaliser les requête api vers le cluster :
```bash
 k get no

NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   5m   v1.29.1
worker1   Ready    <none>          5m   v1.29.1
```

D'un point de vue sécurité, il est important de ne pas rendre le kubeconfig accessible à tous les utilisateurs:
```bash
chmod 600 /root/.kube/config
```

#### OPTION
Plusieurs outils peuvent être nécessaire afin de faciliter l'administration du cluster, voici comment les installer :

##### k9s
```bash
wget https://github.com/derailed/k9s/releases/download/v0.32.4/k9s_linux_amd64.rpm
yum install k9s_linux_amd64.rpm
```

execution en tapant "k9s"

##### kns (namespace switch)
```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
curl https://raw.githubusercontent.com/blendle/kns/master/bin/kns -o /usr/local/bin/kns && chmod +x $_
```

#### Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### lab-01
Ce lab permet d'avoir une description des composants de l'application "app-conference" que notre infrastructure va supporter.

### lab-02
Ce lab va permettre d'installer quelques outils afin de pouvoir exposer et permettre du stockage persistant pour les applications que nous déploierons sur le cluster kubernetes.

#### install baremetal loadbalancer : metallb and nginx ingress controller
Ces deux stacks vont permettre d'installer un loadbalancer et un ingress controller au sein de notre cluster

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
kubectl apply -f https://raw.githubusercontent.com/eazytraining/kubernetes-certified-administrator-training/v1.29/09_lab-09/ip_address_pool/ipaddresspool.yml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/cloud/deploy.yaml
```

Comme nous pouvons le voir, les deux premieres executions permettent d'installer le core du loadbalancer nommé metallb puis de déployer la ressource L2Advertisement qui est associé à la ressource IPAddressPool nommé default qui permet de donner une range de vip pour notre exposition. Cependant, l'ingressclass "nginx" n'aura qu'une seule IP fixe pour son exposition.

La dernière execution permet d'installer l'ingress-controller de type nginx, cette stack déploiera également la ressource "IngressClass" nommé "nginx" qui permettra aux ressources de type ingress de nos application d'être exposé sur l'IP externe choisie ddans l'IPAddressPool.
Cependant, il est interessant de noter que l'installation déploiera la version controller-v1.7.0, qui est une version ne supportant que les kubernetes de version 1.27, 1.26, 1.25, 1.24, or nous utilisons dans le cadre de cette formation, un kubernetes en version 1.29.2.
Même si cette version n'ont supporté est fonctionnel, il est cependant plus interessant de passer sur une version supporté d'un point de vue sécuritaire:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

en accord avec la table des versions de l'ingress-nginx:

### Supported Versions table

Supported versions for the ingress-nginx project mean that we have completed E2E tests, and they are passing for
the versions listed. Ingress-Nginx versions **may** work on older versions, but the project does not make that guarantee.

|  Supported  | Ingress-NGINX version | k8s supported version        | Alpine Version | Nginx Version | Helm Chart Version |
|:--:|-----------------------|------------------------------|----------------|---------------|------------------------------|
| 🔄 | **v1.10.1**            | 1.29, 1.28, 1.27, 1.26        | 3.19.1         | 1.25.3        | 4.10.1*                 |
| 🔄 | **v1.10.0**            | 1.29, 1.28, 1.27, 1.26        | 3.19.1         | 1.25.3        | 4.10.0*                 |
| 🔄 | **v1.9.6**            | 1.29, 1.28, 1.27, 1.26, 1.25        | 3.19.0         | 1.21.6        | 4.9.1*                 |
| 🔄 | **v1.9.5**            | 1.28, 1.27, 1.26, 1.25        | 3.18.4         | 1.21.6        | 4.9.0*                       |
| 🔄 | **v1.9.4**            | 1.28, 1.27, 1.26, 1.25        | 3.18.4         | 1.21.6        | 4.8.3                        |
| 🔄 | **v1.9.3**            | 1.28, 1.27, 1.26, 1.25        | 3.18.4         | 1.21.6        | 4.8.*                        |
| 🔄 | **v1.9.1**            | 1.28, 1.27, 1.26, 1.25        | 3.18.4         | 1.21.6        | 4.8.*                        |
| 🔄 | **v1.9.0**            | 1.28, 1.27, 1.26, 1.25        | 3.18.2         | 1.21.6        | 4.8.*                        |
|  | v1.8.4            | 1.27, 1.26, 1.25, 1.24        | 3.18.2         | 1.21.6        | 4.7.*                        |
|  | v1.7.1            | 1.27, 1.26, 1.25, 1.24        | 3.17.2         | 1.21.6        | 4.6.*              |
|    | v1.6.4                | 1.26, 1.25, 1.24, 1.23       | 3.17.0         | 1.21.6        | 4.5.*              |
|    | v1.5.1                | 1.25, 1.24, 1.23             | 3.16.2         | 1.21.6        | 4.4.*              |
|    | v1.4.0                | 1.25, 1.24, 1.23, 1.22       | 3.16.2         | 1.19.10†      | 4.3.0              |
|    | v1.3.1                | 1.24, 1.23, 1.22, 1.21, 1.20 | 3.16.2         | 1.19.10†      | 4.2.5              |

See [this article](https://kubernetes.io/blog/2021/07/26/update-with-ingress-nginx/) if you want upgrade to the stable
Ingress API.


Nous pouvons maintenant voir que nous avons un ingressclass nommé "nginx" et notre ingress-nginx esposant sur l'IP externe 192.168.99.150 à l'aide notre metallb par la méthode L2.
```bash
 k get ingressclass

NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       3m24s

k get svc -n ingress-nginx -o wide

NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE     SELECTOR
ingress-nginx-controller             LoadBalancer   10.99.65.98     192.168.99.150   80:30373/TCP,443:32364/TCP   3m12s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
ingress-nginx-controller-admission   ClusterIP      10.110.43.139   <none>           443/TCP                      3m12s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-ngin
```


#### Install Storage provisionner
Nous allons maintenant installer un stockage provisionner afin de permettre à nos applications d'avoir du stockage persistant de type nfs.

```bash
nfs_base_url=https://raw.githubusercontent.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner-1.8.0/deploy/kubernetes

kubectl create -f ${nfs_base_url}/deployment.yaml
kubectl create -f ${nfs_base_url}/rbac.yaml
kubectl create -f ${nfs_base_url}/class.yaml
```

Nous allons ajouter une annotation à la storageclass "example-nfs" afin quelle soit par default :

```bash
kubectl patch storageclass example-nfs -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```