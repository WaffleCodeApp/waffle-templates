# Setting up AWS

Requirements:
- An AWS account

This doc assumes that you are logged in as a root user, and have nothing else set up.

Below the following are covered:
- Enabling logging for account level security
- Creating users
- Creating an AWS subaccount to deploy to


If you have things already set up, then you probably know how to adapt all this to your situtaion, so it still might be valuable to keep readin.

## Enable CloudTrail for logging operations

CloudTrail logs changes in the account, which is important for security and compliance purposes, so it's helpful to set it up as soon as possible. Storing the logs have a monthly fee though.

Go to `CloudTrail` on the AWS Console and push `Create a trail`, it's OK to use the default settings.

## Creating user(s)

It's highly recommended not to use your AWS account with you root access. You can create a user with the least required permissions in the `AWS Console > IAM`.

Once you created a user that you'll use for deploying the templates in this repo, it's strongly recommended to turn on MFA for them. The users can turn this on for themselves.

## Requiring MFA for your user(s)

Once the newly added users have the MFA set and working, it's time to add restrictive policies to have to use it every time.

Policies for requiring MFA:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "EnableUsersManageTheirPassword",
            "Effect": "Allow",
            "Action": [
                "iam:ChangePassword",
                "iam:GetAccountPasswordPolicy"
            ],
            "Resource": "arn:aws:iam::123456789012:user/${aws:username}"
        },
        {
            "Sid": "EnableUsersManageTheirAccessKeys",
            "Effect": "Allow",
            "Action": [
                "iam:CreateAccessKey",
                "iam:ListAccessKeys",
                "iam:DeleteAccessKey"
            ],
            "Resource": "arn:aws:iam::123456789012:user/${aws:username}"
        },
        {
            "Sid": "EnableUsersManageTheirLoginProfiles",
            "Effect": "Allow",
            "Action": [
                "iam:CreateLoginProfile",
                "iam:GetLoginProfile",
                "iam:DeleteLoginProfile",
                "iam:UpdateLoginProfile"
            ],
            "Resource": "arn:aws:iam::123456789012:user/${aws:username}"
        },
        {
            "Sid": "EnableUsersManageTheirSigningCertificates",
            "Effect": "Allow",
            "Action": [
                "iam:ListSigningCertificates",
                "iam:DeleteSigningCertificate",
                "iam:UpdateSigningCertificate",
                "iam:UploadSigningCertificate"
            ],
            "Resource": "arn:aws:iam::123456789012:user/${aws:username}"
        },
        {
            "Sid": "EnableUsersManageTheirSSHPublicKeys",
            "Effect": "Allow",
            "Action": [
                "iam:ListSSHPublicKeys",
                "iam:GetSSHPublicKey",
                "iam:DeleteSSHPublicKey",
                "iam:UpdateSSHPublicKey",
                "iam:UploadSSHPublicKey"
            ],
            "Resource": "arn:aws:iam::123456789012:user/${aws:username}"
        },
        {
            "Sid": "EnableUsersManageTheirMFA",
            "Effect": "Allow",
            "Action": [
                "iam:ListMFADevices",
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice"
            ],
            "Resource": [
                "arn:aws:iam::123456789012:user/${aws:username}",
                "arn:aws:iam::123456789012:mfa/${aws:username}"
            ]
        },
        {
            "Sid": "EnableUsersBasicIAMOperations",
            "Effect": "Allow",
            "Action": [
                "iam:ListAccountAliases",
                "iam:ListUsers",
                "iam:ListServiceSpecificCredentials",
                "iam:GetAccountSummary"
            ],
            "Resource": "arn:aws:iam::123456789012:*"
        },
        {
            "Sid": "EnableUsersBasicSTSOperations",
            "Effect": "Allow",
            "Action": [
                "sts:GetSessionToken"
            ],
            "Resource": "arn:aws:sts::123456789012:*"
        },
        {
            "Sid": "DenyAccessWithoutMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:ChangePassword",
                "iam:GetAccountPasswordPolicy",
                "iam:CreateAccessKey",
                "iam:DeleteAccessKey",
                "iam:CreateLoginProfile",
                "iam:GetLoginProfile",
                "iam:DeleteLoginProfile",
                "iam:UpdateLoginProfile",
                "iam:ListSigningCertificates",
                "iam:DeleteSigningCertificate",
                "iam:DeleteSSHPublicKey",
                "iam:ListMFADevices",
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

Where `123456789012` is the number of your root account.

I'd highly recommend denying any other permissions for the user on the root account. For example if the top level DSNs is registered in the root account's Route63 service, then it might be useful to enable the `AmazonRoute53FullAccess` to the new user.

## Deploying to a sub-account

It's a very simple and straightfoward way of handling service-level perimissions in different SDLC stages to deploy the related environments into different AWS accounts.

This can easily achived without additional costs by using the `AWS Organizations > AWS Accounts > Add an AWS account` service on the AWS console.

You can explicitly assign policies to your users to enable them to switch into the new account when they log in to the root AWS account as a user.

Policies for switching to the newly added account:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::210987654321:role/OrganizationAccountAccessRole",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
```

Where `210987654321` is the number of your new account, and `OrganizationAccountAccessRole` is the rolename you chose at account creation.


Once this is done, sign in with the new user and switch roles to the new account.

