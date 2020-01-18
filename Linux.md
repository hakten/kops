
#### Step 1. Create a bastion host or user
* **Option 1:** Create a bastion host and assign a role with administrator access or seperate full accesses on required AWS services
* **Option 2:** Or alternatively, you can create a user with required permission and configure your machine with AWS config with the access and secret key

#### Step 2. Create a publicly hosted zone in Route53
* **Option 1** : If you bought your domain with AWS, then you should already have a hosted zone in Route53. If you plan to use this domain then no more work is needed.
* **Option 2:** In this scenario you want to contain all kubernetes records under a subdomain of a domain you host in Route53. This requires creating a second hosted zone in route53, and then copying the NS servers of your SUBDOMAIN up to the PARENT domain in Route53.

#### Step 3. Create a bucket for state info that kops will be using
```
aws s3 mb s3://"your-s3-bucket" --region us-east-1
```

#### Step 4. Install kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```

#### Step 5. Install kops
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

#### Step 6. Create the cluster
```
kops create cluster "your-domain-name" --state=s3://"your-s3-bucket" --master-count=3 --master-size="t2.micro" --master-zones="eu-west-1a,eu-west-1b,eu-west-1c" --node-count=3 --node-size="t2.micro" --zones="eu-west-1a,eu-west-1b,eu-west-1c" --networking="weave" --topology="private" --ssh-public-key="~/.ssh/id_rsa.pub"
```

#### Step 7. Edit and configure your cluster if needed
```
kops edit cluster "your-domain-name" --state=s3://"your-s3-bucket"
```

#### Step 8. Update the cluster - this will actually create the cluster
```
kops update cluster "your-domain-name" --state=s3://"your-s3-bucket" --yes
