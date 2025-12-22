# 1. rke2-moodle

Dans ce tutoriel, nous allons déployer moodle en tant que webservice sur un cluster kubernetes.

Premierement nous commenceront par installer kubernete via Rancher2 (rke2), qui utilise containerd comme engine à la place de docker engine.

- Pre-requis: ouverture des ports
<img width="382" height="382" alt="Capture d’écran 2025-12-21 101401" src="https://github.com/user-attachments/assets/8da3ed29-24c1-4341-9365-f6c26f0406c2" />

<img width="382" height="382" alt="image" src="https://github.com/user-attachments/assets/463df7cc-c9b5-46f3-9487-0be6002d6736" />

- Stack: 2 debian12 (rke2master et rke2node), rke2, cilium,

### rke2 architecture
<img width="600" height="250" alt="Capture d’écran 2025-12-21 202723" src="https://github.com/user-attachments/assets/d697bce0-0b52-4fd0-a355-e6d77b82e309" />


## Installation rke2 sur `rke2master`
- Mise à jour des paquets sur les deux vms 
```
sudo apt-get update && sudo apt-get upgrade -y
```
- Installation et démarrage du service rke

```
sudo su -
```

```
curl -sfL https://get.rke2.io | sh -
```

```
systemctl enable rke2-server.service
```

```
systemctl start rke2-server.service
```
- Suivre en logs les configurations

<img width="382" height="553" alt="Capture d’écran 2025-12-21 085907" src="https://github.com/user-attachments/assets/46b74463-17f3-4a31-b6e2-e6b0e45ada11" />

- On exporte le chemin en variables d'environnement les binaires

```
export PATH=$PATH:/var/lib/rancher/rke2/bin/
```

<img width="382" height="37" alt="Capture d’écran 2025-12-21 094515" src="https://github.com/user-attachments/assets/f6d8f4d9-dec9-4afb-831f-96df179623d5" />

<img width="382" height="31" alt="Capture d’écran 2025-12-21 095435" src="https://github.com/user-attachments/assets/8d0c36b8-6916-4e69-9765-e42b3a8459c5" />


- On configure kubectl
  
```
mkdir -p ~/.kube
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown $(whoami):$(whoami) ~/.kube/config
```
<img width="655" height="55" alt="Capture d’écran 2025-12-21 095615" src="https://github.com/user-attachments/assets/4eb4748e-4613-47be-b078-e299f0f44647" />

- Le token qui servira à conneter le rke2node
<img width="920" height="71" alt="Capture d’écran 2025-12-21 092156" src="https://github.com/user-attachments/assets/b12ff189-9690-4b86-8a0a-7d321da87b62" />

`K107d9c2f590b3a10c6a1bd03e5ba2a431af0a3cda6b0270614d568921e9e746dd5::server:1e31961f57df49a3c03f9e1b51548fbf`

## Installation rke2 sur `rke2node`

```
sudo su -
```

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
```

```
systemctl enable rke2-agent.service
```

```
mkdir -p /etc/rancher/rke2/
```
- On ajoute le token et l'adresse ip, informations récuperer sur le rke2master
```
vim /etc/rancher/rke2/config.yaml

```

```
server: https://192.168.1.124:9345
token: K107d9c2f590b3a10c6a1bd03e5ba2a431af0a3cda6b0270614d568921e9e746dd5::server:1e31961f57df49a3c03f9e1b51548fbf
```

```
systemctl start rke2-agent.service
```
- Suivi des logs
<img width="542" height="267" alt="image" src="https://github.com/user-attachments/assets/353ac38e-65b1-4e7b-9177-f92f650b4f4c" />

- On Vérifie bien que le node rke2node s'est bien connecter au cluster
<img width="690" height="77" alt="Capture d’écran 2025-12-21 100429" src="https://github.com/user-attachments/assets/0e8648c8-e490-406c-8c62-f760bb3e8f89" />

- Rappels des commandes kubectl
<img width="622" height="500" alt="Capture d’écran 2025-12-22 045354" src="https://github.com/user-attachments/assets/a9ba7ec0-1993-4065-b2d7-cd607e7bbd07" />

- On vérifie l'état de nos pods
<img width="877" height="182" alt="image" src="https://github.com/user-attachments/assets/c95cf226-6ad6-4dda-86d8-0e592a882ef5" />

- Tout les manifests du `/var/lib/rancher/rke2/server/manifests` n'ont pas été déployé `kubectl get addon -A`
<img width="598" height="167" alt="Capture d’écran 2025-12-22 064645" src="https://github.com/user-attachments/assets/e1c8ce86-2a71-44cd-8b34-f865450c795b" />

- Les logs des pods sont `/var/log/pods` ou ` kubectl logs nom du pod -m kube-system` ou ` kubectl cluster-info dump | grep "error" | head` ou `/var/lib/rancher/rke2/agent/logs/kubelet.log`

<img width="658" height="42" alt="Capture d’écran 2025-12-22 102339" src="https://github.com/user-attachments/assets/940a2a69-fa56-4539-85e9-9b4a6f37c2a5" />

- helm version renvoie command not found et on remarque se sont les pods créés à partir des charts helm qui ne sont pas lancés. Installons helm le master
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash
```

```
helm ls --all-namespaces
```
## Configuration cni Cilium
C'est canal qui est installé comme cni par default, nous allons mettre à la place Cilium pour générer le trafic dans le cluster.


## Service Moodle 

## Installation du Dashboard


# 2 Gestion des upgrade

# 3 Gestion des backups

# 4 Suppression de l'infra

- Source:
  
  https://docs.rke2.io/install/quickstart

  https://www.youtube.com/watch?v=CVyBlwYYHDQ&list=WL&index=28&t=14556s

  https://www.youtube.com/watch?v=GAKttTUTWeQ&t=136s
