# Kubernetes Secret for (Harbor) Registry Authentication

This guide explains how to create a Kubernetes `Secret` of type `kubernetes.io/dockerconfigjson` to authenticate with a Harbor registry.

## Secret YAML Configuration

Below is the YAML configuration for the `Secret`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: harbor-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <BASE64_ENCODED_DOCKERCONFIG>
```

The `<BASE64_ENCODED_DOCKERCONFIG>` placeholder must be replaced with the base64-encoded content of a `.dockerconfigjson` file.

## Steps to Generate the Secret

### 1. Create a `.dockerconfigjson` File

Create a file named `config.json` with the following content, replacing `USERNAME`, `PASSWORD`, and `EMAIL` with your Harbor registry credentials:

```json
{
  "auths": {
    "harbor.devvops.ir": {
      "username": "USERNAME",
      "password": "PASSWORD",
      "email": "EMAIL",
      "auth": "BASE64(username:password)"
    }
  }
}
```

#### Generate the `auth` Field

The `auth` field is the base64-encoded string of `username:password`. For example, if your credentials are:

- **Username**: `admin`
- **Password**: `123456`

Run the following command to generate the `auth` value:

```bash
echo -n 'admin:123456' | base64
```

**Output**: `YWRtaW46MTIzNDU2`

Update the `config.json` file with this value:

```json
{
  "auths": {
    "harbor.devvops.ir": {
      "username": "admin",
      "password": "123456",
      "email": "test@example.com",
      "auth": "YWRtaW46MTIzNDU2"
    }
  }
}
```

### 2. Base64 Encode the `config.json` File

Encode the entire `config.json` file to base64 with the following command:

```bash
cat config.json | base64 -w 0
```

**Example Output**:

```bash
eyJhdXRocyI6eyJoYXJib3IuZGV2dm9wcy5pciI6eyJ1c2VybmFtZSI6ImFkbWluIiwicGFzc3dvcmQiOiIxMjM0NTYiLCJlbWFpbCI6InRlc3RAZXhhbXBsZS5jb20iLCJhdXRoIjoiWVdSbGFXTjZNVEl6TlRJek5EVT0ifX19
```

### 3. Update the Secret YAML

Replace `<BASE64_ENCODED_DOCKERCONFIG>` in the YAML with the base64-encoded output. The final `Secret` YAML will look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: harbor-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJoYXJib3IuZGV2dm9wcy5pciI6eyJ1c2VybmFtZSI6ImFkbWluIiwicGFzc3dvcmQiOiIxMjM0NTYiLCJlbWFpbCI6InRlc3RAZXhhbXBsZS5jb20iLCJhdXRoIjoiWVdSbGFXTjZNVEl6TlRJek5EVT0ifX19
```

### 4. Apply the Secret

Save the YAML content to a file named `harbor-secret.yaml` and apply it to your Kubernetes cluster:

```bash
kubectl apply -f harbor-secret.yaml
```

## Referencing the Secret in a Deployment

To use the `harbor-secret` in a Kubernetes `Deployment`, add the `imagePullSecrets` field under the `spec` section (where `containers` is defined). This allows the Deployment to authenticate with the Harbor registry to pull images.

Here’s an example of a complete `Deployment` YAML that references the `harbor-secret`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rent-fast-api-deployment
  labels:
    app: rent-fast-api
spec:
  replicas: 4
  selector:
    matchLabels:
      app: rent-fast-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: rent-fast-api
    spec:
##add this line in your deployment ##
      imagePullSecrets:
        - name: harbor-secret
####################################
      containers:
        - name: rent-fast-api
          image: registry/rent-fast-api:latest
          ports:
            - containerPort: 8001
          resources:
            limits:
              memory: "2Gi"
              cpu: "2000m"
            requests:
              memory: "1Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /
              port: 8001
            initialDelaySeconds: 50
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /
              port: 8001
            initialDelaySeconds: 60
            periodSeconds: 30
```

### Explanation

- **imagePullSecrets**: This field references the `harbor-secret` created earlier, enabling Kubernetes to use the credentials to pull the `rent-fast-api` image from `harbor.devvops.ir`.
- Save this YAML as `rent-fast-api-deployment.yaml` and apply it with:

```bash
kubectl apply -f rent-fast-api-deployment.yaml
```
