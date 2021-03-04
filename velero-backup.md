Install app velero for creating backups k8s cluster

https://velero.io/


Install velero client.

wget https://github.com/vmware-tanzu/velero/releases/download/v1.4.2/velero-v1.4.2-linux-amd64.tar.gz

tar -xvf velero-v1.4.2-linux-amd64.tar.gz -C /tmp

sudo mv /tmp/velero-v1.4.2-linux-amd64/velero /usr/local/bin

velero version

 

CREATE S3 BUCKET AND IAM ROLE FOR VELERO https://www.eksworkshop.com/intermediate/280_backup-and-restore/prerequisites/
aws iam create-user --user-name velero


cat > velero-policy.json <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVolumes",
                "ec2:DescribeSnapshots",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:CreateSnapshot",
                "ec2:DeleteSnapshot"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObject",
                "s3:AbortMultipartUpload",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": [
                "arn:aws:s3:::${VELERO_BUCKET}/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::${VELERO_BUCKET}"
            ]
        }
    ]
}
EOF
Attach policy to velero IAM User

aws iam put-user-policy \
  --user-name velero \
  --policy-name velero \
  --policy-document file://velero-policy.json
Create an access key for the user:


aws iam create-access-key --user-name velero > velero-access-key.json
Now, let’s set the VELERO_ACCESS_KEY_ID and VELERO_SECRET_ACCESS_KEY environment variables and save them to bash_profile.


export VELERO_ACCESS_KEY_ID=$(cat velero-access-key.json | jq -r '.AccessKey.AccessKeyId')
export VELERO_SECRET_ACCESS_KEY=$(cat velero-access-key.json | jq -r '.AccessKey.SecretAccessKey')
echo "export VELERO_ACCESS_KEY_ID=${VELERO_ACCESS_KEY_ID}" | tee -a ~/.bash_profile
echo "export VELERO_SECRET_ACCESS_KEY=${VELERO_SECRET_ACCESS_KEY}" | tee -a ~/.bash_profile
Create a credentials file (velero-credentials) specfic to velero user in your local directory (~/environment). We will need this file when we install velero on EKS


cat > velero-credentials <<EOF
[default]
aws_access_key_id=$VELERO_ACCESS_KEY_ID
aws_secret_access_key=$VELERO_SECRET_ACCESS_KEY
EOF

Install velero 1.4.2

helm install vmware-tanzu/velero --namespace velero \
--set-file credentials.secretContents.cloud=/home/******/velero \
--set configuration.provider=aws \
--set configuration.backupStorageLocation.name=default \
--set configuration.backupStorageLocation.bucket=eksworkshop-backup-1603191556-19697 \
--set configuration.backupStorageLocation.config.region=eu-central-1 \
--set configuration.volumeSnapshotLocation.name=default \
--set configuration.volumeSnapshotLocation.config.region=eu-central-1 \
--set image.repository=velero/velero \
--set image.tag=v1.4.2 \
--set image.pullPolicy=IfNotPresent \
--set initContainers[0].name=velero-plugin-for-aws \
--set initContainers[0].image=velero/velero-plugin-for-aws:v1.1.0 \
--set initContainers[0].volumeMounts[0].mountPath=/target \
--set initContainers[0].volumeMounts[0].name=plugins \
--generate-name
Create backup https://www.eksworkshop.com/intermediate/280_backup-and-restore/backup-restore/

 velero backup create staging-backup --include-namespaces staging
velero v1.5.1 does not work with aws plugin: velero-plugin-for-aws


RESTORE BACKUP
velero restore create --from-backup <backup-name>


Upgrade Velero server to 1.5.2
https://velero.io/docs/v1.5/upgrade-to-1.5/


velero version
kubectl get deployments.apps -n velero
kubectl set image deployment/velero-1603195718 velero=velero/velero:v1.5.2 -n velero


