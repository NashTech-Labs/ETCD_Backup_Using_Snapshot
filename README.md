## **README: Restoring ETCD Snapshot in Kubernetes Cluster**

### Overview:

This guide outlines the steps to restore an ETCD snapshot in a Kubernetes cluster. ETCD is a distributed key-value store used by Kubernetes to store cluster state and configuration data. Restoring an ETCD snapshot is essential for disaster recovery and data integrity purposes.

### Pre-requisites:

- Access to the Kubernetes cluster.
- Basic understanding of Kubernetes components and ETCD.
- Permissions to modify ETCD pod manifests and restart services.

### Steps:

#### 1. Retrieve Snapshot Information:

- Use the following command to gather necessary information from the ETCD pod:
  ```
  kubectl describe pod etcd-controlplane -n kube-system
  cat /etc/kubernetes/manifests/etcd.yaml
  ```

#### 2. Connect to the ETCD Cluster:

- Set the ETCDCTL_API environment variable to 3:
  ```
  export ETCDCTL_API=3
  ```

- Save the ETCD snapshot:
  ```
  etcdctl snapshot save <SNAPSHOT_FILE_PATH/snapshot.db> \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
  ```

#### 3. Verify Snapshot:

- Check the status of the ETCD snapshot:
  ```
  etcdctl snapshot status <SNAPSHOT_FILE_PATH/snapshot.db>
  ```

#### 4. Stop API Server:

- Stop all API server instances:
  ```
  systemctl stop kube-apiserver
  ```

#### 5. Restore State in ETCD:

- Restore the ETCD snapshot:
  ```
  etcdctl --data-dir <data-dir-location> snapshot restore <SNAPSHOT_FILE_PATH/snapshot.db> \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
  ```

#### 6. Change Ownership:

- Change ownership of the restored data directory:
  ```
  sudo chown -R etcd:etcd <data-dir-location>
  ```

#### 7. Update ETCD Pod Manifest:

- Modify the ETCD pod manifest to use the restored data directory. Edit the manifest file located at "/etc/kubernetes/manifests/etcd.yaml".

#### 8. Restart Services:

- Reload systemd daemon:
  ```
  systemctl daemon-reload
  ```

- Restart ETCD and API server services:
  ```
  service etcd restart
  service kube-apiserver start
  ```

### Conclusion:

Restoring an ETCD snapshot is crucial for maintaining the integrity and availability of your Kubernetes cluster. By following these steps, you can ensure that your cluster data is securely backed up and recoverable in case of emergencies. Always exercise caution and perform these actions during maintenance windows to minimize disruptions to your cluster operations.
