
# Terminal Command for K8s Kind Voting App

## 1. Creating and Managing Kubernetes Cluster with Kind

- Create a bash script `install_kind.sh` for installing kind
  ```bash
  #!/bin/bash
  # For AMD64 / x86_64
  [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
  chmod +x ./kind
  sudo cp ./kind /usr/local/bin/kind
  rm -rf kind
  ```
  
- Give Exectuable permission to file
  ```bash
  chmod +x install_kind.sh
  ```
- Run the Script
  ```bash
  ./install_kind.sh
  ```
- Create a `config.yml` for installing cluster in Docker
  ```bash
  kind: Cluster
  apiVersion: kind.x-k8s.io/v1alpha4

  nodes:
    - role: control-plane
      image: kindest/node:v1.30.0
    - role: worker
      image: kindest/node:v1.30.0
    - role: worker
      image: kindest/node:v1.30.0
  ```

- Create a 3-node Kubernetes cluster using Kind:
  ```bash
  kind create cluster --config=config.yml --name my-cluster
  ```

---

## 2. Installing kubectl

- Install kubectl using `install_kubectl.sh` script
  ```bash
  #!/bin/bash

  # Variables
  VERSION="v1.30.0"
  URL="https://dl.k8s.io/release/${VERSION}/bin/linux/amd64/kubectl"
  INSTALL_DIR="/usr/local/bin"

  # Download and install kubectl
  curl -LO "$URL"
  chmod +x kubectl
  sudo mv kubectl $INSTALL_DIR/
  kubectl version --client

  # Clean up
  rm -f kubectl

  echo "kubectl installation complete."
  ```

- Give Exectuable permission to file
  ```bash
  chmod +x install_kubectl.sh
  ```

- Check cluster information:
  ```bash
  kubectl cluster-info 
  kubectl get nodes
  kind get clusters
  ```

---

## 3. Managing Docker and Kubernetes Pods

- Check Docker containers running:
  ```bash
  docker ps
  ```

- List all Kubernetes pods in all namespaces:
  ```bash
  kubectl get pods -A
  ```
---

## 4. Installing Argo CD

- Create a namespace for Argo CD:
  ```bash
  kubectl create namespace argocd
  ```

- Apply the Argo CD manifest:
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
  
- Check pods in Argo CD namespace:
  ```bash
  kubectl get pods -n argocd
  ```

- Check services in Argo CD namespace:
  ```bash
  kubectl get svc -n argocd
  ```

- Expose Argo CD server using NodePort:
  ```bash
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
  ```

- Forward ports to access Argo CD server:
  ```bash
  kubectl port-forward -n argocd service/argocd-server 8443:443 --address 0.0.0.0 &
  ```
**Note:-** Enable port 8443 on AWS Instance Security Groups


---

## 5. Argo CD Initial Admin Password

- Retrieve Argo CD admin password:
  ```bash
  kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
  ```

---

## 6. Cloning and Running the Example Voting App

- Clone the voting app repository:
  ```bash
  git clone https://github.com/harshitsahu2311/Voting-app-kubernetes-Project.git
  cd Voting-app-kubernetes-Project/
  ```

- Forward local ports for accessing the voting and result apps:
  ```bash
  kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
  kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
  ```

---

## 7. Installing Kubernetes Dashboard

- Deploy Kubernetes dashboard:
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
  ```

- Create a manifest `dashboard-adminuser.yml` for creation of `admin-user`:
  ```bash
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: admin-user
    namespace: kubernetes-dashboard
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: admin-user
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kubernetes-dashboard
  ```

- Run the manifest file:
  ```bash
  kubectl apply -f dashboard-adminuser.yml
  ```
  
- Check the `admin-user` Service:
  ```bash
  kubectl get svc -n kubernetes-dashboard
  ```

- Create a token for dashboard access:
  ```bash
  kubectl -n kubernetes-dashboard create token admin-user
  ```
  Copy the Token

- Forward local ports for accessing the kubernets dashboard:
  ```bash
  kubectl port-forward -n kubernetes-dashboard svc/kubernetes-dashboard 8080:443 --address 0.0.0.0 &
  ```

---



---



