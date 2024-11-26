
**Student ID**: 22110093  
**Name**: Dinh Thi Thanh Vy  
**Course**: INSE33030E_02FIE

---
# Task 1: Public-key based authentication 
**Question 1**: 
Implement public-key based authentication step-by-step with openssl according the following scheme.
![image](https://github.com/user-attachments/assets/55b7a69b-b482-43f9-a29d-ee319a5c7da6)


**Answer 1**:

## 1. Setup Environment
### 1.1 Alice and Bob Configuration
- Alice (server): IP 10.9.0.5
- Bob (client): IP 10.9.0.6

![image](https://github.com/user-attachments/assets/d79db62f-7a51-4669-9557-6aadd5648726)

![image](https://github.com/user-attachments/assets/fba5e91e-046d-4824-ba38-8e51ede60dfc)

### 1.2 Create a Text File

Alice creates a text file that will be securely sent to Bob:

```echo 'ISlab healing this morning ...........!!!!' >> textfile.txt```
![image](https://github.com/user-attachments/assets/4af5e76c-435c-4e1f-98a8-c218c54d7a5f)

## 2. Key Generation
### 2.1 Generate RSA Key Pair (Bob's Machine)

On Bob’s machine, generate an RSA private key:

```openssl genrsa -out keypair.pem 2048```

- ```openssl genrsa```: Generates an RSA private key.
- ```-out keypair.pem```: Saves the private key to keypair.pem.
- ```2048```: Sets the RSA key length to 2048 bits, which balances security and performance.

### 2.2 Extract Public Key

From the private key, extract the public key:

```openssl rsa -in keypair.pem -pubout -out publickey.crt```

- ```-in keypair.pem```: Input the private key.
- ```-pubout```: Extract the public key.
- ```-out publickey.crt```: Save the public key to publickey.crt.

![image](https://github.com/user-attachments/assets/2e2ce151-976d-4e29-b10c-1cc12b49f755)

## 3. Transfer Public Key to Alice
### 3.1 Grant Permissions
Before transferring, ensure the public key file is accessible:

```chmod 777 /home```
### 3.2 Send Public Key to Alice
Transfer Bob’s public key to Alice using scp:

```scp publickey.crt alice@10.9.0.5:/home```

![image](https://github.com/user-attachments/assets/1edbbdcf-33a2-4317-aff4-96c64c6ec223)

If the SSH server is not running on Alice’s machine, install and start it:
```
apt update
apt install openssh-server
service ssh start
```
Retry the transfer after starting the SSH service.
![image](https://github.com/user-attachments/assets/88430f3a-8abf-4e75-b7f9-b09179dae7a6)

![image](https://github.com/user-attachments/assets/6de67abc-0100-4d6f-9227-90c2c7ea8dca)

--> ALice recieved public key
## 4. Alice Prepares the Encrypted Data
### 4.1 Generate a Random Symmetric Key

Alice generates a random symmetric key for AES encryption:

```openssl rand -hex 32 > randompassword```

![image](https://github.com/user-attachments/assets/b0bd8544-3ce1-4e4f-8b59-780d3bdd575b)

### 4.2 Encrypt the Text File with AES

Alice encrypts the file (textfile.txt) using AES encryption with the generated symmetric key:

```openssl enc -aes-256-cbc -in textfile.txt -out file.enc -pass file:/home/randompassword```
- ```enc -aes-256-cbc```: Specifies AES encryption with CBC mode.
- ```-in textfile.txt```: Input file to encrypt.
- ```-out file.enc```: Output encrypted file.
- ```-pass file:/home/randompassword```: Uses the symmetric key stored in randompassword.

### 4.3 Encrypt the Symmetric Key with RSA
Alice encrypts the symmetric key using Bob’s public RSA key:

```openssl rsautl -encrypt -inkey publickey.crt -pubin -in randompassword -out randompassword.encrypted```

- ```rsautl -encrypt```: Encrypts data with RSA.
- ```-inkey publickey.crt```: Uses Bob’s public key.
- ```-pubin```: Indicates the input is a public key.
- ```-in randompassword```: The symmetric key to encrypt.
- ```-out randompassword.encrypted```: Saves the encrypted symmetric key.

## 5. Transfer Encrypted Files to Bob

Alice sends the encrypted data (file.enc) and the encrypted symmetric key (randompassword.encrypted) to Bob:

```scp file.enc randompassword.encrypted bob@10.9.0.6:/home```
![image](https://github.com/user-attachments/assets/c70f8023-2712-4ee9-9a6f-3e39a567a017)

## 6. Bob Decrypts the Data
### 6.1 Decrypt the Symmetric Key
On Bob’s machine, decrypt the symmetric key using his RSA private key:

- ```openssl rsautl -decrypt -inkey keypair.pem -in randompassword.encrypted -out randompassword.decrypted```

- ```rsautl -decrypt```: Decrypts data with RSA.
- ```-inkey keypair.pem```: Uses Bob’s private key.
- ```-in randompassword.encrypted```: Encrypted symmetric key.
- ```-out randompassword.decrypted```: Saves the decrypted symmetric key.
![image](https://github.com/user-attachments/assets/a6de6e30-488c-486d-9442-e36849ab35d7)

### 6.2 Decrypt the File with the Symmetric Key
Bob uses the decrypted symmetric key to decrypt the file:

```openssl enc -aes-256-cbc -d -in file.enc -out file_decrypted.txt -pass file:/home/randompassword.decrypted```
- ```-d```: Specifies decryption mode.
- ```in file.enc```: Input encrypted file.
- ```out file_decrypted.txt```: Output decrypted file.
- ```pass file:/home/randompassword.decrypted```: Uses the decrypted symmetric key.

![image](https://github.com/user-attachments/assets/e64c8d39-d266-406c-b265-c3c67d530b31)

## 7. Completion
Bob now has access to the original file content (file_decrypted.txt), which contains the message:

```ISlab healing this morning ...........!!!!```

Security Principles Explained
Symmetric Key Encryption (AES):

Used to encrypt large files efficiently.
Requires a shared key (randomly generated).
Asymmetric Key Encryption (RSA):

Used to securely share the symmetric key.
Only the recipient with the private RSA key can decrypt the symmetric key.
Secure File Transfer (SCP):

Files are securely transferred between Alice and Bob using SSH.
By combining RSA and AES, the system ensures both security and efficiency in the file transfer process.

# Task 2: Encrypting large message 
Create a text file at least 56 bytes.

**Question 1**:
Encrypt the file with aes-256 cipher in CFB and OFB modes. How do you evaluate both cipher as far as error propagation and adjacent plaintext blocks are concerned. 

**Answer 1**:

- Enrypt file with cfb mode:
  
```openssl enc -aes-256-cfb -in textfile.txt -out ciphertext_cfb.bin -K $(cat randompassword) -iv $(cat iv)```

- Encrypt file with ofb mode:

```openssl enc -aes-256-ofb -in textfile.txt -out ciphertext_ofb.bin -K $(cat randompassword) -iv $(cat iv)```

![image](https://github.com/user-attachments/assets/e8771b5b-3f94-4ffa-8c5a-ecac7ae51c06)

**Error Propagation & Adjacent Plaintext Blocks Dependency**

### CFB Mode:
![image](https://github.com/user-attachments/assets/38be08b7-d250-4fc5-aaa8-1a93503ef639)

- Error Propagation: High.
    - Cause: In CFB, the encryption process relies on feedback from previous ciphertext blocks. This means that each block of ciphertext is linked to the previous block of ciphertext, and as a result, a single-bit error in the ciphertext will affect not only the current plaintext block but also potentially multiple subsequent blocks.
    - Example: If one bit in the ciphertext is corrupted, the decryption of the current and the next few plaintext blocks will be completely altered due to the feedback mechanism.
    - Impact: This high error propagation makes CFB vulnerable in environments where data integrity is critical, as small errors in transmission can corrupt larger portions of the data.

- Block Dependency: Strong.
    - Cause: Since CFB relies on feedback, each plaintext block is directly dependent on the previous ciphertext block. This interdependence means that a corrupted ciphertext block will not only affect the decryption of that block but also corrupt the following blocks, causing a cascading effect.
    - Example: If the ciphertext block corresponding to plaintext block 2 is corrupted, plaintext block 2 will be incorrect, and plaintext block 3 will also be affected because it depends on the incorrect ciphertext block 2.
    - Impact: This strong interdependence between adjacent plaintext blocks ensures that CFB provides robust encryption, but it also means that errors have a wide-ranging impact.


### OFB Mode:
![image](https://github.com/user-attachments/assets/343c3156-ad93-4d10-8634-82cfa9d2c37e)

- Error Propagation: Low.
    - Cause: In OFB, the keystream is generated independently of the ciphertext, and it does not rely on feedback. This means that errors in the ciphertext only affect the corresponding byte in the plaintext, and do not propagate to other plaintext blocks.
    - Example: If one byte in the ciphertext is corrupted, only the corresponding byte in the plaintext will be affected. The rest of the plaintext remains unaffected.
    - Impact: OFB’s low error propagation is beneficial in scenarios where transmission errors are common, like in network communications, because the damage is limited to a single byte rather than multiple blocks.

- Block Dependency: Weak (or No Dependency).
    - Cause: In OFB, the keystream is generated independently of both the plaintext and ciphertext. Each block of plaintext is XORed with the keystream generated independently of any other data. This means that a corrupted ciphertext block only affects the corresponding plaintext block and does not affect adjacent blocks.
    - Example: If ciphertext block 2 is corrupted, only plaintext block 2 will be incorrect, while plaintext blocks 1 and 3 will remain unaffected. 
    - Impact: OFB mode ensures that adjacent blocks are independent, which makes it more resilient to errors. However, this independence also means that the mode doesn't link plaintext blocks together, potentially providing weaker integrity protection compared to CFB.
**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:

In this scenario, you are working with AES encryption in two different modes: CFB (Cipher Feedback) and OFB (Output Feedback). You are also modifying the encrypted file (by changing the 8th bit of the ciphertext) to observe how the decryption process behaves with error propagation in both modes. Below is a detailed explanation of each step:

Environment: 
- Alice: IP 10.9.0.5

**Generate an Initialization Vector (IV)**
```openssl rand -hex 16 > iv```

![image](https://github.com/user-attachments/assets/76b7ca16-79c8-4305-b0cc-5cf2101e556c)


- Purpose: This command generates a random Initialization Vector (IV) of 16 bytes (or 128 bits) in hexadecimal format.
- How it works: The openssl rand -hex 16 command generates 16 random bytes (128 bits) and outputs them in hexadecimal format. This IV is necessary for both CFB and OFB modes of AES encryption to ensure randomness and security during encryption.

**Encrypting the File in AES-256 CFB Mode**
```openssl enc -aes-256-cfb -in textfile.txt -out ciphertext_cfb.bin -K $(cat randompassword) -iv $(cat iv)```
- Purpose: This command encrypts the textfile.txt using AES-256 encryption in CFB mode.
  - ```-aes-256-cfb```: Specifies the encryption algorithm (AES) and mode (CFB) with a 256-bit key.
  - ```-in textfile.txt```: Input file to be encrypted.
  - ```-out ciphertext_cfb.bin```: Output file where the encrypted data will be stored.
  - ```-K $(cat randompassword)```: The -K flag specifies the encryption key. In this case, it uses the randompassword file from Task 1.
  - ```-iv $(cat iv)```: Specifies the IV generated in Step 1, which is necessary for CFB encryption.

**Encrypting the File in AES-256 OFB Mode**

```openssl enc -aes-256-cfb -in textfile.txt -out ciphertext_ofb.bin -K $(cat randompassword) -iv $(cat iv)```

- Purpose: This command encrypts the textfile.txt using AES-256 encryption in OFB mode, similar to CFB but with different feedback mechanisms.
 Explanation: The same flags are used as in the previous step, but the encryption mode is set to OFB instead of CFB.

**Modifying the 8th Byte in the Ciphertext**
- CFB Mode:
```dd if=/dev/zero bs=1 count=1 seek=7 conv=notrunc of=ciphertext_cfb.bin```
- OFB Mode:
```dd if=/dev/zero bs=1 count=1 seek=7 conv=notrunc of=ciphertext_ofb.bin```

![image](https://github.com/user-attachments/assets/18ef3fbf-38b4-49c1-bd23-eca9e1e27da1)

**Purpose:** This step simulates corruption by modifying the 8th byte of the ciphertext (after the 7th byte) by replacing it with zeros.
- ```dd if=/dev/zero```: Specifies that the input data comes from /dev/zero (which produces zero bytes).
- ```bs=1```: Sets the block size to 1 byte.
- ```count=1```: This limits the operation to just 1 byte.
- ```seek=7```: This tells dd to skip the first 7 bytes, so it starts modifying at the 8th byte.
- ```conv=notrunc```: Ensures that the file is modified in place, rather than truncated.
of=ciphertext_cfb.bin or ciphertext_ofb.bin: Specifies the encrypted file (depending on the mode).

**Analysis of Error Propagation**
After modifying the ciphertext, you will observe how the decryption behaves for CFB and OFB modes.

![image](https://github.com/user-attachments/assets/c195a16b-72ab-4cac-a5ac-c3c54894a4c6)

**- For CFB Mode:**
Error Propagation:
CFB mode has a feedback mechanism where the encryption of each block depends on the previous block, which means errors in ciphertext propagate.
Since you modified the 8th byte of the ciphertext, the decryption process will fail to correctly decrypt this byte. Additionally, due to feedback in CFB mode, the error will propagate and affect the decryption of subsequent blocks.
**For OFB Mode:**
Error Propagation:
In OFB mode, the ciphertext is XORed with a series of output blocks generated from the encryption of the IV. This mode does not rely on feedback from previous ciphertext blocks, so an error in one ciphertext block only affects the corresponding plaintext block.
Modifying the 8th byte of the ciphertext in OFB mode will only corrupt the corresponding plaintext byte, with no effect on the subsequent bytes.
Visual Comparison of the Result
CFB Mode: Since CFB uses feedback, the error introduced at the 8th byte will propagate, and the output may show noticeable corruption in multiple adjacent plaintext blocks.
OFB Mode: In contrast, OFB mode's lack of feedback means the corruption in the 8th byte will only affect that specific byte in the decrypted file, and the rest of the plaintext remains unaffected.

### Conclusion
CFB Mode: Modifying one byte in the ciphertext will cause errors to propagate to the subsequent blocks, corrupting multiple plaintext blocks.
OFB Mode: A change in one byte of the ciphertext will only affect that specific byte of the plaintext, with no impact on other blocks.









