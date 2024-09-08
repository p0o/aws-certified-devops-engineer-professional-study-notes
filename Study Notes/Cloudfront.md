### Tips
- ACM for Cloudfront should always be in us-east-1
- Self signed certificates do not work with CF

### Field Level Encryption
Is used to provide secure end-to-end connections to origin server by HTTPS. It add an extra layer of security to make sure data is only accessible by a specific instance inside your network (owner of RSA private key)
![[Pasted image 20240801150009.png]]

The following steps provide an overview of setting up field-level encryption. For specific steps, see [Set up field-level encryption](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/field-level-encryption.html#field-level-encryption-setting-up).

1. **Get a public key-private key pair.** You must obtain and add the public key before you start setting up field-level encryption in CloudFront.
    
2. **Create a field-level encryption profile.** Field-level encryption profiles, which you create in CloudFront, define the fields that you want to be encrypted.
    
3. **Create a field-level encryption configuration.** A configuration specifies the profiles to use, based on the content type of the request or a query argument, for encrypting specific data fields. You can also choose the request-forwarding behavior options that you want for different scenarios. For example, you can set the behavior for when the profile name specified by the query argument in a request URL doesn’t exist in CloudFront.
    
4. **Link to a cache behavior.** Link the configuration to a cache behavior for a distribution, to specify when CloudFront should encrypt data.

### Signed URLs and Signed cookies

- In order to create one you need a signer.
- A signer is either a trusted key group or an AWS account that contains a CloudFront key pair (trusted signer).
- Trusted key group is recommended:
	- No need for root user
	- Could be used with API so easier to automate and rotate
	- Could be used with IAM to restrict (for example to upload public keys but not delete them)
	- You can associate higher number of public keys(up to 4 key groups and 5 public key per distribution)
	- 
- Cloudfront Key Pairs is not recommended.
	- Requires root account
	- Could only have 2 active key pairs per AWS account
- With trusted signer you can assign another AWS account as signer

**Reformat the private key (.NET and Java only)**

If you’re using .NET or Java to create signed URLs or signed cookies, you cannot use the private key from your key pair in the default PEM format to create the signature. Instead, do the following:

- **.NET framework** – Convert the private key to the XML format that the .NET framework uses. Several tools are available.
    
- **Java** – Convert the private key to DER format.

### SNI vs Dedicated IP for HTTPS
Read more here: 

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-https-dedicated-ip-or-sni.html


https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#private-content-creating-cloudfront-key-pairs



