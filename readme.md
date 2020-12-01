## HashiCorp Vault Key Management Secrets Engine

### This repository demonstrates the Vault Key Management Secrets Engine. Note: Vault Enterprise 1.6 and the Advanced Data Protection Module are required for this secrets engine. 

### Background: Cloud services often require enycrption keys stored in the providers Key Management System. Many highly regulated companies require the ability to maintain control of key material and require a consistent workflow to distrubute and manage the lifecycle of keys on various cloud services.  


### Setup: 
1. Configure Azure Key Vault - The main.tf file in this repository uses Terraform to configure a Service Principal and a Key Vault in Azure and configures proper permissions for this demo. This Terraform code will provide outputs for credentials that we will use to configure HashiCorp Vault to talk to Azure Key Vault. 

```
terraform init 
terraform apply
```

2. Configure Vault 

### Enable the Key Management Secrets Engine 

```
vault secrets enable keymgmt
```

### Write RSA keys to the secrets engine

```
vault write keymgmt/key/rsa-1 type="rsa-2048"
vault write keymgmt/key/rsa-2 type="rsa-2048"
```

### Create a KMS provider - configure Vault to talk to Azure Key Vault

```
vault write keymgmt/kms/keyvault \
    key_collection="<<FromTFOutput>>" \
    provider="azurekeyvault" \
    credentials=client_id="<<FromTFOutput>>" \
    credentials=client_secret="<<FromTFOutput>>" \
    credentials=tenant_id="<<FromTFOutput>>"
```

### Distribute Keys to Vault
```
vault write keymgmt/kms/keyvault/key/rsa-1 \
    purpose="encrypt,decrypt" \
    protection="hsm"

vault write keymgmt/kms/keyvault/key/rsa-2 \
    purpose="sign" \
    protection="hsm"
```

### Use the Vault CLI to verrify the keys have been distrubuted to Azure Key Vault 
```
vault list keymgmt/kms/keyvault/key/
```

### Use the Azure UI to verify the keys and their purpose. 

### Rotate the key 
```
vault write -f keymgmt/key/rsa-1/rotate
```


### Reference:
 https://learn.hashicorp.com/tutorials/vault/key-management-secrets
 https://www.vaultproject.io/docs/secrets/key-management