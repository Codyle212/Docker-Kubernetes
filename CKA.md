---
title: CKA
created: '2024-03-05T00:11:10.527Z'
modified: '2024-03-29T05:32:54.203Z'
---

# CKA

- Important Practices
  - Cluster Upgrade
  - ETCD backup 2
  - Network Policies
  - View Certificate
  - Ingress
  - Explore Environment+CNI+CoreDNS
  - Cluster Installation

## Tips
- `kubectl run {pod_name} --image={container_image}` can be used to create a pod
- To access other nodes use `ssh node01`
- `kubectl get event -o wide` is used to inspect the events
- practice upgrade **K8S Cluster** with kubeadm , and backup **etcd** in a kubeadm environment
- when looking for components of controlplane, try to get pod information of `kube-apiserver`, setting are specified
- another ways to check setting of a service `ps -aux|grep -i {service}`
### Pods
- Create an NGINX Pod: 
  - `kubectl run nginx --image=nginx`
- Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
  - `kubectl run nginx --image=nginx --dry-run=client -o yaml`
### Deployments
- Create a deployment named nginx with nginx as image and 4 replicas
  - `kubectl create deployment nginx --image=nginx --replicas=4 `
- Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.
  - `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`
- Make necessary changes to the file (for example, adding more replicas) and then create the deployment.
  - `kubectl create -f nginx-deployment.yaml`
- In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.
  - `kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`
- To Scale a Deployment or ReplicaSet 
  - `kubectl scale --replicas=6 rs {replicaset_name}`or `kubectl scale --replicas=6 deploy {deployment_name}`
- To set image of deployment or ReplicaSet
  - `kubectl set image deployment nginx nginx=nginx:1.18`
### Service
- By Default create an ClusterIP, so we don't need to specify type for ClusterIP
- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
  -  the following will automatically use the pod's labels as selectors)
  - `kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`
  -  the follwing pods labels are not used as selectors, instead it will assume selectors as app=redis.
  - `kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`
- Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
  - The following will automatically use the pod's labels as selectors, but node port can be defined
  - `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`
  - The following will not use the pod's labels as selectors, app=nginx is assumed and can't pass in selector
  - `kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`
### Ingress
- Check If there is a ingress class, if so add that as the `ingressClassName` of the defination
- To create a ingress imperatively `kubectl create ingress <ingress-name> --rule="host/path=service:port"`
### Create Role and RoleBinding
- `kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods`
- `kubectl create rolebinding developer-binding-myuser --role=developer --user=myuser`
- add to kubeconfig
- `kubectl config set-credentials myuser --client-key=myuser.key --client-certificate=myuser.crt --embed-certs=true`
- `kubectl config set-context myuser --cluster=kubernetes --user=myuser`
## Introduction
- ReplicaSet can manage pod that is not created under the replicaset defination, hence `selector` is required
- get resource with specific label `kubectl get {resource} --selector {key=value}`
- use apiVersion app/v1 for ReplicaSet, v1 only supports ReplicationController
- Scaling Replica Set, `kubectl scale --replicas=6 -f {file.yaml}` or change replicas field and `kubectl replace -f {file.yaml}`
- Scaling Replica Set, `kubectl scale -replicas=6 replicaset {replicaset_name}`
- Scaling Replica Set, `kubeclt edit replicaset {replicaset_name}` and change Replica value
- Service wiht ClusterIp can be access via the ClusterIP or the Service Name
- adding namespace field under metadata, will specify when namespace this resource is deployed on
- namespace can be specified using the --namespace={ns} flag
- Create name space `kubectl create namespace {ns_name}`
- change namespace `kubectl config set-context$(kubectl config current-context) --namespace={ns_name}`
- Delete or Recreate with `kubectl replace --force -f {resource_file}`
- `-n` and `--namespace` can be used interchangably 
## Scheduling
- specify `nodeName` field under spec of a Pod defination to manually schedule
- use Binding and create a POST request to pod binding api with Binding in JSON as data to
- Annotations are used for infometric purposes, like build version,tool detail,author
<br>
- Taint: NoSchedule|PreferNoSchedule|NoExecute 
  - NoSechedule: will not schedule on any circumstance, PreferNoSchedule: Tries not to schedule on that Node
  - NoExecute: Pod(No toleration) evicted, Pod(toleration+tolerationSeconds) evicted after tolerationSeconds
  - Pod with toleration and without a tolerationSeconds will remain bound to the Node
- Tainting only tells node to accept pod with certain tolerations
- Master Node have a taint that prevents Pod being scheduled on the Node
- To untaint a Node,use `kubectl taint node {node_name} {taint_effect}-`
  - Example `kubectl taint node controlplane node-role.kubernetes.io/master:NoSchedule-`
