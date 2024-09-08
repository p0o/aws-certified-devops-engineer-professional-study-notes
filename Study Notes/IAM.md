
For access to billing, apart form IAM policies you need to activate IAM Access on the Billing console.

### sts:AssumeRole
Calls are made by a user to assume a role for themselves as long as they are in the trust policy of the role.

### iam:PassRole
Allows any user (IAM Principal) with this permission to attach any role to a service such as EC2, Lambda or many more. Exmaple usage with Lambda:
```bash
aws lambda create-function 
    --function-name my-function 
    --runtime nodejs14.x 
    --zip-file fileb://my-function.zip 
    --handler my-function.handler 
    --role arn:aws:iam::123456789012:role/service-role/MyTestFunction-role-tges6bf4
```

This could be a used as a weak-link to elevate access in security breaches. More infromation on that here:

https://tutorialsdojo.com/understanding-the-iampassrole-permission/

### SCPs


> [!NOTE] How they work?
> Remember that if SCP exist it should explicitly Allow the action, otherwise it's an implicit Deny by default


##### Protect specific IAM paths with an SCP

Example: The SCP blocks principals from passing a role unless it has a “team” tag with the value “security” and is in the IAM path /security_app_roles/.

```text
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::*:role/security_app_roles/*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalTag/team": "security"
        }
      }
    }
  ]
}
```

Learn more here:

https://aws.amazon.com/blogs/security/how-to-use-the-passrole-permission-with-iam-roles/


### Difference between Service Role and Service Linked Role

Service roles and service-linked roles differ in their functions and management.  

• **Service Roles:** These are IAM roles that a service assumes to perform actions on your behalf. IAM administrators can create, modify, and delete these roles.

• **Service-Linked Roles:** These are a specific type of service role linked to an AWS service. The service assumes the role to perform actions on your behalf, but IAM administrators can only view, not edit, their permissions.


### IAM Access Analyzer
AWS IAM Access Analyzer helps you identify the resources in your organization and accounts, such as Amazon S3 buckets or IAM roles, that are shared with an external entity. This lets you identify unintended access to your resources and data, which is a security risk.

You can set the scope for the analyzer to an organization or an AWS account. This is your zone of trust. The analyzer scans all of the supported resources within your zone of trust. When Access Analyzer finds a policy that allows access to a resource from outside of your zone of trust, it generates an active finding.


### Permissions Boundaries
Is used to limit the actions of a specific user in terms of how they can use `IAM:*.` . Allowing them to managed themselves and others within a certain boundary set by a higher admin.
Permission boundary itself is a normal IAM Policy that should be protected within IAM policies to avoid restricted users to make changes in it.

### Policy evaluation logic
When an AWS service receives a request from any principal, AWS completes several steps to determine whether to allow or deny the request.

AWS apply Allow or Deny based on this order:

1. Explicit deny
2. SCPs
3. Resource Policy
4. Permission Boundary
5. Session Policy
6. Identity Policy


![[Pasted image 20240824182105.png]]