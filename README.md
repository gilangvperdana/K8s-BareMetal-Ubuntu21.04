# K8s Baremetal mini-playGround
    Learn a little about Baremetal Kubernetes. 
    But to play with me, make sure you have installed the local kubernetes (baremetal).
```Environment:
Environment:
    1. 3x Ubuntu21.04 Server Local (VirtualBox).
    2. 1Master.
    3. 2Worker.
    4. NIC (NAT,HostOnly,Bridge).
    5. Some IPv4 addresses for MetalLB to hand out.

Mission:
    1. Deploy metalLB
    2. Deploy Kubernetes Dashboard
    3. Deploy Nginx Ingress Controller
    4. Create 3 deployments and expose using the ingress hostname
    
Goals:
   1. Can implement load balancers on kubernetes bare metal.
   2. Can use Kubernetes dashboard for monitoring or cluster control.
   3. Can access to the cluster using only 1 ingress endpoint.
```

## 1. Deploy MetalLB
    MetalLB source : https://metallb.universe.tf/installation/
```
Installation:
Goes to Directory 1.MetalLB, and apply that 3 file.
$ kubectl create -f 1.NS-MLLB.yaml,2.MLLB.yaml
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

Check Metallb:
$ kubectl get all -n metallb-system (make sure all components are running).

Last, deploy ConfigMap:
$ kubectl create -f 3.ConfigMap-MLLB.yaml
If you want to custom your ip loadbalancer, you can change ip on file 3.ConfigMap-MLLB.yaml and go to line 12.

In some cases sometimes the ip remains "pending", if you experience this, please do this:
$ kubectl edit configmap -n kube-system kube-proxy
add this to this section:
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
strictARP: true
```

## 2. Deploy Kubernetes Dashboard with LoadBalancer
```
Installation:
Goes to Directory 2.Dashboard-LoadBalancer, and apply that 2 file.
$ kubectl create -f 1.Dashboard.yaml
$ kubectl create -f 2.ServiceLB-Dashboard.yaml

Check K8s Dashboard Component:
$ kubectl get pod -A -n kubernetes-dashboard
$ kubectl get service -A n kubernetes-dashboard (check the external ip loadbalancer)

Access kubernetes dashboard:
https://iploadbalancer/#/login

Generate token:
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

$ cat <<EOF | kubectl apply -f -
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
EOF

Print Token:
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

Copy your token to login.

Clean up User Token:
$ kubectl -n kubernetes-dashboard delete serviceaccount admin-user
$ kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
```

## 3. Deploy NGINX Ingress Controller
```
Installation:
Goes to Directory 3.Ingress-Nginx-Controller, and apply that file.
$ kubectl create -f 1.nginx-v0.33.0.yaml

Check Componenent:
$ kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx --watch (wait until all components are running).
  
Check controller Version:
$ kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version

Check controller service:
$ kubectl get service -A (wait until the controller gets the ip from MetalLB).
```

## 4. Test Deployment Ingress
```
Installation:
Goes to Directory 4.Test-Deployment-Ingress, and apply that 4 file.
$ kubectl create -f Deployment1.yaml,Deployment2.yaml,Deployment3.yaml,Ingress-3Deployment.yaml

Check :
$ kubectl get pod (Make sure all pod running)
$ kubectl get ingresses (Make sure HOSTS appear, copy that if you want to test)
# kubectl get services -n ingress-nginx (Copy External IP, if you want to test)

Edit in your Hosts file OS:
on Ubuntu:
$ nano /etc/hosts
ip-external-ingress-nginx     hosts-url-ingresses
ip-external-ingress-nginx     hosts-url-ingresses
ip-external-ingress-nginx     hosts-url-ingresses

Access that ingress:
http://hosts-url-ingresses (http://ex.example.com) #test access must be same network.
```