<br>
- Node Selecotors and Affinity are set on pods so they only run on particular nodes
- to use Node Selector, Node must be labeled before Pod creation, NodeSelector is specified on spec of a Pod
- label nodes `kubectl label nodes {node_name} {label_key=label_val}
- Node Affinity offer fine-grained Node placement for Pod, 2 types
  - `requiredDuringSchedulingIgnoredDuringExecution`, used when pod must be placed according to affinity rules
  - `preferredDuringSchedulingIgnoredDuringExecution`, scheduler will try best to place according to affinity rules
  - planned `requiredDuringSchedulingRequiredDuringExecution`, this evicts any pod that dont follow affinity rule when label changes
<br>
- Resource can be specified when creating pod defination with `resources` tag under containers lists
- Resource requests can be specifed in pod defination with `requests` tag under `resources`, guarantee pod will get requested resource
- Resource limits can be specifed in pod defination with `limits` tag under `resources`, max of what a pod can use
- Pod exceed cpu limits will throttle, Pod can exceed mem limit, but constantly exceed> terminated(OOM Error)
- Limit range defines default value for container and pods which is applied at namespace level, only affect new pod creation 
- Resource Quota can defined at namespace level which limits the total resource a namespace can get
<br>
- DaemonSet Ensure one copy of a pod runs on each node in a cluster, monitoring solution,log agent, kube-proxy, networking
- No shortcut on creating daemonset, but use the K8s Documentation and Search DaemonSet, Similar format to replicaset
<br>
- Static Pod can Pod defination that is put in directory of the work node and managed by kubelet
- the manifest location is configured in kubelet.service. `--pod-manifest-path=/etc/kubernetes/mainfests` when configuring the kubelet 
- alternative path in config in yaml, and use `staticPodPath: /etc/kubernetes/mainfests`
- Static Pod use case, deploy the control plane using docker image, 1.have kubelet,2. put mainfest,3.when crash->restart service
- To identify Static Pod use `kubectl describe pods -A|grep "Controlled By"`, Static Pod are controlled by nodes
- Another way is the look at the name attached to the end of the pod, if it's a node name -> staic pod
- To access other nodes use `ssh node01`
<br>
- For multiple Scheduler, we need to create different scheduler configuration file and create a service with new config file
```
  apiVersion: kubescheduler.config.k8s.io/v1
  kind: KubeSchedulerConfiguration
  profiles:
  - schedulerName: my-scheduler
```
- in service file ExecStart = binary and --config is pass of the config file
- Scheduler can also be deployed as a pod, then pass in scheduler.yaml as a config option in command section of the container
- in Multi-Master Setup, when multiple scheduler are present, leaderElection where can choose the scheduler to lead scheduling activities
- To schedule the pod with specified scheduler, use `schedulerName:{scheduler_name}` under spec of the pod
- priority class have a value that determines what priority is given on scheduling queue
- filtering phase filter out node that don't have requirement for hosting pod
- scoring phase scores base on different weight, i.e free space
- in binding phase , a pod is binded to a node
- Extension point on 4 phases allows us to use our owne plugins
- multiple scheduler profile can be defines for a single node, profiles can have plugins enabled or disable default plugins
## Monitoring and Logging
- Opensource solutions: metricserver,prometheus,elk stack,datadog,dynatrace
- Metric Server is a in memory monitoring solution, does not store on disk
- cAdvisor is responisible for retriving performance metrics from pod and expose them through kubelet api
- minikube> minikube enable addons metrics server, else git clone files and deploy with `kubectl apply`
- `kubectl top node` displays mem and cpu of each node, `kubectl top pod` display performance metrics of pods
- use `kubectl log {pod_name}` to view log, use `-f` for a continous stream, attach containe name if it is a multicontainer pod
## Application Lifecycle Management
- Rollout is used to update the deployment to a new version,  rollout is also done during deployment creation
- Check Status of rollout `kubectl rollout status deployment/myapp-deployment`
- Check Revisions and history of deployment `kubectl rollout history deployment/myapp-deployment`
- Deployment Strategy
  - Recreate, Destory all current pods and recreate new pods, will have downtime
  - Rolling Update(Default), take certain number of curretn pod and replace them with new pod
- Rollout are trigger when defination file is modified and apply or setting changes imperatively
- example of imperative change `kubectl set image deployment/{deployment_name} {image_name}={docker_image}`
- TO rollback `kubectl rollout undo deployment/{deployment_name}`
<br>
- CMD will be replaced by args of docker run, whereas ENTRYPOINTS wiil have args appended to it and executed
- CMD can be used to setup the default behaviour of ENTRYPOINT if no args where passed in, ENTRYPOINT have higher priority
- anything that should be appended to docker run command are in args under containers list, override cmd in Dockerfile
- `command` field in container defination is corresponding to entrypoint of docker image, anything in that field replaces entrypoint
- `--command {command} {arg1}.. {argN}` is used to pass in command in kubectl run , `-- {arg1}.. {argN}` is used to pass in arguments 
<br>
- use env list under containers to specify environmental variables, env vars are list of name,value pairs
- ConfigMap are used to Config in form of key value pairs
- create configmap `kubectl create configmap {config_name} --from-literal={key=value}`, use from literal multiple time to specify config
- create secret `kubectl create secret generic {config_name} --from-literal={key=value}*n`
- create secret from file `kubectl create secret generic {secret_name} --from-file={file_path}`
- when create secret declaratively, data value need to be base64 encoded
- when creating secret as volume, file are created with the key and the content is the secret value
<br>
- Sidecar container can be used  as log agent to collect logs directly from localhost
- There are also the adapter and the ambassador pattern for configuring multi-container pod
- init Container can be used to inspect if other services are running before starting a container
- init containers will be restarted repeatly until it suceed then proceed onto the container in Pod
## Cluster Maintainance
- `--pod-eviction-timeout=5m0s` is a argument set on kube-controller, master node waits for that amount before consider them as dead node
- `kubectl drain node01` will gracefully terminate pod and recreate them on other node
- `kubectl uncordon node01` makes pod schedulable on a node again, needs to be run after drain
- `kubectl cordon node01` makes pod unschedulable on a node
<br>
- Cluster Upgrade one minor version at a time
- When upgrading, start with master node, management is down for the duration
- When upgrading Worker Nodes, All at once, One at a time, One at a time with additional node
- Detailed Deployment steps [K8S Cluster upgrade with Kubeamd](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- when using kubeadm, we can use `kubeadm upgrade plan` to check available packages for different component
- kubeadm will require kubelet on worker node to be upgraded manually
- first upgrade kubeadm then use `kubeamd upgrade apply v{1.current+1.0}`
- to upgrade kubelet, use `apt-get upgrade kubelet=desired_version` then restart kubelet.service
- for worker nodes kubelet, we need to drain the node first then perform the upgrade,
- once done use `kubeadm upgrade node config --kubelet-version {new-version}`, then use uncordon to make is as schedulable
<br>
- Resource configuration can be stored in defination files and query kube-apiserver for details
- ETCD have a configure `--data-dir` that is used to store data
- To use API_Version 3 as default`export ETCDCTL_API=3`, by etcdctl uses api v2
- ETCD have a snapshot save command, `etcdctl snapshot save snapshot.db`(API v3) snapshot with name of snapshot.db is created
- Insepct snapshot status with `ETCDCTL_API=3 etcdctl snapshot status snapshot.db`
- To restore a snapshot
  1. stop kube-apiserver, `service kube-apiserver stop`
  2. `etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup`, data-dir specify the path of the backup file
  3. configure `etcd.service` to use --data-dir=backup_file_directory
  4. run `sudo systemctl daemon-reload && service etcd restart
  5. resart kube-apiserver, `service kube-apiserver start`
- For all etcdctl command specify these fields
  ```
  --endpoints=https://127.0.0.1:2379
  --cacert=/etc/etcd/ca.crt
  --cert=/etc/etcd/etcd-server.crt
  --key=/etc/etcd/etcd-server.key
  ```
- when back up is restore, it is configured as new members of new cluster, if etcd is run as a service, `chown -R etcd:etcd {backup_dir}`
- `--etcd-servers` can be found in kube-apiserver, localhost?stacked:external
- `ectdctl member list` listed out member of the etcd cluster
## Security
- users and accounts are not directly managed by kubernetes, but kubernetes manages service accounts
- kube-apiserver authenticates requests from user before processing it
  - 1.static passwordfile, 2. static token, 3. certificate, 4. 3rd party identity service
- Basic: Create a csv file with user details and pass in `--basic-auth-file=user-details.csv` in kube-api configuration, then restart kubeapi
- Same thing with Kubeamd, pass `--basic-auth-file=user-details.csv` as command, then restart the container
- To use `curl -v -k https://master-node:6443/api/v1/pods -u "user1:password1"`
- Token: Similar to Basic auth, pass in `--token-auth-file=user-token-details.csv` in kubeapi service configuration
- To use `curl .... --header "Authroization: Bearer {token}`
- Both Basic Auth and Token Auth csv can have a 4th field group, Both approaches are not recommanded
<br>
- A certificate is a trusted document that contains a public key and other data of the respective private key owner
- generate a rsa private key `openssl genrsa -out my-bank.key 1024`
- generate public key based on private key `openssl rsa -in my-bank.key -pubout mybank.pem`
- create a cert signing request `openssl req -new -key my-bank.key -out my-bank.csr -subj"/C=US/ST=CA/O=MyOrg, inc./CN=mydomain.com"
- public key or certificate ends with .pem or .cert, private key ends with .key or -key.pem
<br>
- Server
  - Kube-Apiserver have `apiserver.crt` as certificate and `apiserver.key` as private key 
  - ETCD server have `etcdserver.crt` as certificate and `etcdserver.key` as private key
  - kubelet server have `kubelet.crt` as certificate and `kubelet.key` as private key
- Client
  - admin have `admin.cert` as certificate and `admin.key` as private key 
  - kube-scheduler have `scheduler.crt` as certificate and `scheduler.key` as private key
  - kube-controller-manager have `controller-manager.crt` as certificate and `controller-manager.key` as private key  
  - kueb-proxy have `kube-proxy.crt` as certificate and `kube-proxy.key` as private key
- Kube-Apiserver is also client of etcd and kubelet, it can use the original cert and key or genertate new client cert and key
- For ectd `apiserver-etcd-client.crt`,`apiserver-etcd-client.key` 
- For kubelet `apiserver-kubelet-client.crt`and`apiserver-kubelet-client.key`
- Certificate Authority have a separte certficate `ca.crt` and private key `ca.key`
<br>
- CA
  1. use `openssl genrsa -out ca.key 2048` to create Certificate Authority Key
  2. use `openssl req -new -key ca.key -subj "/CN=Kubernetes-CA" -out ca.csr ` to generate a certificate signing request
  3. use `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt` to sign the certificate 
  4. CA key pair are now being used to sign other keys
  5. use `openssl x509 -in {cert} -text -noout` to view a certificate detail
- Admin
  1. use `openssl genrsa -out admin.key 2048` to create a private key for Admin user
  2. use `openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters -out admin.csr` to create a certificcate signing request
  3. use `openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt` to sign the certificate with CA key pair
  4. To identify a amdin account as admin user, GROUP: SYSTEM:MASTERS or SYSTEM:KUBERNETES add `/O=system:masters` to step 2
- Other Client Certificates
  - follow the same step as admin user
  - Kube-scheduler, group must be SYSTEM:KUEB-SCHEDULER
  - Kube-controller-manager, group must be SYSTEM:KUBE-CONTROLLER-MANAGER
  - Kube-proxy, group must be SYSTEM:KUBE-PROXY
- Certificate and key can be used to make API call `curl ... --key admin.key --cert admin.crt --cacert ca.crt`
- Alternative way is to store the certificate and key in kube config file, which will be used to access cluster
- ca.crt is used by all components (client/server) to confirm each other's identity
- ETCD
  1. if ETCD is deployed in a cluster, peer certificate are required secure communication between member of the etcd cluster
  2. ETCD Configuration
    ```
      --key-file=/path-to-certs/etcdserver.key
      --cert-file=/path-to-certs/etcdserver.crt
      --peer-cert-file=/path-to-certs/etcdpeer1.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    ```
- Kube-Apiserver
  - Have many alias which all need to be inclued in certificate signing process
  1. use `openssl genrsa -out apiserver.key 2048`
  2. have a openssl.cnf file with alias and Service ip and host ip
    ```
      [req]
      req_extensions = v3_req
      distinguished_name = req_distinguished_name
      [req_distinguished_name]
      [ v3_req ]
      basicConstraints = CA:FALSE
      keyUsage = nonRepudiation, digitalSignature, keyEncipherment
      subjectAltName = @alt_names
      [alt_names]
      DNS.1 = kubernetes
      DNS.2 = kubernetes.default
      DNS.3 = kubernetes.default.svc
      DNS.4 = kubernetes.default.svc.cluster.local
      IP.1 = ${K8S_SERVICE_IP}
      IP.2 = ${MASTER_HOST}
    ```
  3. run `openssl req -new -key admin.key -subj "/CN=kube-apiserver -out apiserver.csr -config openssl.cnf` to generate CSR with conf file
  4. run `openssl x509 -req -in apiserver.csr -CA ca.crt -CAKey ca.key -out apiserver.crt`
  5. configuration for kube-apiserver
    ```
      --etcd-cafile=/var/lib/kubernetes/ca.pem 
      --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt 
      --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key 
      --etcd-servers=${ETCD_ENDPOINTS} 
      --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem 
      --kubelet-client-certificate=/var/lib/kubernetes/apiserver-kubelet-client.crt 
      --kubelet-client-key=/var/lib/kubernetes/apiserver-kubelet-client.key 
      --client-ca-file=/var/lib/kubernetes/ca.pem 
      --tls-cert-file=/var/lib/kubernetes/apiserver.crt
      --tls-private-key-file=/var/lib/kubernetes/apiserver.key
    ```
- Kubelet
  - Have many nodes, create one for each node and use them in kubelet-config.yaml in each node
  - Example
    ```
      apiVersion: kubelet.config.k8s.io/v1beta1
      kind: KubeletConfiguration
      authentication:
        x509:
          clientCAFile: "/var/lib/kubernetes/ca.pem"
      tlsCertFile: "/var/lib/kubelet/kubelet-node01.crt"
      tlsPrivateKeyFile: "/var/lib/kubelet/kubelet-node01.key"
    ```
  - There are also client certificate for kubeapi to talk to the kube-apiserver with group Name of system:node:{node_name}
  - Kubelet also need to be configured with group system:nodes
- Troubleshooting 
  1. use `journalctl -u {service_name} -l` to view the service log if deployed hard way
  2. use `kubectl log {pod}` to see the log for that pod
  3. if kube-apiserver is down, use docker ps -a for find the container and use `docker log {containe_id}`
<br>
- Certificate Signing API allows user to create a certificate signing request object, which can be reviewed and approved
- CSR can be defined as yaml, with kind:CertificateSigningRequest and request field with base64 encoded CSR 
- run `kubectl get csr` to view all current Certificate Signing Reqeuest
- run `kubectl certificate approve {csr_name}` to approve a Certificate Signing Reqeuest, Only by admin
- certificate can be view by `kubectl get csr {csr_name} -o yaml, base64 encoded certificate can be found in yaml file
- Controller-Manger have 2 field that is responsible for certificate signing request
  1. --cluster-signing-cert-file=/etc/kubernetes/pki/ca.cert
  2. --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
<br>
- Kube Config file will be store in $HOME/.kube/config by default, then we don't to pass in variables when using kubectl
- Kubeconfig contains contexts,clusters,users
  - Cluseter are varies kubernetes cluster the users can access
  - Users are accounts with access to these clusters
  - Context defines which user are used to access which clusters
- Cluster contains the server connection address and the certificate of the certificate authority
- Users contains the client certificate and client key, client cert signed by trusted CA proves who you are
- Context contain the reference(name) of cluster and user, links cluster and user
- Context can also have namespace field defined which will act as default namespace when switching context
- current-context field in Kube Config determines which context to use by default
- run `kubectl config view` to view the kubeconfig file
- to view custom config file `kubectl config view --kubeconfig=my-custom-config`
- to change the context `kubectl config use-context {context}` to change the current-context
- use full path of certificate in the kubeconfig file 
- certificate-authority can be replaced by `certificate-authority-data: {base64_crt_file`
<br>
- Kubernetes have multiple api path that are responsible for cluster functionality. namely /api and /apis
- When using curl to request api, we need to pass in client-key,crt and ca.crt
- alternatily we can use `kubectl proxy` that start a proxy on localhost:8001 and use current-context from kubeconfig 
<br>
- Kubernetes have 4 Authorization Mechanism:Node,ABAC,RBAC,Webhook
  - Node Authorizer get kubelet privilege to read services,endpoint,nodes and pos, write Node Status,Pod Status,Events
  - Kubelet having group tag of system:node:{}, anything with this groups is authorized by node authorizer
  - ABAC: Associate user/group to a set of permissions, attributes are defined in policy files(JSON)
  - Attribute Based Access Control requires manual editing when performing changes and restart kube-apiserver
  - RBAC: Create Role with permissions and assoicate user/group to that role
  - Role Based Access Controle, when role permission is changed, effects user/group immediately
  - Webhook: Manage Authorization Externally via api calls to 3rd party service like Open Policy Agent
  - Always Allow and Always Deny are two additional Authorization Mode which is suggested by their name
  - AM are argument when run kube-apiserver`--authorization-mode=Node,RBAC`, by default AlwaysAllow, can be multiple Value Separated by Commad
  - The Authorization are handled in the order specified in authorization modem, in previous example Node->RBAC
<br>
- to use RBAC, we need to create a role object, each rule contain apiGroups, resources and verbs
- Additional resourceNames can be added to a rules to give fine grained permission of resource access
```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
    namespace: default
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]   #Allow get pods actions
  - apiGroups: [""]
    resources: ["ConfigMap"]   
    verbs: ["Create"]       #Allow Config MapCreation
```
- Then we link user to a Role with RoleBinding, both role and rolebind are namespace scoped
```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: devuser-develeoper-role-binding
    namespace: default
  subjects:
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role #this must be Role or ClusterRole
    name: developer
    apiGroup: rbac.authorization.k8s.io
```
- get role `kubectl get roles` , use `kubectl describe role {role_name}` to get detailed information
- get role-binding `kubectl get rolebindings`, use `kubectl describe rolebinding {role_binding}` to get detailed information
- to Check access, use `kubectl auth can-i {action}`, action can be create deployment, delete nodes
- to Check access as someone else with `kubectl auth can-i {action} -as {user} --namespace {namespace}`
<br>
- Roles only work on namespace scoped resources and apigroups
- Cluster Scoped Resource: 1.Nodes 2.PV 3.clusterroles 4.clusterrolebindings 5.certificatesigningrequests 6.namespace
- to see namespace scope resources `kubectl api-resources --namespaced=true` 
- to see cluster scoped resources `kubectl api-resources --namespaced=false`
- ClusterRole are for Cluster Scoped Resources, i.e perform actions on a node
- When giveing access to namespace scoped resources in ClusterRole, the access is granted in **all namespaces**
```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    # "namespace" omitted since ClusterRoles are not namespaced
    name: secret-reader
  rules:
  - apiGroups: [""]
    #
    # at the HTTP level, the name of the resource for accessing Secret
    # objects is "secrets"
    resources: ["secrets"]
    verbs: ["get", "watch", "list"]
```
- ClusterRole binding binds user to a cluster role
```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: secret-reader-role-binding
  subjects:
  - kind: Group
    name: manager # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: secret-reader
    apiGroup: rbac.authorization.k8s.io
```
- Cluster role defined for namespace level gives access accross all namespaces
<br>
- Service Account are used by other services to interact with the kuberntest cluster, i.e Prometheus pull k8s api for metrics
- To Create Service Account `kubectl create serviceaccount {serviceaccount_name}`
- To View Service Account `kubectl get serviceaccount` or `kubectl get sa`, use `kubectl describe sa {sa_name}` to view details
- Service Account will create a token , which is used external applicatio to authenticate to kubernetes api
- Token generated by service account is store as a secret object and linked to the service account
- to view the token , `kubectl describe secret {token_id}`
- if service calling k8s api is deployed in the k8s cluster, service token secret can be mounted as a volume
- Every Namespace has its own default service account, when pod created, token is automatically mounted as a volume in the pod
- To disable automatic token mounting, set `automountServiceAccountToken` to **false** under spec field of the Pod
- To use a different service account, specify `serviceAccountName` variable under spec field of the Pod, must use replace --force 
- TokenRequestAPI released in K8S 1.22 , Token generate are audience bound,time bound and object bound
- Token Created via TokenRequestAPI when Pod is created, and mounted to Pod as a projected volume
- In 1.24, Creating Service account no longer generates a secret, run `kubectl create token {sa_name}` to create a token bound to SA
<br>
- image is abstracted, like nginx is library/nginx, library is default account for offical Docker images, default registry is Docker Hub
- Most cloud provider like AWS, GCP, Azure provides private registers
- To use private registry, we have to login with credentials, when using a container runtime
- We create a secret in K8s that stores docker-registry that is built to store docker credentials
```
  kubectl create secret docker-registry regcred 
  --docker-server=<your-registry-server> 
  --docker-username=<your-name> 
  --docker-password=<your-password> 
  --docker-email=<your-email>
```
- Then we specify `imagePullSecrets:` lists under the container with the secret we just created
<br>
- All Container in Docker are separted by namespaces, uses the same kernal as the Host
- From Host prespective, all processs on Host are visiable as processes on the system
- By default Docker runs processes in host as the root user, to change attach --user=user_id in docker run
- User to executeing the process can also be defined in Dockerfile using `USER {user_id}`
- `/usr/include/linux/capability.h` contains the capability of what root user can perform
- use --cap-add {capability} in docker run to add root capabilities
- use --cap-drop {capability} in docker run to remove root user capabilities
- use --privileged in docker run command to enable all capabilities
- `securityContext` under spec sets the context for pod level
- `securityContext` under container item is defined for continaer
- `runAsUser: {user_id}` defines which user for Pod or Container
- To added Capability to container define a capabilities section under securityContext
``` 
  securityContext:
    runAsUser: 1000
    capabilities: 
      add: ["MAC_ADMIN"]
