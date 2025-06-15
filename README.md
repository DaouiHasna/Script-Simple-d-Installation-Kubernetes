

# Script Simple d'Installation de Kubernetes

**Auteures** : Hasna Daoui, Diawara Nana
**Projet 3-P2**

---

## Description

Dans un contexte où les volumes de données et la complexité applicative ne cessent de croître,
**Kubernetes** s’impose comme la solution d’orchestration incontournable pour les conteneurs.

Ce projet propose une solution **automatisée, évolutive et reproductible** pour déployer un cluster Kubernetes, intégrant :

* **Kubeflow** pour la gestion des workflows de Machine Learning (MLOps)
* **Rook Ceph** pour un stockage distribué et persistant

Objectif : simplifier la mise en place d’un environnement cloud-native robuste, sécurisé et prêt à l’usage.

---

## Fonctionnalités

* Installation automatique des dépendances (Docker, kubeadm, Containerd, etc.)
* Initialisation du nœud maître avec `kubeadm init`
* Génération d’un token pour les nœuds workers
* Installation de Calico (CNI réseau)
* Déploiement de Rook Ceph pour le stockage distribué
* Installation de Kubeflow pour le MLOps

---

## Structure du projet

```
├── installmaster.sh   # Script à exécuter sur le nœud master
├── installworker.sh   # Script à exécuter sur le(s) nœud(s) worker
```

---

## Prérequis

* Le nœud master doit être en **mode Bridge**
* Une partition de disque dédiée doit être ajoutée avant exécution

---

## Instructions d’exécution

### Sur le nœud **master** :

```bash
sudo nano installmaster.sh
sudo chmod +x installmaster.sh
./installmaster.sh
```

### Sur le(s) nœud(s) **worker** :

```bash
sudo nano installworker.sh
sudo chmod +x installworker.sh
./installworker.sh
```

---

## Dépannage – Erreurs courantes

### 1. Swap activé

**Symptôme :** `swap is enabled`
**Solution :**

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

### 2. Conflit de cgroup driver (Docker vs kubelet)

**Solution :**

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

sudo systemctl restart docker
sudo systemctl restart kubelet
```

---

### 3. Pods bloqués en Pending

**Cause :** CNI non installé
**Solution :**

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

### 4. Kubeflow – "Unable to fetch pipeline runs"

**Solution :**

```bash
kubectl logs -n kubeflow deployment/ml-pipeline-ui
kubectl get svc -n kubeflow
```

---

### 5. Rook Ceph – CrashLoopBackOff

**Cause possible :** Pas de disque libre détecté
**Solution :**

* Vérifier qu’un disque libre est disponible
* Examiner les logs :

```bash
kubectl -n rook-ceph logs <nom-du-pod>
```

---

### 6. Connexion refusée avec `kubectl port-forward`

**Solution :**

```bash
kubectl get pods -n kubeflow
kubectl port-forward svc/ml-pipeline-ui -n kubeflow 8080:80
```

---

## Conclusion

Ce projet permet le déploiement complet et automatisé d’une infrastructure cloud-native, combinant Kubernetes, Kubeflow et Rook Ceph.
Il facilite l’installation d’un environnement évolutif, sécurisé et adapté aux charges de travail intensives comme le Machine Learning.


