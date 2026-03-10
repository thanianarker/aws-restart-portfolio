# Data Protection Using Encryption

## Overview

I completed a comprehensive hands-on lab on cryptographic data protection using AWS Key Management Service (KMS) and the AWS Encryption CLI. Through this lab, I gained practical experience implementing encryption for sensitive data, understanding symmetric cryptography, and managing encryption keys in a cloud environment. I created a KMS encryption key, configured an EC2 instance with AWS credentials and the encryption toolkit, and successfully encrypted plaintext files into ciphertext and decrypted them back to readable form. By the end of this lab, I demonstrated mastery of encryption fundamentals and the ability to protect sensitive data using AWS's cryptographic services.

## Topics Covered

After completing this lab, I was able to:

- Create and manage AWS KMS encryption keys
- Configure symmetric encryption (same key for encrypt/decrypt)
- Set up AWS credentials on EC2 instances
- Install and configure the AWS Encryption CLI
- Encrypt plaintext files into ciphertext
- Decrypt ciphertext back to plaintext
- Use encryption context to add metadata to encrypted data
- Verify encryption/decryption operations succeeded
- Understand the cryptographic transformation process
- Apply key commitment security policies
- Interpret and manage encrypted file outputs

## Introducing the Technologies

### Cryptography Fundamentals

Cryptography is the science of transforming information into secret code, ensuring confidentiality, authenticity, and integrity. The two primary operations are encryption (plaintext → ciphertext) and decryption (ciphertext → plaintext).

| Cryptographic Term | Definition | Impact |
|-------------------|-----------|--------|
| **Plaintext** | Original, readable data | Vulnerable to unauthorized access if unprotected |
| **Ciphertext** | Encrypted, unreadable data | Protected; meaningless without decryption |
| **Encryption Key** | Secret value used to transform plaintext | Must be secured; controls access to data |
| **Encryption Algorithm** | Mathematical process for transformation | Determines security strength and performance |
| **Decryption** | Reverse process transforming ciphertext to plaintext | Requires correct key and algorithm |

### Encryption Types: Symmetric vs. Asymmetric

| Encryption Type | Key Count | Performance | Use Case | Complexity |
|-----------------|-----------|-------------|----------|-----------|
| **Symmetric** | 1 (shared key) | Fast, efficient | Encrypting bulk data, files, databases | Lower |
| **Asymmetric** | 2 (public + private) | Slower, compute-intensive | Key exchange, digital signatures, TLS | Higher |

For this lab, I used **symmetric encryption** where the same key encrypts and decrypts data. This approach is ideal for protecting files and bulk data.

### AWS Key Management Service (KMS)

AWS KMS is a managed service for creating and managing cryptographic keys. It provides:

| KMS Feature | Benefit |
|-------------|---------|
| Hardware Security Modules (HSMs) | Keys stored in FIPS 140-2 validated hardware |
| Key Rotation | Automatic key versioning without re-encrypting data |
| Access Control | IAM policies control who can use each key |
| Audit Logging | CloudTrail logs all key usage for compliance |
| Multi-Region Keys | Replicate keys across regions for disaster recovery |

### AWS Encryption CLI

The AWS Encryption CLI is a command-line tool for encrypting and decrypting data using AWS KMS keys. It provides options for:

| CLI Parameter | Purpose | Example |
|---------------|---------|---------|
| `--encrypt` | Perform encryption operation | `--encrypt` |
| `--decrypt` | Perform decryption operation | `--decrypt` |
| `--input` | Specify file to encrypt/decrypt | `--input secret.txt` |
| `--wrapping-keys` | Specify KMS key to use | `key=$keyArn` |
| `--encryption-context` | Add metadata to encryption | `purpose=test` |
| `--commitment-policy` | Enable key commitment security | `require-encrypt-require-decrypt` |
| `--output` | Specify output directory | `--output ~/output/.` |

## Tasks

### Task 1: Create AWS KMS Encryption Key

#### Step 1.1: Navigate to KMS Service

I opened the AWS Management Console and searched for "KMS" in the search bar. I selected **Key Management Service** to access the KMS dashboard.

#### Step 1.2: Create Symmetric KMS Key

I clicked **Create a key** and configured the following:

| Setting | Value |
|---------|-------|
| Key Type | Symmetric |
| Key Material Origin | KMS (default) |

📝 **Note**: Symmetric keys use a single shared secret to encrypt and decrypt. This makes them ideal for protecting files and bulk data where performance is important.

#### Step 1.3: Configure Key Metadata

On the **Add labels** page, I configured:

| Setting | Value |
|---------|-------|
| Alias | MyKMSKey |
| Description | Key used to encrypt and decrypt data files |

✅ **Task complete**: Key alias configured for easy reference.

#### Step 1.4: Set Key Administrative Permissions

On the **Define key administrative permissions** page, I searched for and selected the **voclabs** IAM role in the **Key administrators** section.

📝 **Note**: The voclabs IAM role was pre-created for this lab environment. Granting it administrative permissions allows it to manage the key's lifecycle.

