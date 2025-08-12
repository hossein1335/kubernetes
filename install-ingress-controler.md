# NGINX Ingress Controller Setup Guide

This guide provides instructions for cloning the NGINX Ingress Controller repository and setting up role-based access control (RBAC) for deployment. It assumes you are using the latest release unless specified otherwise.

## Clone the Repository

1. Clone the NGINX Ingress Controller repository using the following command, replacing `<version_number>` with the specific release you want to use:

   ```bash
   git clone https://github.com/nginx/kubernetes-ingress.git --branch <version_number>
   ```

   For example, to use version 5.1.0, run:

   ```bash
   git clone https://github.com/nginx/kubernetes-ingress.git --branch v5.1.0
   ```

2. Change the active directory to the cloned repository:

   ```bash
   cd kubernetes-ingress
   ```

## Set Up Role-Based Access Control (RBAC)

1. Create a namespace and a service account:

   ```bash
   kubectl apply -f deployments/common/ns-and-sa.yaml
   ```

2. Create a cluster role and binding for the service account:

   ```bash
   kubectl apply -f deployments/rbac/rbac.yaml
   ```

### Additional RBAC for NGINX App Protect or NGINX App Protect DoS

If you plan to use NGINX App Protect or NGINX App Protect DoS, apply the following additional roles and bindings:

- **NGINX App Protect**: Create the App Protect role and binding:

   ```bash
   kubectl apply -f deployments/rbac/ap-rbac.yaml
   ```

- **NGINX App Protect DoS**: Create the App Protect DoS role and binding:

   ```bash
   kubectl apply -f deployments/rbac/apdos-rbac.yaml
   ```
