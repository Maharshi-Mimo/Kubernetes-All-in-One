# SSL Certificates in Kubernetes

## Understanding SSL/TLS Certificates
SSL (Secure Sockets Layer) and its successor TLS (Transport Layer Security) certificates are digital certificates that authenticate the identity of a website/service and enable encrypted connections.

### Key Components
- **Public Key**: Shared with clients
- **Private Key**: Kept secure on the server
- **Certificate Authority (CA)**: Trusted third party that signs certificates
- **Digital Signature**: Verifies certificate authenticity

## SSL/TLS in Kubernetes

### 1. Internal Communication
Kubernetes uses certificates for:
- API Server authentication
- Etcd cluster communication
- Kubelet to API Server communication
- Controller Manager to API Server
- Scheduler to API Server

### 2. Common Certificate Types
```yaml
- Server Certificates:
  - kube-apiserver.crt
  - etcd-server.crt
  - kubelet.crt

- Client Certificates:
  - admin.crt
  - kube-scheduler.crt
  - kube-controller-manager.crt

- Root CA Certificates:
  - ca.crt
```

## Practical Examples

### 1. Creating Self-Signed Certificates
```bash
# Generate CA private key
openssl genrsa -out ca.key 2048

# Generate CA certificate
openssl req -x509 -new -nodes -key ca.key -subj "/CN=KUBERNETES-CA" -days 10000 -out ca.crt

# Generate server key
openssl genrsa -out server.key 2048

# Generate server CSR
openssl req -new -key server.key -subj "/CN=kube-apiserver" -out server.csr

# Sign the certificate
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
```

### 2. Configuring TLS in Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

### 3. Creating TLS Secret
```bash
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/key.key
```

## Best Practices

1. **Certificate Rotation**
   - Implement automatic certificate rotation
   - Use short-lived certificates
   - Monitor certificate expiration

2. **Security Measures**
   - Keep private keys secure
   - Use strong encryption (minimum RSA 2048-bit)
   - Regular certificate audits

3. **Tools and Solutions**
   - cert-manager for automated certificate management
   - Let's Encrypt for free TLS certificates
   - HashiCorp Vault for secret management

## Common Issues and Troubleshooting

1. **Certificate Expiration**
   ```bash
   # Check certificate expiration
   openssl x509 -in cert.pem -noout -dates
   ```

2. **Certificate Chain Issues**
   ```bash
   # Verify certificate chain
   openssl verify -CAfile ca.crt server.crt
   ```

3. **Certificate Viewing**
   ```bash
   # View certificate contents
   openssl x509 -in cert.pem -text -noout
   ```

## Cert-Manager Integration

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-cert
spec:
  secretName: example-tls
  duration: 2160h # 90 days
  renewBefore: 360h # 15 days
  subject:
    organizations:
    - Example Corp
  commonName: example.com
  isCA: false
  privateKey:
    algorithm: RSA
    size: 2048
  usages:
    - server auth
    - client auth
  dnsNames:
  - example.com
  - www.example.com
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

## References
- [Kubernetes Documentation - Certificates](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)
- [Cert-Manager Documentation](https://cert-manager.io/docs/)
- [Let's Encrypt](https://letsencrypt.org/docs/)
