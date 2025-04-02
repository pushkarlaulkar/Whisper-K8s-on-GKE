Instructions to deploy **YoPass** on GCP GKE Auto Pilot cluster
  1. Deploy GKE Auto Pilot cluster GCP Console.
  2. Create a namespace. ` kubectl create ns yopass `
  3. Deploy the **memcached** & **yopass** deployment & service using the below command

     ```
     kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml
     ```
  4. Deploy **Ingress**, **Managed Certificate** & **Frontend Config** which will create an ALB listening on port 443 by running below command. Before running below command, in the **Managed Certificate** put the domain name you need for your application.
     ```
     kubectl -n yopass apply -f ingress-all.yml
     ```
  7. Run ` kubectl -n yopass get ingress ` to retrieve the ALB IP ( Please wait couple of minutes ). Create an A record in **Cloud DNS** or your own DNS service pointing your domain name to this IP.
  8. Once the DNS entry has been added it will take couple of minutes ( can be 60 minutes in some cases ) for the certificate to be generated. Run ` kubectl -n yopass get managedcertificate ` or in GCP console to check for **Active** status.
  9. Access the app using `https://your_domain_name`.

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