#### Step 1.5: Set Key Usage Permissions

On the **Define key usage permissions** page, I selected the **voclabs** IAM role in the **This account** section.

💭 **Consider**: Usage permissions determine which principals (users, roles, services) can use the key to encrypt and decrypt. Administrative permissions control who can modify the key itself.

#### Step 1.6: Review and Finish

I reviewed all settings and clicked **Finish** to create the key.

#### Step 1.7: Copy KMS Key ARN

I clicked the **MyKMSKey** link to view the key details. I copied the **ARN (Amazon Resource Name)** value:

```
arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012
```

I saved this ARN to a text editor for use in later tasks.

✅ **Task complete**: KMS encryption key successfully created and ARN copied.

💭 **Consider**: The ARN uniquely identifies the key across AWS regions and accounts. It can also be referenced by key ID, alias name, or alias ARN depending on the context.

### Task 2: Configure File Server Instance

#### Step 2.1: Connect to File Server Instance

I navigated to the **EC2** service and selected the **File Server** instance from the instances list. I clicked **Connect** to open the connection options.

I selected the **Session Manager** tab and clicked **Connect** to establish a shell session without needing SSH credentials.

📝 **Note**: Session Manager provides secure shell access without managing SSH keys. It requires an IAM role with appropriate permissions, which was pre-configured on the File Server instance.

#### Step 2.2: Configure AWS Credentials

I navigated to the home directory and ran the AWS configuration wizard:

```bash
cd ~
aws configure
```

When prompted, I entered:

| Prompt | Entry |
|--------|-------|
| AWS Access Key ID | 1 (temporary placeholder) |
| AWS Secret Access Key | 1 (temporary placeholder) |
| Default region name | Copied from Vocareum AWS Details page |
| Default output format | (left blank, pressed Enter) |

This created the initial AWS credentials file at `~/.aws/credentials`.

#### Step 2.3: Update AWS Credentials File

I accessed the Vocareum AWS Details page and selected **Show** next to **AWS CLI**. I copied the credential block containing the actual AWS access key, secret access key, and session token.

I opened the credentials file in vi editor:

```bash
vi ~/.aws/credentials
```

I deleted the placeholder credentials by typing `dd` multiple times, then pasted the actual credentials:

```
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
aws_session_token = AQoDYXdzEJr...(session token content)...
```

I saved the file by pressing Escape, typing `:wq`, and pressing Enter.

✅ **Task complete**: AWS credentials configured on File Server instance.

📝 **Note**: The session token is temporary and expires after the lab ends. The credentials file is read by AWS CLI commands to authenticate API requests.

#### Step 2.4: Install AWS Encryption CLI

I installed the AWS Encryption CLI using pip and updated the PATH:

```bash
pip3 install aws-encryption-sdk-cli
export PATH=$PATH:/home/ssm-user/.local/bin
```

The first command installs the encryption CLI tool. The second command adds the installation directory to the PATH so the `aws-encryption-cli` command is available in the terminal.

✅ **Task complete**: AWS Encryption CLI installed and available.

### Task 3: Encrypt and Decrypt Data

#### Step 3.1: Create Plaintext Files

I created three text files to demonstrate encryption:

```bash
touch secret1.txt secret2.txt secret3.txt
echo 'TOP SECRET 1!!!' > secret1.txt
```

I verified the contents of the plaintext file:

```bash
cat secret1.txt
```

**Output:**
```
TOP SECRET 1!!!
```

#### Step 3.2: Create Output Directory and Set KMS Key Variable

I created a directory for encrypted output:

```bash
mkdir output
```

I set the KMS key ARN in a variable for use in encryption commands:

```bash
keyArn=arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012
```

I replaced the ARN with the actual KMS key ARN I copied in Task 1.

💭 **Consider**: Storing the key ARN in a variable makes the encryption command cleaner and allows me to reference the same key multiple times without retyping.

#### Step 3.3: Encrypt Plaintext File

I encrypted the secret1.txt file using the AWS Encryption CLI:

```bash
aws-encryption-cli --encrypt \
                     --input secret1.txt \
                     --wrapping-keys key=$keyArn \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --commitment-policy require-encrypt-require-decrypt \
                     --output ~/output/.
```

**Command Parameters Explained:**

| Parameter | Purpose |
|-----------|---------|
| `--encrypt` | Specifies encryption operation |
| `--input secret1.txt` | Source file to encrypt (plaintext) |
| `--wrapping-keys key=$keyArn` | KMS key to use for encryption |
| `--metadata-output ~/metadata` | File to store encryption metadata |
| `--encryption-context purpose=test` | Additional authenticated data (AAD) |
| `--commitment-policy require-encrypt-require-decrypt` | Enforce key commitment security |
| `--output ~/output/.` | Output directory for encrypted file |

I verified the command succeeded by checking the exit status:

```bash
echo $?
```

**Output:** `0` (indicates success)

#### Step 3.4: View Encrypted File

I listed the output directory:

