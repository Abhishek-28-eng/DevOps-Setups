Step 1 — Create the Argo CD Namespace

Create the namespace where Argo CD will be installed:

kubectl create namespace argocd


This is required because the official Argo CD manifests assume the argocd namespace.

 Step 2 — Apply the Official Argo CD Installation Manifests

Apply the installation manifests directly from the Argo CD repository:

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


This will install all core Argo CD components (API server, controller, repo server, and more) into the argocd namespace.

 Step 3 — Verify Installation

Check that the Argo CD pods are running:

kubectl get pods -n argocd


You should see pods such as:

argocd-server

argocd-repo-server

argocd-application-controller

argocd-dex-server

All of them should be in Running or Completed state when ready.

 Step 4 — Retrieve the Admin Password

Argo CD creates an initial admin user with a generated password stored in a Kubernetes secret.

To get it:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


Save this — you’ll use it to log in for the first time.

 Step 5 — Access the Argo CD UI

By default Argo CD’s UI/Server isn’t exposed externally.

 Option A — Port Forward (Simple for Local Use)

Run this:

kubectl port-forward svc/argocd-server -n argocd 8080:443


Then open in your browser:

https://localhost:8080


Login with:

Username: admin

Password: (retrieved above)

 Option B — Expose via Ingress

Since k3s already has an ingress controller (Traefik):

Create an Ingress that routes to argocd-server, e.g.:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
spec:
  rules:
  - host: argocd.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443


Add a /etc/hosts entry on your machine pointing argocd.local to your node’s IP.

 This lets you access Argo CD via a friendly URL.

 Optional — Change the Admin Password

Once logged in:

Click the user icon in the UI.

Choose Settings → Change Password.

Set a strong password.

Important for production and security.
  
