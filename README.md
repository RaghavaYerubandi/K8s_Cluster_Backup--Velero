# K8s_Cluster_Backup

### Install Velero on K8s Cluster
- Download the file to `/usr/local/bin`
~~~bash
wget https://github.com/vmware-tanzu/velero/releases/download/v1.15.0/velero-v1.15.0-linux-amd64.tar.gz
~~~
**Extract the file**
~~~bash
tar -xzvf velero-v1.15.0-linux-amd64.tar.gz
~~~
**Verify the Version**
~~~bash
root@ragh-k8s-control-191b22796c9:~# velero version
Client:
        Version: v1.15.0
        Git commit: 1d4f1475975b5107ec35f4d19ff17f7d1fcb3edf
<error getting server version: no matches for kind "ServerStatusRequest" in version "velero.io/v1">
~~~
### Create an S3 Bucket in Aws.
- Create an s3 bucket in AWS
- Create an IAM user to access s3 bucket with s3 bucket permissions.
- Create a file to store the credentials, here `awscred_velero`.


### Installing Velero
- Use the below configuration to setup velero
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
