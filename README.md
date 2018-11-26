# OpenSSL-Enc-By-Example
I didnt see many resources on the interwebs on how to use the OpenSSL-ENC commands so I am creating one here. 
Here are some examples of how to use openssl-enc for symmetric cipher encryption and decryption on Kali linux.  My commands and research are based on the openssl-enc man page which you can access with: `man enc`

Symmetric ciphers like AES (and even DES) frequently come up during penetration testing or CTF challenges so its helpful to have a quick one-liner reference like this page for quickly dealing with them.

## What is Openssl-enc?  
OpenSSL comes with a handy utility that provides symmetric cipher commands for encryption or decryption of vartious block and stream ciphers. For convience zlib compression and Base64 encoding can also be performed on data by the OpenSSL-enc utility.

## What Cipher are support by OpenSSL-ENC?
At the time of writing this post, OpenSSL-ENC supported 111 ciphers.  
```
root@kali:~# openssl enc -ciphers
Supported ciphers:
-aes-128-cbc               -aes-128-cfb               -aes-128-cfb1             
-aes-128-cfb8              -aes-128-ctr               -aes-128-ecb              
-aes-128-ofb               -aes-192-cbc               -aes-192-cfb              
-aes-192-cfb1              -aes-192-cfb8              -aes-192-ctr              
-aes-192-ecb               -aes-192-ofb               -aes-256-cbc              
-aes-256-cfb               -aes-256-cfb1              -aes-256-cfb8             
-aes-256-ctr               -aes-256-ecb               -aes-256-ofb              
-aes128                    -aes128-wrap               -aes192                   
-aes192-wrap               -aes256                    -aes256-wrap              
-bf                        -bf-cbc                    -bf-cfb                   
-bf-ecb                    -bf-ofb                    -blowfish                 
-camellia-128-cbc          -camellia-128-cfb          -camellia-128-cfb1        
-camellia-128-cfb8         -camellia-128-ctr          -camellia-128-ecb         
-camellia-128-ofb          -camellia-192-cbc          -camellia-192-cfb         
-camellia-192-cfb1         -camellia-192-cfb8         -camellia-192-ctr         
-camellia-192-ecb          -camellia-192-ofb          -camellia-256-cbc         
-camellia-256-cfb          -camellia-256-cfb1         -camellia-256-cfb8        
-camellia-256-ctr          -camellia-256-ecb          -camellia-256-ofb         
-camellia128               -camellia192               -camellia256              
-cast                      -cast-cbc                  -cast5-cbc                
-cast5-cfb                 -cast5-ecb                 -cast5-ofb                
-chacha20                  -des                       -des-cbc                  
-des-cfb                   -des-cfb1                  -des-cfb8                 
-des-ecb                   -des-ede                   -des-ede-cbc              
-des-ede-cfb               -des-ede-ecb               -des-ede-ofb              
-des-ede3                  -des-ede3-cbc              -des-ede3-cfb             
-des-ede3-cfb1             -des-ede3-cfb8             -des-ede3-ecb             
-des-ede3-ofb              -des-ofb                   -des3                     
-des3-wrap                 -desx                      -desx-cbc                 
-id-aes128-wrap            -id-aes128-wrap-pad        -id-aes192-wrap           
-id-aes192-wrap-pad        -id-aes256-wrap            -id-aes256-wrap-pad       
-id-smime-alg-CMS3DESwrap  -rc2                       -rc2-128                  
-rc2-40                    -rc2-40-cbc                -rc2-64                   
-rc2-64-cbc                -rc2-cbc                   -rc2-cfb                  
-rc2-ecb                   -rc2-ofb                   -rc4                      
-rc4-40                    -seed                      -seed-cbc                 
-seed-cfb                  -seed-ecb                  -seed-ofb            
```


## Encrypting Data with AES
The Advanced Encryption Standard (AES), is a block cipher adopted as an encryption standard by the U.S. government for military and government use. There are a few different cipher methods available for use in AES, the most commonly used ones are ECB and CBC.  

ECB (Electronic Codebook) is essentially the first generation of the AES. It is the most basic form of block cipher encryption.  

CBC (Cipher Blocker Chaining) is an advanced form of block cipher encryption. With CBC mode encryption, each ciphertext block is dependent on all plaintext blocks processed up to that point. This adds an extra level of complexity to the encrypted data. 

We are going to use CBC for this example which requires both a Key and an Initialization Vector (IV). An initialization vector is a random number used in combination with a secret key as a means to encrypt data. This number is sometimes referred to as a nonce, or "number occuring once" as an encryption program uses it only once per session.  The Key and IV need to be passed to the OpenSSL-ENC command in hex format.  The in AES-128, the key MUST have a length that is a multiple of 16, but the IV can be any length as it is really more like an inital offset value.

```
root@kali:~# echo -n 'thiskeyis16chars' | od -A n -t x1 |sed 's/ //g'
746869736b6579697331366368617273
root@kali:~# echo -n 'mynonrandomiv' | od -A n -t x1 |sed 's/ //g'
6d796e6f6e72616e646f6d6976
```
We now have our Key in HEX value: **746869736b6579697331366368617273**  
And our IV in HEX value: **6d796e6f6e72616e646f6d6976**  
So lets use them to encrypt a secret message: **mysecretmessage**  

Here is a quick test of the Openssl-enc command for encrypting data using AES128:
```
root@kali:~# echo mysecretmessage | openssl enc -e -aes-128-cbc -K 746869736b6579697331366368617273 -iv 6d796e6f6e72616e646f6d6976 -nosalt -nopad -base64 -p
key=746869736B6579697331366368617273
iv =6D796E6F6E72616E646F6D6976000000
Js9rW0xBiQURttTdWSPqVg==
```

The message was encrypted and returned the cipher text in a base64 format as: **Js9rW0xBiQURttTdWSPqVg==**

Now lets use our key and IV to decrypt this message:
```
root@kali:~# echo Js9rW0xBiQURttTdWSPqVg== | openssl enc -d -aes-128-cbc -K 746869736b6579697331366368617273 -iv 6d796e6f6e72616e646f6d6976 -nopad -nosalt -base64 -p
key=746869736B6579697331366368617273
iv =6D796E6F6E72616E646F6D6976000000
mysecretmessage
```
Our base64 cipher text has now been decrypted to the plain-text value: **mysecretmessage**
