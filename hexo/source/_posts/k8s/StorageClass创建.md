---
title: StorageClass创建
date: 2020-04-29 15:45:57
tags: 
- StorageClass
- 存储
categories: k8s
---



# 创建

要使⽤ StorageClass，我们就得安装对应的⾃动配置程序，⽐如我们这⾥存储后端使⽤的是 nfs，那
么我们就需要使⽤到⼀个 nfs-client 的⾃动配置程序，我们也叫它 Provisioner，这个程序使⽤我们已
经配置好的 nfs 服务器，来⾃动创建持久卷，也就是⾃动帮我们创建 PV.

- ⾃动创建的 PV 以 ${namespace}-${pvcName}-${pvName}  这样的命名格式创建在 NFS 服务器上的共享数据⽬录中
- ⽽当这个 PV 被回收后会以 archieved-${namespace}-${pvcName}-${pvName}  这样的命名格式存在NFS 服务器上。

<!--more-->

当然在部署 nfs-client  之前，我们需要先成功安装上 nfs 服务器，服务地址是192.168.220.103，共享数据⽬录是/data/k8s/，然后接下来我们部署 nfs-client 即可，我们也可以直接参考 nfs-client 的⽂档：https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client，进⾏安装即可。

第⼀步：配置 Deployment，将⾥⾯的对应的参数替换成我们⾃⼰的 nfs 配置（nfs-client.yaml）

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-client-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
      - name: nfs-client-provisioner
        image: 192.168.220.101/library/nfs-client-provisioner # quay.io/external_storage/nfs-client-provisioner:latest
        volumeMounts:
          - name: nfs-client-root
            mountPath: /persistentvolumes
        env:
          - name: PROVISIONER_NAME
            value: fuseim.pri/ifs
          - name: NFS_SERVER
            value: 192.168.220.102
          - name: NFS_PATH
            value: /data/k8s
      volumes:
      - name: nfs-client-root
        nfs:
         server: 192.168.220.102
         path: /data/k8s

```

第⼆步：将环境变量 NFS_SERVER 和 NFS_PATH 替换，当然也包括下⾯的 nfs 配置，我们可以看到我们这⾥使⽤了⼀个名为 nfs-client-provisioner 的 serviceAccount  ，所以我们也需要创建⼀个 sa，然后绑定上对应的权限：（nfs-client-sa.yaml）

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
    kind: ClusterRole
    name: nfs-client-provisioner-runner
    apiGroup: rbac.authorization.k8s.io

```

我们这⾥新建的⼀个名为 nfs-client-provisioner 的 ServiceAccount  ，然后绑定了⼀个名为 nfs-client-provisioner-runner 的 ClusterRole  ，⽽该 ClusterRole  声明了⼀些权限，其中就包括对 persistentvolumes  的增、删、改、查等权限，所以我们可以利⽤该 ServiceAccount  来⾃动创建PV。

第三步：nfs-client 的 Deployment 声明完成后，我们就可以来创建⼀个 StorageClass  对象了：（nfs-client-class.yaml）

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false" # When set to "false" your PVs will not be archived
                           # by the provisioner upon deletion of the PVC.

```

我们声明了⼀个名为 course-nfs-storage 的 StorageClass  对象，注意下⾯的 provisioner  对应的值⼀定要和上⾯的 Deployment  下⾯的 PROVISIONER_NAME 这个环境变量的值⼀样。

创建资源

```sh
[15:57:06 root@k8s-01 /opt/k8s/work/StorageClass]$kubectl get pod | grep nfs
nfs-client-provisioner-6dd658dd67-tgmwr   1/1     Running   1          43m


[15:57:08 root@k8s-01 /opt/k8s/work/StorageClass]$kubectl get storageclass
NAME                            PROVISIONER      AGE
managed-nfs-storage (default)   fuseim.pri/ifs   33m
```

# 测试

上⾯把 StorageClass  资源对象创建成功了，接下来我们来通过⼀个示例测试下动态 PV，⾸先创建⼀个 PVC 对象：(test-pvc.yaml)

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

我们这⾥声明了⼀个 PVC 对象，采⽤ ReadWriteMany 的访问模式，请求 1Mi 的空间，但是我们可以看到上⾯的 PVC ⽂件我们没有标识出任何和 StorageClass 相关联的信息，那么如果我们现在直接创建这个 PVC 对象能够⾃动绑定上合适的 PV 对象吗？显然是不能的(前提是没有合适的 PV)，我们这⾥有两种⽅法可以来利⽤上⾯我们创建的 StorageClass 对象来⾃动帮我们创建⼀个合适的 PV:

- 第⼀种⽅法：在这个 PVC 对象中添加⼀个声明 StorageClass 对象的标识，这⾥我们可以利⽤⼀个 annotations 属性来标识，如下：

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

- 第⼆种⽅法：我们可以设置这个 course-nfs-storage 的 StorageClass 为 Kubernetes 的默认存储后端，我们可以⽤ kubectl patch 命令来更新：

```sh
$ kubectl patch storageclass course-nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

上⾯这两种⽅法都是可以的，当然为了不影响系统的默认⾏为，我们这⾥还是采⽤第⼀种⽅法，直接创建即可：我们可以看到⼀个名为 test-pvc 的 PVC 对象创建成功了，状态已经是 Bound 了，是不是也产⽣了⼀个对应的 VOLUME 对象，最重要的⼀栏是 STORAGECLASS，现在是不是也有值了，就是我们刚刚创建的 StorageClass 对象 course-nfs-storage。

接下来我们还是⽤⼀个简单的示例来测试下我们上⾯⽤ StorageClass ⽅式声明的 PVC 对象吧：(test-pod.yaml)

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
  - name: nfs-pvc
    persistentVolumeClaim:
      claimName: test-pvc

```

然后我们可以在 nfs 服务器的共享数据⽬录下⾯查看下数据：

```sh
$ ls /data/k8s/
default-test-pvc-pvc-73b5ffd2-8b4b-11e8-b585-525400db4df7
```

我们可以看到下⾯有名字很⻓的⽂件夹，这个⽂件夹的命名⽅式是不是和我们上⾯的规则：${namespace}-${pvcName}-${pvName}是⼀样的，再看下这个⽂件夹下⾯是否有其他⽂件：

```sh
$ ls /data/k8s/default-test-pvc-pvc-73b5ffd2-8b4b-11e8-b585-525400db4df7
SUCCESS
```

