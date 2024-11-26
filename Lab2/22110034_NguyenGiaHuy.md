# Lab #2,22110034, Nguyen Gia Huy, INSE331280E_02FIE
# Task 1: Public-key based authentication
**Question 1**:  Implement public-key based authentication step-by-step with openssl according the following scheme. ![alt text](file://C:%5CUsers%5CUser%5CDownloads%5Cimage-1.png?msec=1732583896635)

**Answer 1**:

## 1. Create and configure 2 Docker containers

- First, create 2 containers for the 2 virtual machines of client and server with names `vm_client` and `vm_server`:

```bash
docker run -dit --name vm_client ubuntu bash
docker run -dit --name vm_server ubuntu bash
```

- After creating 2 containers, access each container to install OpenSSL:

```bash
apt update
apt install openssl -y
```

- For convenience of storing keys, create a directory to store keys in each container:

```bash
mkdir /keys
```

## 2. Generate and share Public/Private Keys

*Now generate public and private keys to use during authentication.*

- Generate RSA key pair for `Client`:

  ```bash
  openssl genpkey -algorithm RSA -out /keys/client_private.key
  openssl rsa -pubout -in /keys/client_private.key -out /keys/client_public.key
  ```

- Generate RSA key pair for `Server`:

  ```bash
  openssl genpkey -algorithm RSA -out /keys/server_private.key
  openssl rsa -pubout -in /keys/server_private.key -out /keys/server_public.key
  ```

## 3. `Server` creates and encrypts challenge message

- Creates a challenge message and save it to a `.txt` file inside the `keys` folder:

  ```bash
  echo "Please prove your identity" > /keys/challenge_message.txt
  ```

- Then, encrypt the challenge message with the **client's public key**:

  ```bash
  openssl pkeyutl -encrypt -in /keys/challenge_message.txt -out /keys/encrypted_challenge.bin -pubin -inkey /keys/client_public.key
  ```

## 4. `Client` decrypts and signs the challenge

### 4.1. Decrypt received challenge message with client's private key:

Decrypt the challenge message of the received encrypted file `encrypted_challenge.bin` with the private key `client_private.key` and save it to a `.txt` file:

```bash
openssl pkeyutl -decrypt -inkey /keys/client_private.key -in /keys/encrypted_challenge.bin -out /keys/decrypted_challenge.txt
```

Check the decrypted file to see if the result is correct:

```bash
cat /keys/decrypted_challenge.txt
```

*Result:*

![image-20241126103850566](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20241126103850566.png)

### 4.2. Sign decrypted challenge mesage with client's private key

After decryption, the client signs the decrypted challenge message with its private key:

```bash
openssl dgst -sha256 -sign /keys/client_private.key -out /keys/signed_challenge.bin /keys/decrypted_challenge.txt
```

*The digital signature for the challenge message will then be saved to the file `signed_challenge.bin`. This verifies that the client actually received and understood the message from the server.*

### 4.3. Encrypt the signature with the private key and send it to the server

```bash
openssl pkeyutl -encrypt -inkey /keys/client_private.key -in /keys/signed_challenge.bin -out /keys/encrypted_signed_challenge.bin
```

## 5. Verify signature using client's public key
Decrypt the signature using the client's public key:

```bash
openssl rsautl -decrypt -inkey /keys/client_public.key -pubin -in /keys/encrypted_signed_challenge.bin -out /keys/decrypted_signed_challenge.bin
```

Verify the signature by comparing it with the sent challenge message:

```bash
openssl dgst -sha256 -verify /keys/client_public.key -signature /keys/decrypted_signed_challenge.bin /keys/decrypted_challenge.txt
```