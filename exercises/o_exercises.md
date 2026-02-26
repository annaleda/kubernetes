[bee.yaml](https://github.com/user-attachments/files/25587075/bee.yaml)[bee.yaml](https://github.com/user-attachments/files/25587056/bee.yaml)### Pod Security

Update the pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.
[pod-ubuntu-sleeper.yaml](https://github.com/user-attachments/files/25586999/pod-ubuntu-sleeper.yaml)

Note:
Set the capabilities within the container, not at the pod level, as setting them at the pod level is not supported.
  - Pod Name: ubuntu-sleeper
  - Image Name: ubuntu
  - SecurityContext: Capability SYS_TIME
---

Is run as a root user?

kubectl exec multi-pod -- id

---
Now update the pod to also make use of the NET_ADMIN capability.

Note:
Ensure that you set the capabilities at the container level rather than at the pod level, as configuring them at the pod level is not supported.
  - Pod Name: ubuntu-sleeper
  - Image Name: ubuntu
  - SecurityContext: Capability SYS_TIME
  - SecurityContext: Capability NET_ADMIN

------

### Taints and Tollerations

Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule

kubectl taint nodes node01 spray=mortein:NoSchedule

Create a new pod with the nginx image and pod name as mosquito.

kubectl run mosquito --image=nginx

-----

Create another pod named bee with the nginx image, which has a toleration set to the taint mortein.
  - Image name: nginx
  - Key: spray
  - Value: mortein
  - Effect: NoSchedule
  - Status: Running


controlplane ~ ➜  kubectl describe node node01 | grep Taint
Taints:             spray=mortein:NoSchedule

controlplane ~ ➜  

kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml

vi bee.yaml

kubectl apply -f bee.yaml

Observe that the bee pod has been scheduled on node node01 due to the toleration that has been configured for the pod.

Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
                             spray=mortein:NoSchedule

-------

controlplane ~ ➜  kubectl describe node controlplane | grep Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

controlplane ~ ➜  

kubectl edit node controlplane

apiVersion: v1
kind: Node
metadata:
  annotations:
    flannel.alpha.coreos.com/backend-data: '{"VNI":1,"VtepMAC":"9a:91:d1:27:f8:67"}'
    flannel.alpha.coreos.com/backend-type: vxlan
    flannel.alpha.coreos.com/kube-subnet-manager: "true"
    flannel.alpha.coreos.com/public-ip: 10.244.220.207
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2026-02-26T18:20:51Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: controlplane
    kubernetes.io/os: linux
    node-role.kubernetes.io/control-plane: ""
    node.kubernetes.io/exclude-from-external-load-balancers: ""
  name: controlplane
  resourceVersion: "6077"
  uid: 1f5e40a3-ee3f-4f88-8632-36f8d5cbeb9a
spec:
  podCIDR: 172.17.0.0/24
  podCIDRs:
  - 172.17.0.0/24
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
status:
  addresses:
  - address: 10.244.220.207
    type: InternalIP
  - address: controlplane
    type: Hostname
  allocatable:
    cpu: "16"
    ephemeral-storage: "421832229804"
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 64830160Ki
    pods: "110"
  capacity:
    cpu: "16"
    ephemeral-storage: 457717264Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 64932560Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: "2026-02-26T18:21:04Z"
    lastTransitionTime: "2026-02-26T18:21:04Z"
    message: Flannel is running on this node


-----

apiVersion: v1
kind: Node
metadata:
  annotations:
    flannel.alpha.coreos.com/backend-data: '{"VNI":1,"VtepMAC":"9a:91:d1:27:f8:67"}'
    flannel.alpha.coreos.com/backend-type: vxlan
    flannel.alpha.coreos.com/kube-subnet-manager: "true"
    flannel.alpha.coreos.com/public-ip: 10.244.220.207
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2026-02-26T18:20:51Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: controlplane
    kubernetes.io/os: linux
    node-role.kubernetes.io/control-plane: ""
    node.kubernetes.io/exclude-from-external-load-balancers: ""
  name: controlplane
  resourceVersion: "6077"
  uid: 1f5e40a3-ee3f-4f88-8632-36f8d5cbeb9a
spec:
  podCIDR: 172.17.0.0/24
  podCIDRs:
  - 172.17.0.0/24
status:
  addresses:
  - address: 10.244.220.207
    type: InternalIP
  - address: controlplane
    type: Hostname
  allocatable:
    cpu: "16"
    ephemeral-storage: "421832229804"
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 64830160Ki
    pods: "110"
  capacity:
    cpu: "16"
    ephemeral-storage: 457717264Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 64932560Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: "2026-02-26T18:21:04Z"
    lastTransitionTime: "2026-02-26T18:21:04Z"
    message: Flannel is running on this node
    reason: FlannelIsUp
    status: "False"
    type: NetworkUnavailable
-----------------------
Which node is the POD mosquito on now?
controlplane ~ ➜  kubectl get pods -o wide
NAME         READY   STATUS             RESTARTS      AGE   IP           NODE           NOMINATED NODE   READINESS GATES
bee          1/1     Running            0             14m   172.17.1.2   node01         <none>           <none>
mosquito     1/1     Running            0             35m   172.17.0.4   controlplane   <none>           <none>
pod          0/1     CrashLoopBackOff   2 (19s ago)   38m   172.17.0.5   controlplane   <none>           <none>
replicaset   0/1     CrashLoopBackOff   2 (19s ago)   33m   172.17.0.6   controlplane   <none>           <none>

controlplane ~ ➜  

