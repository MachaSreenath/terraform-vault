# Ways to Secure Terraform

There are a few ways to manage sensitive information in Terraform files. Here are some of the most common methods:

---

## 1. Use the Sensitive Attribute

- Terraform provides a `sensitive` attribute that can be used to mark variables and outputs as sensitive.  
- When a variable or output is marked as sensitive, Terraform will not print its value in the console output or in the state file.

**Example:**

```hcl
variable "aws_access_key_id" {
  sensitive = true
}
````

---

## 2. Secret Management System

* Store sensitive data in a **secret management system** (e.g., HashiCorp Vault, AWS Secrets Manager).
* These systems are built specifically to store passwords, API keys, SSH keys, and other secrets.
* Terraform can be configured to fetch secrets dynamically.

**Example with Vault:**

```hcl
data "vault_generic_secret" "aws_access_key_id" {
  path = "secret/aws/access_key_id"
}

variable "aws_access_key_id" {
  value = data.vault_generic_secret.aws_access_key_id.value
}
```

---

## 3. Remote Backend

* Store and **encrypt the Terraform state file** in a secure remote backend.
* This prevents local exposure of sensitive information.
* Common backends include **Terraform Cloud** and **AWS S3 (with server-side encryption)**.

**Example:**

```hcl
terraform {
  backend "s3" {
    bucket         = "my-secure-terraform-state"
    key            = "state/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
  }
}
```

---

## 4. Environment Variables

* Use **environment variables** to store secrets and avoid hardcoding them.
* Terraform can read environment variables during runtime.

**Example:**

```bash
export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
```

In Terraform:

```hcl
variable "aws_access_key_id" {
  type    = string
  default = ""
}

variable "aws_secret_access_key" {
  type    = string
  default = ""
}
```

Terraform will automatically pick up values from environment variables prefixed with `TF_VAR_`.
For example:

```bash
export TF_VAR_aws_access_key_id=YOUR_ACCESS_KEY_ID
export TF_VAR_aws_secret_access_key=YOUR_SECRET_KEY
```

---

## ✅ Best Practices

* Never commit secrets to GitHub or any VCS.
* Always use `.gitignore` to exclude sensitive files (like `terraform.tfstate`).
* Rotate credentials regularly.
* Use IAM roles instead of static credentials whenever possible.

---