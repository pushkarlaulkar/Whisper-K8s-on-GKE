Instructions to deploy **YoPass** on AWS EKS Auto Mode
  1. Deploy EKS cluster through Auto Mode through AWS Console. Add the ` CoreDNS `, ` VPC CNI `, ` Kube Proxy ` add ons.
  2. Create a namespace. ` kubectl create ns yopass `
  3. Deploy the **memcached** & **yopass** deployment & service using the below command

     ```
     kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml
     ```
  4. Deploy **Ingress**, **Ingress Class** & **Ingress Params** which will create an ALB listening on port 443. We will need to provide the arn of the certificate in the **Ingress** object. The certificate for the domain name needs to be created in ACM and DNS validation or Email validation needs to be done prior to creating these resources.
  5. Run the command ` kubectl -n yopass apply -f ingress-all.yml `
  6. Run ` kubectl -n yopass get ingress ` to retrieve the ALB DNS.
  7. Point the domain name in Route 53 to the ALB as an A (alias) record.
  8. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm, perform below steps
  1. Generate a certificate from ACM for your domain name. The certificate arn will be required in the next step since we are running ALB on port 443.
  2. Run the command

     ```
     helm install whisper ./helm --namespace yopass --create-namespace --set certificate_arn=arn_got_from_previous_step
     ```
  3. Get the ALB DNS using ` kubectl -n yopass get ingress ` and point the domain name in Route 53 to the ALB as an A (alias) record.
  4. Access the app using ` https://your_domain_name `.
  5. Uninstall the app using ` helm uninstall whisper --namespace yopass `.

-----------------------------

**ArgoCD** installation
To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Apply ArgoCD manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  3. Create a certificate for the preferred ArgoCD domain name in ACM and put the arn of the certificate in ` argocd-ingress.yml ` & run

     ```
     kubectl -n argocd apply -f argocd-ingress.yml
     ```
  4. Run ` kubectl -n argocd get ingress ` to retrieve the ALB DNS. Point the domain name in Route 53 to the ALB as an A (alias) record.
  5. Access ArgoCD using ` https://argocd_domain_name `.
  6. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
     ```
