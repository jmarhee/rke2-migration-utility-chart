# RKE2 Migration Utility

A Helm chart to prepare RKE1 clusters for RKE2 installation.

## Setup

This utility assumes an S3-located RKE snapshot:

```bash
rke etcd snapshot-save --config cluster.yaml --name pre-migrate --s3 --access-key ${S3_ACCESS_KEY} --secret-key ${S3_SECRET_KEY} --bucket-name ${BUCKET} --s3-endpoint s3.yourhost.com
````

Once completed, and confirmed in the S3 bucket, proceed to run the chart.

In `values.yaml`, provide the following values:

```yaml
s3config:
  accessToken: ""
  secretKey: ""
  bucketName: "rke-snapshot"
  endpoint: "s3.amazon.com" # i.e. any S3-compatible endpoint (Minio, etc.)
  snapshot: "pre-migrate" # No file extension
  additional: "" # i.e. "--folder-name someFolder"
```

## Installation

This chart will install a DaemonSet that will complete the migration from the snapshot data from RKE into the anticipated filesystem locations for RKE2.

```bash
helm install rke2-migration-utility ./rke2-migration-utility -f rke2-migration-utility/values.yaml
```



## Post-Installation

Watch the installation of the DaemonSet to ensure all of the node are marked Ready:

```bash
kubectl get ds -n kube-system migration-agent -w
```

Then you can proceed to stop Docker:

```bash
systemctl disable docker
systemctl stop docker
```

then proceed to install RKE2:

```bash
curl -sfL https://get.rke2.io | sh -
```

and start and enable RKE2-server:

```bash
systemctl start rke2-server
systemctl enable rke2-server
```