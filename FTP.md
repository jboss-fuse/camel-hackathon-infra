# Apache FTP Server

Apache FTP Server for file transfer operations with passive mode support.

## Quick Start

### Deploy

```bash
# Via Helm
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm install ftp-server hackathon/ftp -n <namespace>

# Or via direct YAML
oc create -f ftp/ftp-server.yaml -n <namespace>
```

### Connect from Within Cluster

```bash
# From your Camel application or any pod in the cluster
ftp://ftp-server.<namespace>.svc.cluster.local:2121

# Default credentials
Username: test
Password: test
```

### Connect from External Applications

FTP requires port-forwarding for external access (OpenShift Routes don't support FTP protocol):

```bash
# Forward all FTP ports (control + passive mode)
for port in {2121..2130}; do
  oc port-forward -n <namespace> svc/ftp-server $port:$port > /dev/null 2>&1 &
done

# Connect from your local machine
curl -u test:test ftp://localhost:2121/

# Stop port-forwards when done
pkill -f "port-forward.*ftp-server"
```

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| Username | `test` | FTP username |
| Password | `test` | FTP password |
| Control Port | 2121 | FTP control connection port |
| Passive Ports | 2121-2130 | Passive mode data transfer ports |
| Image | quay.io/fuse_qe/apache-ftp:latest | Container image |

### Custom Credentials

```bash
helm install ftp-server hackathon/ftp \
  --set ftp.username=admin \
  --set ftp.password=secret123 \
  -n <namespace>
```

## Usage Examples

### Upload File
```bash
# From within cluster
curl -u test:test -T myfile.txt ftp://ftp-server.<namespace>.svc.cluster.local:2121/

# From local machine (with port-forward)
curl -u test:test -T myfile.txt ftp://localhost:2121/
```

### Download File
```bash
# From within cluster
curl -u test:test ftp://ftp-server.<namespace>.svc.cluster.local:2121/myfile.txt -o myfile.txt

# From local machine (with port-forward)
curl -u test:test ftp://localhost:2121/myfile.txt -o myfile.txt
```

### List Files
```bash
# From within cluster
curl -u test:test ftp://ftp-server.<namespace>.svc.cluster.local:2121/

# From local machine (with port-forward)
curl -u test:test ftp://localhost:2121/
```

## Camel Integration

### Camel Route Example

```java
from("timer:upload?period=60000")
    .setBody(constant("Hello FTP at ${date:now:yyyy-MM-dd HH:mm:ss}"))
    .setHeader(Exchange.FILE_NAME, constant("test-${date:now:yyyyMMdd-HHmmss}.txt"))
    .to("ftp://ftp-server.mynamespace.svc.cluster.local:2121/uploads" +
        "?username=test&password=test&passiveMode=true");

from("ftp://ftp-server.mynamespace.svc.cluster.local:2121/uploads" +
     "?username=test&password=test&passiveMode=true&delete=true")
    .log("Downloaded: ${header.CamelFileName}")
    .to("log:ftp-download");
```
