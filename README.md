# k3s-and-more
```
curl -sfL https://get.k3s.io | sh -




curl -sL https://github.com/cert-manager/cert-manager/releases/download/v1.11.2/cert-manager.yaml > cert-manager.yaml
kubectl apply -f cert-manager.yaml

curl -sL https://github.com/cert-manager/cert-manager/releases/download/v1.11.2/cert-manager.crds.yaml > cert-manager.crds.yaml
kubectl apply -f cert-manager.crds.yaml

--------
letsencrypt-issuer-production.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: bien@bienlab.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: traefik



REDIRECT

cat <<EOF > middleware.yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect
spec:
  redirectScheme:
    scheme: https
    permanent: true
EOF

kubectl -n default apply -f middleware.yaml
kubectl -n default get middleware


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: juiceshop-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
    cert-manager.io/cluster-issuer: letsencrypt-prod
    traefik.ingress.kubernetes.io/router.middlewares: default-redirect@kubernetescrd
spec:
  rules:
  - host: juiceshop.laikacat.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: juiceshop
            port:
              number: 3000
  tls:
  - hosts:
    - juiceshop.laikacat.com
    secretName: juiceshop.laikacat.com-tls
```
