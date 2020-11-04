# Fargate PoC

```sh
eksctl create cluster -f cluster-fargate.yaml --profile saml
```

Now enable OIDC for pod roles

```sh
eksctl utils associate-iam-oidc-provider --cluster fargate-cluster --approve --profile saml --region eu-west-1
```

Now check IAM (providers). And check this for pod roles:
https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/

Get and apply the IAM policy for balancers

```sh
aws iam create-policy --policy-name ALBIngressControllerIAMPolicy --policy-document file://alb-ingress-iam-policy.json --profile saml --region eu-west-1
```

Now apply K8S roles and bindings

```sh
kubectl apply -f rbac-role.yaml
```

Create the IAM Service Account (and then check cloudformation [there is a new
role in place])

```
eksctl create iamserviceaccount \
--name alb-ingress-controller \
--namespace kube-system \
--cluster fargate-cluster \
--attach-policy-arn arn:aws:iam::459181319149:policy/ALBIngressControllerIAMPolicy \
--approve --region eu-west-1 --profile saml
```

`N.B. 459181319149 is the AWS Account Id`

Check also service accounts (there is the alb-ingress-controller service
account)

```sh
kubectl get serviceaccounts -n kube-system
```

Now you can apply the balancer controller (**CHANGE THE YAML**)

```sh
kubectl apply -f alb-ingress-controller.yaml
```

Check for pods in the kube-system

```sh
kubectl get pods -n kube-system
```

Now deploy the application

```sh
kubectl apply -f nginx-deployment.yaml
```

Then deploy the application service

```sh
kubectl apply -f nginx-service.yaml
```

Now deploy the ingress rule (then check balancers)

```sh
kubectl apply -f nginx-ingress.yaml
```

