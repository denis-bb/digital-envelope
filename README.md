# Digital envelope

This script encrypts files or pipe with symmetric algorithms with unique session key and put the session key encrypted with a asymmetric algorithm to that file/pipe.

Also script decrypt pipe/file that was encrypted by it.

It uses OpenSSL for encryption and can do self-extracted files.

## Use case

There are many use cases for this script can be. But I use it on the server side when creating automated backups, encrypt its and put them to external storage. In this case you must provide a password for archive tools and this password must be stored on server as plain text. So an attacker can compromise your password on the server and access to your external storage. After that he can get extract information from your backups without access you internal server as long as he want.

The main problem that motivates me to wrote this script is a plain text password stored on server. I want to make simple tool that provide ability encrypt files with asymmetric algorithm and store on server only public key or public key with encrypted private key.

## Usage
```
>$ ./envelope 

Usage: envelope [options]

Options:

  -h|--help              show this help
  -v|--version           show version info
                         
  --create               create envelope
  --extract              extract file from envelope
                         
  --envelope <file>      Creating file, if not set the STDIN/STDOUT will be used
  --in <file             Input file. Default stdin 
  --out <file>           Out file. Default stdout
                         
  --public-key <file>    Public key for creating envelope
  --private-key <file>   Private key for self extracting archive (must be encrypted when creating sfx envelope)
  --password <password>  Private key password for decryption
                         
  --sfx                  create self extracting archive
  --encrypt-alg          Encryption algotithm. Default aes-256-cbc
 
```

### Key creation
#### Private 
```
>$ openssl genrsa --des3 --out test.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.......................................................................+++++
...................................................................................+++++
e is 65537 (0x010001)
Enter pass phrase for test.key:
Verifying - Enter pass phrase for test.key:
```
In this example 2048 bit ``test.key`` will be created and encrypted DES3 with the provided password.

> Please be carefully when encrypt small files. Symmetric key will be encrypted with RSA algorithm that can't encrypt block smaller than some size. That size depends from RSA key size. So for every encrypted file encrypted symmetric key will be added and size overhead will be occurred (for example over 500 bytes for 4096 keys). For files less than 1Kbyte it can be larger than the file size.

#### Public
```
>$ openssl rsa -in test.key -pubout -out test.pub
Enter pass phrase for test.key:
writing RSA key
```

### Test that it's work
```
echo -e "\n\tWorked like a charm\!\n"|./envelope --create --public-key test.pub  |./envelope --extract --private-key test.key
Enter pass phrase for /dev/fd/63:

	Worked like a charm\!

```

### Create self extracted archive
```
tar -cj test*|./envelope --create --sfx --public-key test.pub --private-key test.key > backup.tar.gz.enc
```
### ...and extract it
```
bash ./backup.tar.gz.enc|tar -xj --directory /tmp
```
