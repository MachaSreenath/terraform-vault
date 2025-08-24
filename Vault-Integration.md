# Vault Integration

This document provides step-by-step instructions for setting up **HashiCorp Vault** on an AWS EC2 Ubuntu instance and connecting it with **Terraform** using AppRole authentication.

---

## Launch an Ubuntu EC2 Instance

You can spin up an Ubuntu EC2 instance using either the AWS Console or the AWS CLI. Here’s how to do it via the AWS Management Console:

- Open the **EC2 service** from the AWS Management Console.  
- Click on **Launch Instance**.  
- Select an **Ubuntu Server xx.xx LTS AMI**.  
- Choose your preferred **instance type** (e.g., `t2.micro`).  
- Configure instance details such as storage, networking, and security groups.  
- Finally, click **Launch** to create the instance.  

---

## Installing Vault on the Instance

Once the EC2 instance is ready, connect to it and follow these steps to install Vault:

**Install gpg**
```bash
sudo apt update && sudo apt install gpg
````

**Fetch the signing key and add it to keyring**

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

**Validate the key’s fingerprint**

```bash
gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
```

**Add the HashiCorp repository**

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Update package lists:

```bash
sudo apt update
```

**Install Vault**

```bash
sudo apt install vault
```

---

## Running Vault

Start Vault in development mode (for testing purposes):

```bash
vault server -dev -dev-listen-address="0.0.0.0:8200"
```

---

## Configure Terraform to Access Secrets from Vault

We will now configure Vault with AppRole authentication so Terraform can securely retrieve secrets.

### Step 1: Enable AppRole Authentication

Enable the AppRole method using the Vault CLI:

```bash
vault auth enable approle
```

This will activate the AppRole authentication mechanism inside Vault.

---

### Step 2: Define a Policy

Before creating the AppRole, we need to set up a policy:

```bash
vault policy write terraform - <<EOF
path "*" {
  capabilities = ["list", "read"]
}

path "secrets/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "kv/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "auth/token/create" {
capabilities = ["create", "read", "update", "list"]
}
EOF
```

---

### Step 3: Create an AppRole

Now create the AppRole and associate it with the policy:

```bash
vault write auth/approle/role/terraform \
    secret_id_ttl=10m \
    token_num_uses=10 \
    token_ttl=20m \
    token_max_ttl=30m \
    secret_id_num_uses=40 \
    token_policies=terraform
```

---

### Step 4: Generate Role ID and Secret ID

Finally, generate credentials for Terraform to authenticate.

**a. Get the Role ID**

```bash
vault read auth/approle/role/my-approle/role-id
```

Save this Role ID for your Terraform configuration.

**b. Get the Secret ID**

```bash
vault write -f auth/approle/role/my-approle/secret-id
```

This will generate a Secret ID. Store it securely, as it will be required for Terraform authentication.

---