# rke2-moodle

Dans ce tutoriel, nous allons déployer moodle en tant que webservice sur un cluster kubernetes.

Premierement nous commenceront par installer kubernete via Rancher2 (rke2), qui utilise containerd comme engine à la place de docker engine.

- Pre-requis: ouverture des ports
<img width="382" height="382" alt="Capture d’écran 2025-12-21 101401" src="https://github.com/user-attachments/assets/8da3ed29-24c1-4341-9365-f6c26f0406c2" />

<img width="382" height="382" alt="image" src="https://github.com/user-attachments/assets/463df7cc-c9b5-46f3-9487-0be6002d6736" />

- Stack: 2 debian12 (rke2master et rke2node), rke2, cilium,
  
## Installation rke2 sur `rke2master`
- Mise à jour des deux vms 
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

- On exporte le chemin en variables d'environnement les binaires ``
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
<img width="942" height="567" alt="image" src="https://github.com/user-attachments/assets/353ac38e-65b1-4e7b-9177-f92f650b4f4c" />

- On Vérifie bien que le node rke2node s'est bien connecter au cluster
<img width="690" height="77" alt="Capture d’écran 2025-12-21 100429" src="https://github.com/user-attachments/assets/0e8648c8-e490-406c-8c62-f760bb3e8f89" />


## déploiement du service 


- Source: https://docs.rke2.io/install/quickstart
