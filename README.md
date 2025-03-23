# Data Protection Using Encryption

## 📌 Project Overview

Cryptography plays a vital role in **data security**, ensuring confidentiality, integrity, and authentication. The primary function of cryptography is **encryption**, which converts data into an unreadable format, preventing unauthorized access. **Decryption** reverses this process, restoring the original data.

In this project, I used **AWS Key Management Service (AWS KMS)** and the **AWS Encryption CLI** to: </br>
✅ **Create an encryption key** with AWS KMS.  
✅ **Encrypt plaintext** into ciphertext.  
✅ **Decrypt ciphertext** back to plaintext.  

---

## **1️⃣ Creating an AWS KMS Key**

AWS KMS allows secure creation and management of cryptographic keys across AWS services.

### **Step 1: Create a Symmetric KMS Key**
1. In the **AWS Management Console**, search for **KMS** and open **Key Management Service**.
2. Click **Create a key**.
![image](https://github.com/user-attachments/assets/c05b30e2-ca38-4363-b981-c1ca4d39e1e4)

4. Choose **Key type:** `Symmetric` → Click **Next**.
![image](https://github.com/user-attachments/assets/89c8dcad-2250-4c98-9f3e-0cb611a0181b)

6. Configure key details:
   - **Alias:** `MyKMSKey`
   - **Description:** `Key used to encrypt and decrypt data files.`
![image](https://github.com/user-attachments/assets/ce9cd497-4a18-4fe3-be18-321b54a2c4de)

7. Assign administrative permissions:
   - Under **Key administrators**, select `voclabs` → Click **Next**.
![image](https://github.com/user-attachments/assets/b7b9c7a9-08b2-46ef-b76c-b88a617030d4)

8. Assign usage permissions:
   - Under **Key usage permissions**, select `voclabs` → Click **Next**.
![image](https://github.com/user-attachments/assets/dab0a827-02ca-4a16-8c07-55ca4aeb844e)

9. Review settings and click **Finish**.
![image](https://github.com/user-attachments/assets/4c55dc11-817b-433f-a871-cdc3135537d2)

### **Step 2: Copy the Key ARN**
1. Select `MyKMSKey` from the KMS console.
2. Copy the **Amazon Resource Name (ARN)** for later use:
   ```plaintext
   arn:aws:kms:us-west-2:004261589378:key/6a68ee52-1e87-41c8-b800-38292bbf04fb
   ```
![image](https://github.com/user-attachments/assets/0adc0cd0-a767-45b6-b851-48e5180a7f3a)

---

## **2️⃣ Configuring the File Server Instance**

Before encrypting files, I configured **AWS credentials** and installed the **AWS Encryption CLI**.

### **Step 1: Connect to the EC2 Instance**
1. In the **AWS Console**, search for **EC2** and open it.
2. Select `File Server` → Click **Connect**.
![image](https://github.com/user-attachments/assets/d5df3d37-1a5c-4358-a32b-7afb55f2419a)

4. Choose **Session Manager** → Click **Connect**.
![image](https://github.com/user-attachments/assets/084c605d-bef0-43b1-90c2-53d6bc241e35)

### **Step 2: Configure AWS Credentials**
1. Change to the home directory:
   ```bash
   cd ~
   aws configure
   ```
2. Enter temporary placeholders when prompted:
   - **AWS Access Key ID:** `1`
   - **AWS Secret Access Key:** `1`
   - **Default region name:** `us-west-2`
   - **Default output format:** (Press Enter)

![image](https://github.com/user-attachments/assets/f033607f-3100-493b-a877-5d0f257b33ad)

3. Retrieve the actual credentials already pre-defined for this project.
4. Open the credentials file:
   ```bash
   nano ~/.aws/credentials
   ```
![image](https://github.com/user-attachments/assets/d755376b-6934-416e-9f89-f90bededaefb)

5. Replace the contents with the correct credentials:
   ```plaintext
   [default]
   aws_access_key_id=YOUR_ACCESS_KEY
   aws_secret_access_key=YOUR_SECRET_KEY
   aws_session_token=YOUR_SESSION_TOKEN
   ```
![image](https://github.com/user-attachments/assets/c327d76a-823e-46ce-9dab-1f1c3e1a8671)

6. Save and close the file.
7. Verify the credentials:
   ```bash
   cat ~/.aws/credentials
   ```
![image](https://github.com/user-attachments/assets/9c6d7d83-acc4-4239-a243-dd0f932d9d0f)

### **Step 3: Install AWS Encryption CLI**
1. Install the AWS Encryption CLI:
   ```bash
   pip3 install aws-encryption-sdk-cli
   ```
![image](https://github.com/user-attachments/assets/6fe95d3b-edd0-4e83-8cdd-bf11fdf40d01)

2. Add it to the system path:
   ```bash
   export PATH=$PATH:/home/ssm-user/.local/bin
   ```
![image](https://github.com/user-attachments/assets/bf97a7c4-e302-4d65-a7f4-a8533788b289)

---

## **3️⃣ Encrypting and Decrypting Data**

### **Step 1: Create a Text File**
```bash
touch secret1.txt secret2.txt secret3.txt
echo 'TOP SECRET 1!!!' > secret1.txt
cat secret1.txt
```
![image](https://github.com/user-attachments/assets/14f413f0-afb2-49ac-be65-61af683c0d69)

### **Step 2: Set Up Encryption Key Variable**
```bash
keyArn=arn:aws:kms:us-west-2:004261589378:key/6a68ee52-1e87-41c8-b800-38292bbf04fb
```
![image](https://github.com/user-attachments/assets/b282be3f-2baa-4067-8d9b-3ae17af434cc)

### **Step 3: Encrypt the File**
```bash
aws-encryption-cli --encrypt \
                     --input secret1.txt \
                     --wrapping-keys key=$keyArn \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --commitment-policy require-encrypt-require-decrypt \
                     --output ~/output/.
```
![image](https://github.com/user-attachments/assets/13a50702-8c10-4c45-98a6-27f665e8a4c1)

🔹 **Breakdown of Command:**  
✅ **`--encrypt`** → Encrypts the file.  
✅ **`--input`** → Specifies file to encrypt.  
✅ **`--wrapping-keys key=$keyArn`** → Uses AWS KMS key for encryption.  
✅ **`--metadata-output`** → Saves metadata.  
✅ **`--output ~/output/.`** → Saves encrypted file in `output` directory.  

### **Step 4: Verify Encryption**
```bash
echo $?
```
![image](https://github.com/user-attachments/assets/d5058a80-afea-44b8-97a8-474fa7767f55)

If the command succeeds, the value of $? is 0. If the command fails, the value is nonzero

```bash
ls output
cat output/secret1.txt.encrypted
```
✅ Expected Output: A non readable file that means encryption was successful  
![image](https://github.com/user-attachments/assets/8abb1c39-d1ce-4989-bb84-7f5fe9c2f188)

### **Step 5: Decrypt the File**
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
![image](https://github.com/user-attachments/assets/49dbce20-4fdd-4206-8955-0f5d59b08a01)

### **Step 6: Verify Decryption**
```bash
ls
cat secret1.txt.encrypted.decrypted
```
✅ Expected Output: `TOP SECRET 1!!!`

![image](https://github.com/user-attachments/assets/57f91099-1117-455d-9cf9-e463741be4f5)

---

## **🚀 Conclusion**
Through this project, I successfully: </br>
✅ Created an **AWS KMS encryption key**.  
✅ Configured **AWS Encryption CLI** for secure encryption and decryption.  
✅ Encrypted plaintext into ciphertext using **AWS KMS**.  
✅ Decrypted the ciphertext back into readable plaintext.  

This project demonstrates how AWS **ensures data protection** through strong encryption mechanisms.
