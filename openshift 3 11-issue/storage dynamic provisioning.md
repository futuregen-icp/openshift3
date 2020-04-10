# storage dynamic provisioning을 하지 않는경우

Created: Apr 10, 2020 3:59 PM

### default : pv-registry.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: registry-storage
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: registry-storage
        namespace: default
      nfs:
        path: /ocp/volumes/
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain
    

### default :pvc-registry.yml

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc-registry
      namespace: default
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 500Gi

### pvc  mount

    oc edit deploymentconfig.apps.openshift.io/docker-registry
    ...
    volumes:
          - name: registry-storage
            persistentVolumeClaim:
              claimName: pvc-registry
    ...

### openshift-logging:logging-es-0.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: logging-es-0
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: logging-es-0
        namespace: openshift-logging
      nfs:
        path: /ocp/Logging/logging-es-0
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-logging:logging-es-ops-0.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: logging-es-ops-0
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: logging-es-ops-0
        namespace: openshift-logging
      nfs:
        path: /ocp/Logging/logging-es-ops-0
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-infra:metrics-cassandra-1.yaml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: metrics-cassandra-1
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: metrics-cassandra-1
        namespace: openshift-infra
      nfs:
        path: /ocp/Metrics/metrics-cassandra-1
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-monitoring:prometheus-k8s-db-prometheus-k8s-0.yaml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: prometheus-k8s-db-prometheus-k8s-0
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: prometheus-k8s-db-prometheus-k8s-0
        namespace: openshift-monitoring
      nfs:
        path: /ocp/Metrics/prometheus-k8s-db-prometheus-k8s-0
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-monitoring:prometheus-k8s-db-prometheus-k8s-1.yaml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: prometheus-k8s-db-prometheus-k8s-1
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: prometheus-k8s-db-prometheus-k8s-1
        namespace: openshift-monitoring
      nfs:
        path: /ocp/Metrics/prometheus-k8s-db-prometheus-k8s-1
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-monitoring:alertmanager-main-db-alertmanager-main-0.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: alertmanager-main-db-alertmanager-main-0
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: alertmanager-main-db-alertmanager-main-0
        namespace: openshift-monitoring
      nfs:
        path: /ocp/Metrics/alertmanager-main-db-alertmanager-main-0
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-monitoring:alertmanager-main-db-alertmanager-main-1.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: alertmanager-main-db-alertmanager-main-1
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: alertmanager-main-db-alertmanager-main-1
        namespace: openshift-monitoring
      nfs:
        path: /ocp/Metrics/alertmanager-main-db-alertmanager-main-1
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain

### openshift-monitoring:alertmanager-main-db-alertmanager-main-2.yml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: alertmanager-main-db-alertmanager-main-2
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 500Gi
      claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: alertmanager-main-db-alertmanager-main-2
        namespace: openshift-monitoring
      nfs:
        path: /ocp/Metrics/alertmanager-main-db-alertmanager-main-2
        server: nfs.ocp3-11.fu.te
      persistentVolumeReclaimPolicy: Retain