```
- Capabilites can only be defined on container level, not on Pod level
<br>
- By Default K8s allows traffic from any pod to any other pod or serives with in the cluster
- Network policy is another object in the namespace, can be linked to pods which have different rules that controls network
- Network policy are linked to a pod using selector and labels, Policy type can be Ingress or Egress
- From are created with podSelector and ports is used to define which ports can connect or be connected
```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: db-policy
    namespace: default
  spec:
    podSelector:
      matchLabels:
        role: db
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:
      - podSelector:
          matchLabels:
            role: api-pod
        namespaceSelector
          matchLabels:
            name: prod
      ports:
      - protocol: TCP
        port: 3306
      egress:
      - to:
        - ipBlock:
            cider: 192.168.10.1/32
      ports:
        - protocol: TCP
          port: 80

```
- In order for the policy to take effect, we nned to define them in policyTypes
## Storage
- Docker stores all of its data in /var/lib/docker, Images are Layer and each layer only store changes from previous layer
- When running containers , Docker addes a read/write layer for storing data associated with Docker Engine
- docker volume create {volume_id} create a volume that can be mounted to a path
- volume mount, link a volume to a direcotry in docker
- bindmount, link direatory on host to directory on the pod
- to use `docker run --mount {mount} --source={data_on_host} --target={target_folder_docker}
- Container Storage Interface are used to write your own driver to work of field 
<br>
- Volumes are created to persist data in kubernetes
- volumes can have storage being configured in different ways, 
  - hostPath is using direcotry on host node to store data
- then volume can be mounted to a container
- `volumeMounts` under container sepecify volume to mount and container path
```
container:
  ...
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    hostPath:
      path: /data #directory location on host
      type: Directory
```
- Not advised for multi-node cluster as each file on different nodes might be different
- For Example, we can use awsElasticBlockStorage as a volume, list out volume-id and fsType
- Persistent Volume is created as a large pool of storage which user deploying pod can carve out as required
  - Access Mode of persistent volume can be ReadOnlyMany, ReadWriteOnce, ReadWriteMany
  - Capacity is the amount of Storage to be reserved for this volume
  - hostPath: maps the volume to host node on a directory, AWSEBS is also an option
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv
  spec:
    capacity:
      storage: 5Gi
    accessModes:
      - ReadWriteOnce
    awsElasticBlockStorage:
      volumeId: {volume_id}
      fsType: ext4
  ```
- Persistent Volume Claims are objects used by K8S to bind PV to Claims based on request properties set on volume
- Labels and Selecotor can be used to bind to the right volume, Claims and Volume are 1 to 1
```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pv-claim
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 500Mi
```
- to view existing persistent volume claims `kubectl get pvc`
- to delete persistent volume claim use `kubeclt delete pvc {pvc_name}`
- persistentVolumeReclaimPolicy defines what happens if a pv claim is deleted
  - Retain: Persistent Volume remains until it's manually deleted
  - Delete: Persistent Volume is deleted automatically when pvc is deleted
  - Recycle: Data in PV is be scraped before made available to other claims
- to use PVC on Pod defination, use the following defination under Pods
```
volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName:{pvc_name}
```
- The rest remains the same, use volumeMounts to mount the pvc to container
<br>
- To use custom storage like EBS, it mush already exist on AWS, This is Static Provisioning
- With Storage Class, we can define a provisoner for automatically provisioning of storage on the cloud and attach them to pods
- Example AWS provisioner
```
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: ebs-sc
  provisioner: ebs.csi.aws.com
