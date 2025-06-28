# 🍀 oci-letsencrypt

This project automates the generation and deployment of **Let's Encrypt SSL/TLS certificates** in [Oracle Cloud Infrastructure (OCI)](https://www.oracle.com/cloud/) using **DNS-01** or **HTTP-01** challenges. It supports automatic renewal and integration with OCI Certificates, Vault, and Secrets.

For detailed install instructions see https://redthunder.blog/2022/10/14/managing-multiple-lets-encrypt-certificates-with-oracle-cloud-infrastructure/

## 📌 Features

- 🛡️ Automatic SSL certificate generation with Let's Encrypt
- ✅ Supports **DNS-01** challenge via OCI DNS
- 🌐 NEW: Supports **HTTP-01** challenge using **OCI WAF (Web Application Firewall)**
- 🔐 Integration with OCI Certificates, Vault, and Secrets
- 🔁 Auto-renewal support
- 🪝 Optional auto-deploy functionality

---

## 🔧 HTTP-01 Challenge with OCI WAF (NEW)

✨ **Contributed by [Cristhian Won](https://github.com/cristhianwon)**

This new feature **enables HTTP-01 certificate validation using a WAF policy**, which is ideal when **the DNS of the domain is managed by an external provider and you cannot request frequent updates**, as required by DNS-01. In this scenario, the external party can point the domain to your **OCI Load Balancer public IP**, and with a **WAF policy attached**, the challenge response can be handled dynamically without further DNS changes.

### 📝 Requirements

1. A domain (e.g., `example.com`) pointing to an OCI Load Balancer IP.
2. A **WAF policy** attached to that Load Balancer.
3. Your `config.json` (stored in object storage bucket) must be updated with the WAF info.

### 📁 Example `config.json` for HTTP-01

```json
{
  "certificates": [
    {
      "cn_name": "example.com",
      "alt_names": [],
      "waf_ocid": "ocid1.webappfirewallpolicy.oc1...",
      "waf_region": "sa-santiago-1",
      "certificate_region": "sa-santiago-1",
      "cert_compartment_ocid": "ocid1.compartment.oc1...",
      "auto_deploy": true,
      "vault_region": "sa-santiago-1",
      "vault_ocid": "ocid1.vault.oc1...",
      "vault_master_key_ocid": "ocid1.key.oc1...",
      "renew_days_before_expiry": 30
    }
  ]
}
```

### 🔐 Required IAM Policies

Make sure the dynamic group (e.g., acme-certbot-dg) has the following permissions:

```
Allow dynamic-group acme-certbot-dg to inspect functions-family in compartment example-compartment
Allow dynamic-group acme-certbot-dg to use log-content in compartment example-compartment
Allow dynamic-group acme-certbot-dg to manage leaf-certificate-family in compartment example-compartment
Allow dynamic-group acme-certbot-dg to manage secret-family in compartment example-compartment
Allow dynamic-group acme-certbot-dg to manage key-family in compartment example-compartment
Allow dynamic-group acme-certbot-dg to use vaults in compartment example-compartment
Allow dynamic-group acme-certbot-dg to use objects in compartment example-compartment
Allow dynamic-group acme-certbot-dg to use fn-invocation in compartment example-compartment
Allow dynamic-group acme-certbot-dg to manage waf-policy in compartment example-compartment
Allow dynamic-group acme-certbot-dg to use dns in tenancy

Allow service faas to {KEY_READ} in compartment example-compartment where request.operation = 'GetKeyVersion'
Allow service faas to {KEY_VERIFY} in compartment example-compartment where request.operation = 'Verify'
```

### ⚙️ How It Works

The tool runs as a function or scheduled job that:

1. Loads the certificate config from a JSON file in Object Storage
2. Resolves whether to use DNS or HTTP challenge
3. Issues the challenge via Let's Encrypt
4. For HTTP-01:
   - It configures a temporary URL on the WAF policy
5. Let's Encrypt calls this URL to verify ownership
6. Stores the certificate into Certificate Manager

### 🤝 Acknowledgements

Originally developed by @scotti-fletcher 🎉

HTTP-01 WAF support added by @cristhianwon 🚀
