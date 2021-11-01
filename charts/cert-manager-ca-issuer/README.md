# Helm chart - cert-manager-ca-issuer

A Chart to use your own Certificate Authority for signing TLS certificates using
[cert-manager](https://cert-manager.io/).

## Values

| Property | Default | Description |
|----------|---------|-------------|
| `issuerNamespace` | `cert-manager`| The namespace used for deploying the secret contain the CA certificate and key. Also, if the issuer is of the namespaced kind `Issuer`, the `Issuer` instance itself is also deployed here. |
| `issuerKind` | `ClusterIssuer` | The kind of the issuer. Could be `ClusterIssuer` or the namespaced `Issuer` |
| `issuerName`| `ca-issuer` | The name of the issuer which needs to be specified in the cert-manager annotation described below |
| `issuerSecretName` | `k8s-local-ca-tls` | The name of the secret containing the TLS secrets |
| `issuerTLSSecrets` | _none_| **No defaults. Required.** Should be an object with the keys `tls.crt` and `tls.key`|

## Providing the required TLS secret

You can generate the required TLS secrets entry from your existing certificates by running this command and copying the
entries.

```shell-session
$ kubectl create secret tls k8s-local-ca-tls \
          --namespace cert-manager \
          --cert k8s_local_ca.chained.pem --key k8s_local_ca-key.pem \
          -o yaml --dry-run=client | yq e .data -
...
tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JS...
tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JS...
```

## Using the issuer

Once the chart is installed, you can get automatically signed certificates for your ingress if they use the following
annotation:

```yaml
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "ca-issuer"
```

Substitute your ingress class name as per your setup, and use the value of the config `issuerName` for the `cluster-issuer` annotation.