```
- PV and associated storage will be created automatically when StorageClass is Created, PV defination is no longer needed
- To use Storage Class, define the storage class name in spec of PVC `storageClassName: ebs-sc`
## Networking
- none network, container is isolated, not reachable by outside network or to each other `--network none` option of docker run
- host network, no network isolation between host and  container, but can't same port  `--network host` option of docker run
- bridge network, create an internal private network that the host and containers can attache to default `172.17.0.0`
- use `docker network ls` to list out all docker networks
- docker internal runs the `ip link add docker0 type bridge` command to create vitual network interface
- when container is created docker creates a network namespace for that container and connect to bridge 
- the network namespace created by docker starts with b316
- run `ip link` to see interface created by docker on host , run `ip -n {ns} link` to see the network iterface within namespace
- specify port mapping to forward request to container host to the port of the container
- docker uses a create a portforwarind with iptables with --dport and --to-destination field
- to list the rules created by docker run `iptables -nvL -t nat`
<br>
- in kubernetes the process of creating network interface is abstracted, use bridge command to perfrom tasks
- run `bridge add {container_id} {namespace}` to added a network to a pariticular namespace 
- CNI: Container Network Interface defines how programs should be developed to solve networking in container runtime
- Master and Worker Nodes must have atleast one network interface connected to the same network
- Each network interface must have an address configured, host must have unique hostname set and MAC address
- Kubeapi server uses port 6443 on  master node
- kubelet on master and host uses port 10250 
- kubescheduler uses port 10259 on master node
- kube-controller-manager uses port 10257 on master node
- etcd uses port 2379, and etcd with HA needs 2380 for talking with each other
- work node exposes service access on 30000-32767(ephemeral ports)
- Important commands
  - ip link
  - ip addr or ip addr show type {type}
  - ip addr add {ip} dev eth0
  - route
  - ip route
  - ip route add {network} via {ip}
  - cat /proc/sys/net/ipv4/ip_forward
  - arp
  - netstat -plnt
- to find service with their listening port with detailed information use `netstat -npl`
- to find all network connection to serivce with detailed information use `netstat -npa`
<br>
- networking solution is not provided by kubernetes, we have to bring our own
  - Every Pod should have a ip
  - Every Pod should be able to communicate with every other Pod within the same node
  - Every Pod should be abel to communicate with every other Pod on other Node  without NAT
- CNI defines the responsiblity to container runtime
- CNI is configured by the kubelet service
- cni-bin-dir on the kubelet configuration section contains all supported cni plugins as binary, `/opt/cni/bin`
- cni-config-dir on kubelet configuration contains a set of configuration files for cni `/opt/cni/net.d`
- CNI scripts is stored in `/opt/cni/bin/net-script.sh`
<br>
- Weave is a network solution base on cni, when deployed create an agent on each node and have network topology of the entire cluster
- Weave can be deployed with a single command if kubernetes basic is setup. Weave uses DaemonSet that ensure each node have 1 pod
- run `kubectl logs weave-net-42gc weave -n kube-system` to see the log
- [deploying weave](https://github.com/weaveworks/weave/blob/master/site/kubernetes/kube-addon.md#-installation) 
<br>
- CNI providers needs to take care of assigning IP to Pod without duplicate
- A host-local plugin stores free and assigned IP which can be used for ip assignment, DHCP can be use for dynamic assignment
- `ipam` section of net.d configurations can specify which type of plugin, subnet and route to be used
- to check ip allocation, use `kubectl logs {weave_pod_name} weave -n kube-system` to eaxmine ipalloc-range
<br>
- Service are accessible through a single IP regardless of which Node the Pod is on
- ClusterIP create a ip for Pod with same labels that can be accessed internal
- NodePort create a IP for internal access, as well as exposing the service to outside on Node Ip's ephmerual port
- kubelet invokes cni to configure network for a pod.
- kube-proxy watches changes in cluster through kube-apiserver
- Service when created is assigned a IP address from a pre-defined range, kube-proxy create forwarding rules on each node to pods
- kube-proxy can create manages forwarding rules using userspace, ipvs and iptables
- kube-proxy mode can be configured using `--proxy-mode [userspace|ipvs|iptables]` with default of iptables
- kube-apisever have a `--service-cluster-ip-range` field that is used to configure service, default 10.0.0.0/24
- to search for forwarding rules run `iptables -L -t nat|grep {service_name}`
- the logs to kube-proxy contains the entry of new service at `/var/log/kube-proxy.log`
- to see what proxy is configured, use `kubectl logs {kubeproxy_podname} -n kube-system`
<br>
- Kubernetes deploys a built in dns server by default, manual deployment will require a dns server
- when object are in the same namespace, we can refer to them directly i.e web-service
- when object are in different namespace, we need to refer to them using domain+namespace i.e web-service.apps
- All services are grouped in a subdomain `svc` and root domain is by default `cluster.local`
- So full qualified domain name is `web-service.apps.svc.cluster.local` 
- DNS for Pod is replacing dots in pod ip with dash i.e 10.244.2.5 to 10-244-2.5
- and full qualified domain name is `10-244-2-5.apps.pod.cluster.local`
<br>
- CoreDNS are deployed as a deployment with 2 replica for redundency.
- the configuration file are store in `/etc/coredns/Corefile`, kubernetes plugin can have tld of domain set
- any record the coredns can't resolve is forwarded to proxy, which is set to use `/etc/reslov.conf`
- The CoreDns setting are passed into the Pod as a ConfigMap Object which can be manually adjusted by user
- CoreDns creates a service within the cluster, so other service can access it , named `kube-dns` by default
- IP address of the `kube-dns` service is created as the namespace on pod in their /etc/reslov.conf file
- Kubelet is responsible for adding this nameserver to Pod when they are scheduled, which is in config.yaml of kubelet
- reslov.conf also have a search entry for every level of domain, so service can be access without giving full qualified domain name
<br>
- Ingress allows the use to access application with single externally accessiable url like a Layer 7 Load Balancer
- Ingress can be used to route to different services base on the url path and SSL Security
- Ingress need to be expose as a Node Port or Cloud Native Load Balancer
- To implement, we deploy a reverse proxy service as a ingress controller
- Then we define ingress resources with defination files
- Example Ingress Controllers include GCE, Nginx, Contour, HAProxy, Traefix, Istio
- To configure create a nginx-controller deployment, which is a nginx build with extra intellengence for configuring ingress resource
- nginx controller are stored in `/nginx-ingress-controller` which need to be passed in as arguments to the image
- a config map object is also required to be passed in, which makes it easier to modify configuration setting
- two environment variable with the pod's name and namespace is also require with the fieldPath of metadata.name and metadata.namespace
- the ports used by ingress controller are 80/http and 443/https
- A Node Port Service needs to be creatd for exposing the nginx service to outside, with label selector linking to nginx deployment
- A Service Account needs to be created with correct Role and Role Binding for accessing ingress resources
- Ingress Resource can forward traffic to an app and different apps base on domain and path of the url
```
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: ingress-wear
  spec:
    rules:
    - host: www.mystore.com
      http:
        paths:
        - path: /wear
          backend:
            serviceName: wear-service
            servicePort: 80
        - path: /warch
          backend: 
            serviceName: watch-service
            servicePort: 80
    - host: new.mystore.com #for domain base routing
      ...
