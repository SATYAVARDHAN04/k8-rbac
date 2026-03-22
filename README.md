## RBAC AND SERVICE ACCOUNT
## INSTALL EKSCTL FOR K8S CLUSTER CREATION

```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```

## INSTALL KUBECTL FOR K8S CLUSTER INTERACTION

```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.3/2025-08-03/bin/linux/amd64/kubectl
```
```bash
chmod +x ./kubectl
```
```bash
sudo mv kubectl /usr/local/bin/kubectl
```

## NOW WE WILL CREATE A MANAGED NODE GROUP or IN SHORT A CLUSTER WITH A MASTER AND WORKER NODES

*NOTE: A MANAGED NODE GROUP IS THE SET OF AWS WORKER NODES(INSTANCES) THAT AWS WILL CREATE AND MANAGE ON ITS OWN*
```bash
eksctl create cluster --config-file=eks.yaml
```
```bash
eksctl delete cluster --config-file=eks.yaml
```


### OIDC PROVIDER

```bash
REGION_CODE=us-east-1
CLUSTER=roboshop-newproject
ID=517695827891
```


```bash
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER \
    --approve
```
*CREATE A SECRET USING AWS SECRET MANAGER WITH name as /roboshop/mysql/password and then CREATE A ROLE IN AWS IAM WITH **GetSecretValue** attaching the arn of the secret created*

```bash
eksctl create iamserviceaccount \
  --cluster=$CLUSTER \
  --namespace=roboshop \
  --name=roboshop-mysql-secret \
  --attach-policy-arn=arn:aws:iam::517695827891:policy/roboshopmysqlsecret \
  --override-existing-serviceaccounts \
  --region=$REGION_CODE \
  --approve
```
```bash
kubectl get sa roboshop-mysql-secret -n roboshop -o yaml
```

```bash
aws secretsmanager get-secret-value \
    --secret-id roboshop/mysql/password
```

```bash
kubectlapply -f sa.yaml -n roboshop
```

```bash
kubectl exec -it sa-reader -n roboshop -- bash
```

```bash
aws secretsmanager get-secret-value --secret-id roboshop/mysql/password --query SecretString --output text
```

```bash
kubectl apply -f initcontainers.yaml -n roboshop
```
