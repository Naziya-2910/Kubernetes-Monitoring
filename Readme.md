1. Install HELM
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
DESIRED_VERSION=v3.8.2 bash get_helm.sh
curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2
## To verify the helm version
helm version  
2. Install EKSCTL
sudo apt update && sudo apt upgrade -y
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
3. Add Helm Stable Charts
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
4. Create Prometheus Namespace
kubectl create namespace prometheus
kubectl get ns
5. Install Prometheus using Helm
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
kubectl get pods -n prometheus
kubectl get svc -n prometheus
6. Expose Prometheus and Grafana to the external world
Let’s expose Prometheus and Grafana to the external world there are 2 ways to expose

Node Port

LoadBalancer

let’s go with the LoadBalancer to attach the load balancer we need to change from ClusterIP to LoadBalancer command to get the svc file

kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
change it from Cluster IP to LoadBalancer after changing make sure you save the file

kubectl get svc -n prometheus
a load balancer has been provisioned for Prometheus, allowing access via the link provided on port 9090.

Now,let’s change the SVC file of the Grafana and expose it to the outer world

command to edit the SVC file of grafana

kubectl edit svc stable-grafana -n prometheus
change the type from cluster IP to LoadBalancer as donce before

kubectl get svc -n prometheus
Copy the loadBalancer external IP and ping it in the browser It will be prompeted for Username & Password

Commands to get username
kubectl get secret stable-grafana -n prometheus \
-o jsonpath="{.data.admin-user}" | base64 --decode ; echo
Command to get password
kubectl get secret stable-grafana -n prometheus \
-o jsonpath="{.data.admin-password}" | base64 --decode ; echo
Default Username : admin
Password: nMeKuFKb71zaS5biQjwLLDgEhmTykYe9vCvVZLXJ
Now configure the Dashboards and you can visualize the K8s Cluster
