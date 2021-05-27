# setup kafka cluster on EKS  Using Ansible
Follow the steps to setup kafka cluster
- create  k8s ServiceAccount using below code
```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: litmus-runner
    namespace: default
    labels:
      name: litmus-runner
---

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: litmus-runner
  labels:
    name: litmus-runner
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: litmus-runner
  labels:
    name: litmus-runner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: litmus-runner
subjects:
- kind: ServiceAccount
  name: litmus-runner
  namespace: default
  ```
- create a k8s-secret and provide your aws credentials
```yaml
apiVersion: v1
kind: Secret
metadata:
        name: aws-secret
data:
        AWS_ACCESS_KEY_ID: "your base64-encoded access key"   
        AWS_SECRET_ACCESS_KEY: "your base64-encoded secret key"
        AWS_DEFAULT_REGION: "your base64-encoded region"
        EKS_CLUSTER_NAME:  "your base64-encoded cluster name"
        
```
- create a pod  that will setup your kafka cluster on eks\
  **Note:** here  *MODE* and *PLATFORM* are passed as env variable. There can be two values of MODE ,By default it is set to *setup* ,then it will setup the cluster. To cleanup the cluster change the value of MODE to *cleanup*.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  serviceAccountName: litmus-runner
  containers:
  - name: myapp-container
    image: litmuschaos/kafka-deployer:latest
    imagePullPolicy: Always
    envFrom:
        - secretRef:

              name: aws-secret
    env:
        - name: MODE
          value: "setup"
        - name: PLATFORM
          value: "eks"
```
It takes few minutes to setup the kafka cluster


        
