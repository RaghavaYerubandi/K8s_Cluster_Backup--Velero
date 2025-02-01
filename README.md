# K8s_Cluster_Backup
We will explore how to back up a Kubernetes cluster to an AWS S3 bucket using `Velero`. 🚀
### About:
Velero is a backup and restore solution for Kubernetes clusters. It enables you to back up Kubernetes resources and persistent volumes, and restore them when needed.
### Key Features:
- Backup Kubernetes Resources
  - Saves API objects like Deployments, Services, PVCs, etc.
- Persistent Volume Snapshots 
  - Supports snapshots of PVs in supported storage backends.
- Disaster Recovery 
  - Restores workloads after failures or cluster migrations.
- Namespace or Cluster-Level Backups 
  - You can back up and restore entire clusters or specific namespaces.
- Cloud & On-Prem Storage Support
  - Works with AWS S3, NFS, Ceph and more.

### Installing Velero on Kubernetes:
**Download the file**
~~~bash
wget https://github.com/vmware-tanzu/velero/releases/download/v1.15.0/velero-v1.15.0-linux-amd64.tar.gz
~~~
**Extract the file**
~~~bash
tar -xzvf velero-v1.15.0-linux-amd64.tar.gz
~~~
**Move to `/usr/local/bin`**
~~~bash
mv velero /usr/local/bin/
~~~
**Verify the Version**
~~~bash
velero version
~~~
Below is the desired output for `velero version`
~~~bash
root@ragh-k8s-control-191b22796c9:~# velero version
Client:
        Version: v1.15.0
        Git commit: 1d4f1475975b5107ec35f4d19ff17f7d1fcb3edf
<error getting server version: no matches for kind "ServerStatusRequest" in version "velero.io/v1">
~~~

### Create an S3 Bucket in Aws:
- Create an s3 bucket in AWS
- Create an IAM user to access s3 bucket with s3 bucket permissions.
- Create a Secrets file to store the credentials.

### Creating Secrets:
- Create a Secret for credentials in the `velero` namespace
- Replace Access_id and secret_id with your actual AWS credentials.

~~~bash
kubectl create secret generic velero-aws-credentials \
  --from-literal=aws_access_key_id=<Access_id> \
  --from-literal=aws_secret_access_key=<Secret_id> \
  -n velero
~~~


### Installing Velero
- Use the following configuration to set up Velero.
- The credentials are stored in `awscred_velero`.
~~~bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.5.0 \
  --bucket raghava-s3-bucket \
  --secret-file /awscreds/awscred_velero \
  --backup-location-config region=us-east-2,s3ForcePathStyle="true",s3Url=https://s3.us-east-2.amazonaws.com \
  --snapshot-location-config region=us-east-2
~~~
**Verify the Pods in Velero Namespace**
- Check the running Pods in the velero namespace using the command `kubectl get pods -n velero`.
Below is the Output
~~~bash
root@ragh-k8s-control-191b22796c9:~# ku get pods
NAME                      READY   STATUS    RESTARTS   AGE
velero-67fc997646-jrbwk   1/1     Running   0          107s
~~~

### Taking a Backup with Velero
- Use the below command to take backup for all namespace
~~~bash
velero backup create my-backup --include-namespaces '*'
~~~
- Use the below command to take backup for individual namespace
~~~bash
velero backup create my-backup --include-namespaces <namespace-name>
~~~
- At present, I'm taking a portainer namespace backup.

~~~bash
root@ragh-k8s-control-191b22796c9:~# velero backup create portainer-backup --include-namespaces portainer
Backup request "portainer-backup" submitted successfully.
Run `velero backup describe portainer-backup` or `velero backup logs portainer-backup` for more details.
~~~

### Verify the backups

~~~bash
velero backup get
~~~

### Schedule a backup daily

~~~bash
velero schedule create daily-backup --schedule="0 0 * * *"
~~~

### Restore from backup

~~~bash
velero restore create --from-backup <backup-name>
~~~
or 
~~~bash
velero restore create [RESTORE_NAME] [--from-backup BACKUP_NAME (or) --from-schedule SCHEDULE_NAME] 
~~~

### Points to Remember:
- If the storage is configured with NFS, Velero will not back up NFS-based PVs unless Restic is enabled.
### To enable Restic for NFS backups:
~~~bash
velero install --use-restic
~~~
### Backup with Restic:
~~~bash
velero backup create my-backup --include-namespaces= <namespace-name> --default-volumes-to-restic
~~~
### Restore with Restic:
~~~bash
velero restore create --from-backup my-backup
~~~