```
- backend section defines where the traffic will be routed to
- use `kubectl create -f ingress-wear.yaml` to create ingress
- use `kubectl get ingress` to see ingress in the namespace/cluster
- Rules are used for routing to different services base on different conditions
- Rules are bond to domain name and they can be configured to route based on path
- if host file is not specified, it will be accepted as wildcard, and every incoming traffic will be handle by that rule
- rewrite rule replaces path with the path defined in anonation `nginx.ingress.kubernetes.io/rewrite-target: {/}` 
## Cluster Design
- Base on different purpose cluster can be designed
  1. Education: Minikube or Kubeadm, AWS/GCP
  2. Dev: Multi-Node Cluster with single Master, Kubeadm on Cloud, EKS,GKE,AKS
  3. Production: HA MultiNode Cluster with multi Master, Kubeadm,Kops on AWS
    - 5000 Node on a cluster
    - Upto 150000 Pod
    - Upto 300000 Total Container
    - Upto 100 Pod per Node
- For On Prem Deployment use kubeadm, GKE for GCP, Kops for AWS, AKS for Azure
- Storage
  1. High Performance: SSD backed storage
  2. Multiple Concurrent Connection： Network backed storage
  3. Persistent shared volume: for shared access accross multiple Pod
  4. Label Node with specific disk types
  5. Use Node Selector to assign application to node with specific disk type
- Avoid hosting workload on Master, Kubeadm acheives that by adding taint
- For Large Cluster, we can separate out ETCD for the Master to its own cluster node
<br>
- Minikube deploys on a Virtual Machine and it's only able to create single node cluster
- Kubeamd expecte VM to be ready and can create single or multi Node cluseter
- Turnkey Solutions uses Provisioned VM and execute script to create cluster, we are responsible for Vm provision and maintainance
  - Openshift: On-Prem Kubernetes Platform created by Red Hat, Open Source Container Application Platform built on top of K8S
  - Cloud Foundary Container Runtime: Helps manage HA K8s Clusters
  - Vmware Cloud PKS: uses existing VMware environment to host K8s Cluster
  - Vagrant: Provides scripts to deploy to different CSP
- Hosted Solutions are managed solution KAAS, VM are deployed provider and K8s are configure and maintained by provider
  - Google Container Engine(GKE)
  - Openshift Online
  - Azure Kubernetes Service (AKS)
  - Amazon Elastic Container Service for Kubernetes(EKS)
<br>
- Multi Master Node are require for HA purpose, which give redundency to every component of the cluster
- Kube-Apiserver handles request 1 at a time, Active-Active, have a load balancer for multiple kubeapiserver
- ControllerManager and Scheduler runs in Active-Standby Mode, with the elected leader handling the workload 
- the first Controller Manger to update the Endpoint is the active then the other become standby
```
  kube-controller-manager --leader-elect true [other option]\ #trys to gain lock on endpoint
  --leader-elect-lease-duration 15s #how much time does the leader holds the lock
  --leader-elect-renew-deadline 10s # how much time does the elected leader tries to renew
  --leader-elect-retry-period 2s #time that the process will want to become leader
```
- ETCD: Stacked, deployed in the master node, but the instance is lost when master node goes down, no redundency
- ETCD: External, Deployed separtely from master, require more server, harder to setup, master don't effect etcd
- kube-apiserver is the only client of etcd server, and etcd connection details are in configuration of apiserver
- `--etcd-server=addr1,addr2` is a list of etcd server addresses that are separated by comma
- Etcd is distributed, meaning having data store accross multiple servers, consistent data is available on all instances
- When writing to etcd, only the elected leader can write, others are followers which forward write internally to leader instance\
- The Leader Elect make sure that the data are consistent accross all instance
- Election is done by generate a random timer,  the first finish count down send request to be leader to other instances
- If for a certain period of time, the instance does not get reponse from leader, the reelection process begins
- pass in all the cluster information in `--initial-cluster peer1={}},pee2={}` during cluster creation
## Troubleshooting
- For application failure, break the problem down layer by layer
- check nodePort with curl `curl http://web-service-ip:{port}`
- check if the service discovers endpoints, the `endpoints` field on detailed svc with k describe
- check if the selector of service matchs the selector of pod or deployment
- chekc if the Pod is in a running state, use `kubectl logs {pod}` to inspect pod not ready
<br>
- For controll plane failure, check the status of the nodes, and status of pod running on cluster
- Check all Pods in the kube-system namespace or check the status of the service(kube-apiserver,controller-master,kube-scheduler)
- Check the kubelet service and kube-proxy serive on worker node
- check the logs of control plane component with `kubectl logs {pod}`
- check the service logs if it's not using deployed with kubeadm, `sudo journalctl -u {service}`
<br>
- For Worker Nodes failure, check the status of nodes and use `kubectl describe node {node}` to check detail of nodes
- Node contains conditions on why the node might have failed,can be in state of True|False|Unkown
  - OutOfDisk, True when node is out of space
  - MemoryPressure, True when out of memory
  - DiskPressure, True when disk capacity is low
  - PIDPressure, True when there are too many process running
  - Ready, True when node is overall healthy
- When the Node stops communicating with the master, status of Node is set to unknown
- Check the last heartbeat time on node status, to figure out when the node crashed
- check for CPU,Memory,Disk Space on node with `top` and `df -h` 
- Check for status of kubelet `service kubelet service`
- Check for kubelet logs for potiential issue `sudo journalctl -u kubelet`
- Check Certificates with `openssl x509 -in /var/lib/kubelet/{node.crt} -text -noout`
## Kubectl Advanced Usecases
- Json Path requirements
  1. Identify the command that will give info in raw format
  2. Familiarize with Json format, use `-o json`
  3. Form the Json Path Query `$.items[0].spec`
  4. use the JSON Path query we developed with the same kubectl command `kubectl ... -o=jsonpath='{json-path-query}'`
- Json Path can be concatnated `o=jsonpath={q1}{q2}...{qn}`
- Formating using {`\n`} for newline,{`\t`} for tab
- loops `'{range .items[*]}{jsonpath_query}{end}'`
- custom-column `-o=custom-columns={column_name}:{jsonpath_query},{col2}:{jpq2},...,{coln}:{jpqn}`
- sorting `--sort-by={properties}`
