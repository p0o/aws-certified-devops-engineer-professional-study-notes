
## Types

- Customer managed CMK
- KMS Managed CMK
- AWS owned CMK
## Envelope Encryption
Is the practice of encrypting plaintextdata with data key and encrypting the data key with another key. You could chain this to multiple levels but in the end you would need one root key to decrypt your data. 

![Envelope encryption with multiple key encryption keys](https://docs.aws.amazon.com/images/kms/latest/developerguide/images/key-hierarchy-kms-key.png)


You need envelop encryption for encrypting data bigger than 4 KB since CMK does not allow that

## API Commands

- Encrypt
	- convert plaintext to cyphertext up to 4 KB
- Decrypt
	- converts cihpertext to plaintext using CMK
- GenerateDataKey
	- Generate and returns data key as plaintext and cipher version
	- You can use the plaintext version to encrypt data with a crypto library
- GenerateDataKeyWithoutPlaintext
	- Generate and Returns only ciphertext data key