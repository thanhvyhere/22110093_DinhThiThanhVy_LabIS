
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
Alice (server): IP 10.9.0.5
Bob (client): IP 10.9.0.6
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
3.1 Grant Permissions
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
openssl rand -hex 32 > randompassword

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

**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:







