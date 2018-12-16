







NetworkManager.conf:
```
[keyfile]
unmanaged-devices=veth*
```

Local volume storage is broken, fix:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: local-volume-provisioner-config
  namespace:  local-storage
data:
  storageClassMap: |
    local-hdd:
       hostDir: /mnt/local-storage/hdd
       mountDir:  /mnt/local-storage/hdd

piVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  creationTimestamp: 2018-12-15T20:14:15Z
  generation: 1
  labels:
    app: local-volume-provisioner
  name: local-volume-provisioner
  namespace: local-storage
  resourceVersion: "103526"
  selfLink: /apis/extensions/v1beta1/namespaces/local-storage/daemonsets/local-volume-provisioner
  uid: fe817e30-00a5-11e9-8e44-ac1f6b1623be
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: local-volume-provisioner
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: local-volume-provisioner
    spec:
      containers:
      - env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: spec.nodeName
        - name: MY_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        - name: VOLUME_CONFIG_NAME
          value: local-volume-provisioner-config
        - name: JOB_CONTAINER_IMAGE
          value: "quay.io/external_storage/local-volume-provisioner:v2.2.0"
        image: quay.io/external_storage/local-volume-provisioner:v2.2.0
        imagePullPolicy: IfNotPresent
        name: provisioner
        resources: {}
        securityContext:
          runAsUser: 0
          seLinuxOptions:
            level: s0:c0.c1023
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /mnt/local-storage
          name: local-storage
          mountPropagation: "HostToContainer"
        - mountPath: /etc/provisioner/config
          name: provisioner-config
          readOnly: true
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: local-volume-provisioner
      serviceAccountName: local-volume-provisioner
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /mnt/local-storage
          type: ""
        name: local-storage
      - configMap:
          defaultMode: 420
          name: local-volume-provisioner-config
        name: provisioner-config
  templateGeneration: 1
  updateStrategy:
    type: OnDelete
status:
  currentNumberScheduled: 1
  desiredNumberScheduled: 1
  numberAvailable: 1
  numberMisscheduled: 0
  numberReady: 1
  observedGeneration: 1
  updatedNumberScheduled: 1      

```