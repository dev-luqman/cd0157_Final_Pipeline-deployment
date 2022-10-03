# The below commands are use to run the application on aws resourses

## command for creating cluster
eksctl create cluster --name eksctl-demo --nodes=2 --version=1.22 --instance-types=t2.medium --region=us-east-2

# Check the status, Nodes should be ready
kubectl get nodes

## get aws account user ID 
```
aws sts get-caller-identity --query Account --output text
- Returns the AWS account id similar to 
- 519002666132
```

## Update the `trust.json` file with your AWS account id. 
```
  {
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Principal": {
              "AWS": "arn:aws:iam::<ACCOUNT_ID>:root"
          },
          "Action": "sts:AssumeRole"
      }
    ]
  }
```

## Create an IAM role for codebuild
```
aws iam create-role --role-name UdacityFlaskDeployCBKubectlRole --assume-role-policy-document file://trust.json --output text --query 'Role.Arn'
```

## Attach the Policy to the IAM role using the command
```
aws iam put-role-policy --role-name UdacityFlaskDeployCBKubectlRole --policy-name eks-describe --policy-document file://iam-role-policy.json
```

## EKS config file update 
 __First, get the current ConfigMap and save it to a file:__
 ```
 kubectl get -n kube-system configmap/aws-auth -o yaml > /tmp/aws-auth-patch.yml
 ```

## Edit and save eks config file:
code /System/Volumes/Data/private/tmp/aws-auth-patch.yml, _Add the below code block_
```
- groups:
    - system:masters
    rolearn: arn:aws:iam::<ACCOUNT_ID>:role/UdacityFlaskDeployCBKubectlRole
    username: build 
```

### Update your cluster's **configmap**:
```
kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"
```

### Test - To test your application endpoint, get the external IP for your service:
```
kubectl get services simple-flask-deployment -o wide
```

### note- update the cicd-config file with github info as well as instrusted in doc.

