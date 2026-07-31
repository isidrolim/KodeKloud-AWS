# Create Key Pair via AWS CLI

## Problem Statement

Create an **EC2 Key Pair** with the following requirements:

- **Key Pair Name:** `datacenter-kp`
- **Key Pair Type:** `RSA`
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI is authenticated.
   ```bash
   aws sts get-caller-identity
   ```

2. Verify the target Region.
   ```bash
   aws configure get region
   ```

3. Check if the key pair already exists.
   ```bash
   aws ec2 describe-key-pairs \
     --key-names datacenter-kp \
     --region us-east-1
   ```

4. Create the RSA key pair and save the private key locally.
   ```bash
   aws ec2 create-key-pair \
     --key-name datacenter-kp \
     --key-type rsa \
     --query 'KeyMaterial' \
     --output text \
     --region us-east-1 > datacenter-kp.pem
   ```

5. Secure the private key.
   ```bash
   chmod 400 datacenter-kp.pem
   ```

## Validation

- Verify the key pair exists.
  ```bash
  aws ec2 describe-key-pairs \
    --key-names datacenter-kp \
    --region us-east-1
  ```

- Verify the private key file exists.
  ```bash
  ls -l datacenter-kp.pem
  ```
