

## install nginx-ingress whith helm chart

   ```bash
helm install nginx-ingress oci://ghcr.io/nginx/charts/nginx-ingress \
  --version 2.2.1 \
  --namespace nginx-ingress \
  --create-namespace
   ```

 