```bash
cd output
ls
```

**Output:**
```
secret1.txt.encrypted
```

I attempted to view the encrypted file contents:

```bash
cat secret1.txt.encrypted
```

**Sample Output (unreadable ciphertext):**
```
[binary/encrypted content that appears as random characters]
EXF_ENC_CONTEXT={"purpose":"test"}
[additional binary encrypted data]
```

📝 **Note**: The encrypted file contains ciphertext—random-appearing binary data that is meaningless without the correct encryption key and decryption algorithm. This demonstrates that encryption successfully transforms readable plaintext into protected ciphertext.

⚠️ **Caution**: Even with access to the encrypted file, an attacker cannot read the contents without the KMS key. The file is useless without decryption.

✅ **Task complete**: Plaintext file successfully encrypted to ciphertext.

#### Step 3.5: Decrypt Ciphertext File

I decrypted the encrypted file back to plaintext:

```bash
aws-encryption-cli --decrypt \
                     --input secret1.txt.encrypted \
                     --wrapping-keys key=$keyArn \
                     --commitment-policy require-encrypt-require-decrypt \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --max-encrypted-data-keys 1 \
                     --buffer \
                     --output .
```

**Decryption Command Parameters:**

| Parameter | Purpose |
|-----------|---------|
| `--decrypt` | Specifies decryption operation |
| `--input secret1.txt.encrypted` | Encrypted file to decrypt (ciphertext) |
| `--wrapping-keys key=$keyArn` | KMS key to use for decryption (must match encryption key) |
| `--encryption-context purpose=test` | Must match encryption context used during encryption |
| `--max-encrypted-data-keys 1` | Security check: reject if multiple keys used |
| `--buffer` | Process file in memory (suitable for small files) |
| `--output .` | Output current directory for decrypted file |

#### Step 3.6: View Decrypted File

I listed the current directory to locate the decrypted file:

```bash
ls
```

**Output:**
```
secret1.txt.encrypted
secret1.txt.encrypted.decrypted
```

I viewed the contents of the decrypted file:

```bash
cat secret1.txt.encrypted.decrypted
```

**Output:**
```
TOP SECRET 1!!!
```

✅ **Task complete**: Ciphertext successfully decrypted back to original plaintext.

💭 **Consider**: The decryption process is reversible because I used the same KMS key and encryption context. If the encryption context or key differed, decryption would fail. This provides authenticated encryption—the recipient verifies that decryption produced valid plaintext with the expected context.

## Encryption and Decryption Process

The cryptographic transformation process uses symmetric encryption:

**Encryption Process:**
```
Plaintext (readable)
    ↓
Symmetric Key + Encryption Algorithm
    ↓
Ciphertext (unreadable, protected)
```

**Decryption Process:**
```
Ciphertext (unreadable)
    ↓
Same Symmetric Key + Decryption Algorithm
    ↓
Plaintext (readable, restored)
```

The security depends on:
1. **Key secrecy**: Only authorized users/services access the KMS key
2. **Algorithm strength**: AWS KMS uses AES-256, the cryptographic standard
3. **Proper key management**: AWS KMS ensures keys are stored in FIPS 140-2 validated HSMs

## Conclusion

I successfully demonstrated comprehensive data protection using AWS cryptographic services:

- ✅ Created an AWS KMS symmetric encryption key with proper access controls
- ✅ Configured AWS credentials on an EC2 instance for KMS API access
- ✅ Installed and configured the AWS Encryption CLI
- ✅ Created plaintext files containing sensitive data
- ✅ Encrypted plaintext files to ciphertext using AWS KMS and the encryption CLI
- ✅ Verified encrypted files are unreadable without decryption
- ✅ Decrypted ciphertext back to original plaintext using the same KMS key
- ✅ Verified decrypted contents match original plaintext exactly
- ✅ Understood symmetric encryption and the role of encryption keys
- ✅ Applied encryption context for authenticated encryption
- ✅ Implemented key commitment security policies

This lab demonstrated practical application of cryptographic principles to protect sensitive data in a cloud environment. The ability to encrypt and decrypt data using managed services like AWS KMS is essential for securing customer data, meeting compliance requirements, and protecting intellectual property.

## Additional Resources

- [AWS Key Management Service Documentation](https://docs.aws.amazon.com/kms/)
- [AWS KMS Best Practices](https://docs.aws.amazon.com/kms/latest/developerguide/best-practices.html)
- [AWS Encryption SDK Documentation](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/)
- [AWS Encryption CLI User Guide](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/crypto-cli.html)
- [Symmetric Encryption Concepts](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)
- [FIPS 140-2 Standard](https://en.wikipedia.org/wiki/FIPS_140-2)
- [AWS Identity and Access Management (IAM)](https://docs.aws.amazon.com/iam/)
- [CloudTrail Logging for KMS](https://docs.aws.amazon.com/kms/latest/developerguide/logging-using-cloudtrail.html)
- [Data Protection Compliance](https://docs.aws.amazon.com/security/)
