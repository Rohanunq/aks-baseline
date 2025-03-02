# Generate your client-facing and AKS ingress controller TLS certificates

Now that you have the [prerequisites](./01-prerequisites.md) met, follow these steps to create the TLS certificates that Azure Application Gateway will serve for clients connecting to your web app as well as the AKS ingress controller. If you already have access to appropriate certificates, or can procure them from your organization, consider doing so and skipping the certificate generation steps. **The following steps generate self-signed certs for instructive purposes only. Don't use them for a real production cluster.**

## Steps

1. Set a variable for the domain that will be used in the rest of this deployment.

   ```bash
   export DOMAIN_NAME_AKS_BASELINE="havellsaksbaseline.com"
   ```

1. Generate a client-facing, self-signed TLS certificate.

   > :book: Contoso Bicycle needs to procure a CA certificate for the web site. As this is going to be a user-facing site, they purchase an EV cert from their CA. This will serve in front of the Azure Application Gateway. They will also procure another one, a standard cert, to be used with the AKS Ingress Controller. This one is not EV, as it will not be user facing.

   :warning: Do not use the certificate created by this script for actual deployments. The use of self-signed certificates is for demonstration purposes only. For your cluster, use your organization's requirements for procurement and lifetime management of TLS certificates, *even for development purposes*.

   Create the certificate that will be presented to web clients by Azure Application Gateway for your domain.

   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out appgw.crt -keyout appgw.key -subj "/CN=bicycle.${DOMAIN_NAME_AKS_BASELINE}/O=Contoso Bicycle" -addext "subjectAltName = DNS:bicycle.${DOMAIN_NAME_AKS_BASELINE}" -addext "keyUsage = digitalSignature" -addext "extendedKeyUsage = serverAuth"
   openssl pkcs12 -export -out appgw.pfx -in appgw.crt -inkey appgw.key -passout pass:
   ```

1. Base64 encode the client-facing certificate.

   :bulb: No matter if you used a certificate from your organization or one generated from above, you'll need the certificate (as `.pfx`) to be Base64 encoded for proper storage in Key Vault later.

   ```bash
   export APP_GATEWAY_LISTENER_CERTIFICATE_AKS_BASELINE=$(cat appgw.pfx | base64 | tr -d '\n')
   echo APP_GATEWAY_LISTENER_CERTIFICATE_AKS_BASELINE: $APP_GATEWAY_LISTENER_CERTIFICATE_AKS_BASELINE
   ```

1. Generate the wildcard certificate for the AKS ingress controller.

   > :book: Contoso Bicycle procured a standard CA certificate to be used with the AKS ingress controller. This one is not EV, because it won't be user-facing. The workload team decides to use a wildcard certificate of `*.aks-ingress.havellsaksbaseline.com` for the ingress controller.

   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out traefik-ingress-internal-aks-ingress-tls.crt -keyout traefik-ingress-internal-aks-ingress-tls.key -subj "/CN=*.aks-ingress.${DOMAIN_NAME_AKS_BASELINE}/O=Contoso AKS Ingress"
   ```

1. Base64 encode the AKS ingress controller certificate.

   :bulb: No matter if you used a certificate from your organization or you generated one from above, you'll need the public certificate (as `.crt` or `.cer`) to be Base64 encoded for proper storage in Key Vault later.

   ```bash
   export AKS_INGRESS_CONTROLLER_CERTIFICATE_BASE64_AKS_BASELINE=$(cat traefik-ingress-internal-aks-ingress-tls.crt | base64 | tr -d '\n')
   echo AKS_INGRESS_CONTROLLER_CERTIFICATE_BASE64_AKS_BASELINE: $AKS_INGRESS_CONTROLLER_CERTIFICATE_BASE64_AKS_BASELINE
   ```

### Save your work in-progress

```bash
# run the saveenv.sh script at any time to save environment variables created above to aks_baseline.env
./saveenv.sh

# if your terminal session gets reset, you can source the file to reload the environment variables
# source aks_baseline.env
```

### Next step

:arrow_forward: [Prep for Microsoft Entra ID integration](./03-microsoft-entra-id.md)
