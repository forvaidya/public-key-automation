# public-key-automation

## Background
SSH (openssh) based logins can automated using Private / Public Key pair. Private Key is in possesion of Owner and public key is saved in ~/.ssh/authorized_keys.

Example:
A user *mahesh* needs an access of a build machine *release@build_mahcine (172.10.1.34)*. 
Mahesh will generate a ssh key pair and hand over public key to administraor of *release@build_mahcine* to append it to ~/.ssh/authorized_keys

## Use Case 
This process of adding ssh public key is manual. We can use a feature AuthorizedKeysCommand to automate.

## Solution
+ save your public keys in some place like github. 
+ collect public keys from users by way of pull request and add it time to time
+ each user can have 1 key at a time and it is named after that user.

In OpenSsh's sshd_config there is AuthorizedKeysCommand, note it is run by user root. So make sure to add verified and trusted code.
```
# /etc/ssh/sshd_config
AuthorizedKeysCommand /path/to/public-keys-collect-and-install
AuthorizedKeysCommandUser root
```

A template AuthorizedKeysCommand.txt is provided to be added to script ```public-keys-collect-and-install```

In this template everytime SSH login is attempted Git clone is refreshed, ~/.ssh/authorized_keys is removed and recreated based on content in git clone for respective SSH_REMOTE_USER

## Drawbacks
+ This command is run as root so make sure to execute trusted code in ```public-keys-collect-and-install```
+ I have tested in github and there may be a lag between of git push and reflecting that change after git pull on SSH Server. 
+ This can make SSH Connectivity slow as there is IO operation, like pulling all Public Keys from backend like S3 or Gihub.

## Race Condition 
There is a possibility that 2 or more process of this script are running and latest run will win.
To avoid this a lock object can be created in some backend like S3, mySQL, DynamoDB etc (just like terraform)

## Possible Automation
A workflow can be developed such that a user commits SSH public key in github and opens a PR. Administrator approves or disapproves PR and keys will be available on master branch and will grant an access

For offboarding - develop a workflow which can remove ssh public keys from github master (main) branch

