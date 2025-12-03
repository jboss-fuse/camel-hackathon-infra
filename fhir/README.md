# camel-hackathon-infra - FHIR Chapter

This directory contains Kubernetes manifests for deploying HAPI FHIR server instances with different configurations.

## Available Deployments

### 1. FHIR DSTU3 with Authentication
- **File**: `fhir-dstu3-auth.yaml`
- **FHIR Version**: DSTU3
- **Authentication**: Enabled
- **Default Credentials**: admin/admin
- **Service Name**: `fhir-dstu3-auth`

### 2. FHIR DSTU3 without Authentication
- **File**: `fhir-dstu3-noauth.yaml`
- **FHIR Version**: DSTU3
- **Authentication**: Disabled
- **Service Name**: `fhir-dstu3-noauth`

### 3. FHIR R4 Default (without Authentication)
- **File**: `fhir-r4-default.yaml`
- **FHIR Version**: R4
- **Authentication**: Disabled
- **Service Name**: `fhir-r4-default`

## Deployment Instructions

### Deploy All Instances

```bash
oc apply -f fhir/
```

### Deploy Individual Instances

```bash
# DSTU3 with auth
oc create -f fhir/fhir-dstu3-auth.yaml

# DSTU3 without auth
oc create -f fhir/fhir-dstu3-noauth.yaml

# R4 default
oc create -f fhir/fhir-r4-default.yaml
```

### Access the Services

Each deployment creates:
- A **ClusterIP Service** for internal cluster access
- An **OpenShift Route** for external HTTPS access with automatic TLS termination

The routes are automatically created when you apply the manifests.

#### Get Route URLs

```bash
# Get all route URLs
oc get routes

# Get specific route URL
oc get route fhir-dstu3-auth -o jsonpath='{.spec.host}'
oc get route fhir-dstu3-noauth -o jsonpath='{.spec.host}'
oc get route fhir-r4-default -o jsonpath='{.spec.host}'
```

#### Alternative: Port-forward for Local Access

If you prefer to access services locally without using routes:

```bash
# Port-forward to access locally
oc port-forward service/fhir-dstu3-auth 8080:8080
oc port-forward service/fhir-dstu3-noauth 8081:8080
oc port-forward service/fhir-r4-default 8082:8080
```

### Test the Deployments

```bash
# Get the route URLs first
FHIR_DSTU3_AUTH=$(oc get route fhir-dstu3-auth -o jsonpath='{.spec.host}')
FHIR_DSTU3_NOAUTH=$(oc get route fhir-dstu3-noauth -o jsonpath='{.spec.host}')
FHIR_R4_DEFAULT=$(oc get route fhir-r4-default -o jsonpath='{.spec.host}')

# Test FHIR metadata endpoint (no auth)
curl https://$FHIR_DSTU3_NOAUTH/fhir/metadata

# Test with authentication
curl -u admin:admin https://$FHIR_DSTU3_AUTH/fhir/metadata

# Test R4 instance
curl https://$FHIR_R4_DEFAULT/fhir/metadata

# Or test from within the cluster using service names
oc run curl-test --image=curlimages/curl -i --rm --restart=Never -- \
  curl http://fhir-dstu3-noauth:8080/fhir/metadata
```

## Configuration

### Changing Credentials

To change the default credentials for the authenticated instance, edit the environment variables in `fhir-dstu3-auth.yaml`:

```yaml
env:
  - name: AUTH_USERNAME
    value: "your-username"
  - name: AUTH_PASSWORD
    value: "your-password"
```

For production, use Kubernetes Secrets:

```bash
# Create secret
oc create secret generic fhir-auth-credentials \
  --from-literal=username=admin \
  --from-literal=password=secure-password

# Update deployment to use secret (modify the yaml)
env:
  - name: AUTH_USERNAME
    valueFrom:
      secretKeyRef:
        name: fhir-auth-credentials
        key: username
  - name: AUTH_PASSWORD
    valueFrom:
      secretKeyRef:
        name: fhir-auth-credentials
        key: password
```

### Resource Limits

Default resource limits are configured as:
- **Requests**: 512Mi memory, 250m CPU
- **Limits**: 2Gi memory, 1000m CPU

Adjust these based on your workload requirements.

### FHIR Versions

The HAPI FHIR server supports multiple versions. Common values:
- `DSTU3` - FHIR DSTU3
- `R4` - FHIR R4
- `R5` - FHIR R5

See [HAPI FHIR Version Enum](https://github.com/hapifhir/hapi-fhir/blob/master/hapi-fhir-base/src/main/java/ca/uhn/fhir/context/FhirVersionEnum.java) for all supported versions.

## Cleanup

```bash
# Delete all FHIR deployments
oc delete -f fhir/

# Or delete individually
oc delete -f fhir/fhir-dstu3-auth.yaml
oc delete -f fhir/fhir-dstu3-noauth.yaml
oc delete -f fhir/fhir-r4-default.yaml
```

## Image Information

- **Image**: `quay.io/fuse_qe/hapi-fhir:v8.2.0`
- **Port**: 8080
- **Base Path**: `/fhir`

## Health Checks

All deployments include:
- **Liveness Probe**: Checks `/fhir/metadata` every 10s after 60s initial delay
- **Readiness Probe**: Checks `/fhir/metadata` every 5s after 30s initial delay

The startup may take some time as the container logs indicate it needs to complete bulk job initialization.
