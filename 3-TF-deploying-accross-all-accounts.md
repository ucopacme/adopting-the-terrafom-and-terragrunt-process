# Implimenting terragrunt code accross all support CloudHosting AWS accounts.

The pieces in play are as followed:




## 1) Ensure the link to your code is already located in the 'accounts.ts' file.

The accounts.ts file is located here:

```
ucop-terraform-config/terraform/bin/util/src/accounts.ts

```
The links are the basic path to your terragrunt code.

If you require to add a new link to be added, you add it here within the 'accounts.ts' file.

```
grep 'cloudwatch' accounts.ts
    choices: ['', 'cloudwatch/rules/local-iam/terragrunt.hcl','lambda/check-user/terragrunt.hcl','INSERT NEW PATH HERE'],
      'list of links to create, comma delimited list with no spaces, "cloudwatch/rules/local-iam/terragrunt.hcl,cloudwatch/rules/root-access/terragrunt.hcl"'
```

## 2) what does this link actually do?

Example: let's looks at choice: cloudwatch/rules/local-iam/terragrunt.hcl

starting from the root of all accounts..
```
its-big-prod     its-fdw-dev      its-iso-prod     its-ppers-poc    its-seg-log      its-uc-tes       links
common_vars.json its-bigri-prod   its-fdw-prod     its-ldc          its-redline-dev  its-seg-master   its-ucop-setdev  template

─➤  ls links/cloudwatch/rules/local-iam/terragrunt.hcl
links/cloudwatch/rules/local-iam/terragrunt.hcl
```
The above link is a hard link to the terragrunt file needed to install the lambda function 'check-user'


If you make a change to : links/cloudwatch/rules/local-iam/terragrunt.hcl and then you re-run the code in any account this terragrunt code is installed it will pick up the changes.


# New implimentations of terragrunt code

If you intend to setup a new piece of code.


# The walk-thru:


## 3) Ensure all the accounts you require to run your code  in exist in the master branch, if not create any missing accounts.

This is how you create any missing accounts:

From the root of the repository:

-  update the common_vars.json file and add your new account under the accounts section.

Add your piece of code as like the following:
```
        "env": "prod",
        "id": "012345678901",
        "name": "myaccount",
        "owner": "john.doe@gmail.com",
        "role": "role/myrole"
    }
```



- Run the following commands. 
```

# Readonly - will not create

node bin/util/build/src/accounts.js -a myaccount
 - will report only to STDOUT

# Will create directory and underlying files.

node bin/util/build/src/accounts.js -a myaccount -d false
 - will report to STDOUT and create the following underlying dir/files.

%  ls -la myaccount
total 16
drwxr-xr-x@  5   staff   160 Sep 30 15:23 .
drwxr-xr-x@ 45   staff  1440 Sep 30 15:23 ..
-rw-r--r--   1   staff   280 Sep 30 15:23 account.hcl
-rw-r--r--   1   staff    37 Sep 30 15:23 common.tfvars
drwxr-xr-x@  4   staff   128 Sep 30 15:23 us-west-2

Note: it defaults the first region as US-WEST-2, which is our default region.

```

As of now you have the basic requirements needed to start adding terragrunt code. 


# 4) Adding your piece of terragrunt code to a new account:

The process to adding your new terragrunt code to an account is as follows:

From the root directory:

## Doing a single account:
```
node bin/util/build/src/accounts.js  -l lambda/check-user/terragrunt.hcl -a  myaccount -r us-west-2 -d false

%ls myaccount/us-west-2/lambda/check-user/terragrunt.hcl
myaccount/us-west-2/lambda/check-user/terragrunt.hcl

you now see your terragrunt file..you are ready to run it.
```

## Blasting your code to more than one account:

- Run the following command:
- NOTE: if you use the -a (it takes regex expressions, so for account 'myaccount', I could of put 'mya' and it would of cought it.
- NOTE: you can use the -o option if you want to install your code on a particular owners accounts. it too takes regex expression, EG: 'weis' will pick up weisbrods accounts.
- NOTE: you must change the -l, -r, -a , -o parameters to what you require.
- NOTE: you must also change what what terragrunt does, an (init, plan, apply -auto-approve) 
```
for i in `node bin/util/build/src/accounts.js  -l lambda/check-user/terragrunt.hcl -a myaccount  -r us-west-2 | jq -r '.[].name'` 
do
echo "pushd $i/us-west-2/lambda/check-user && pwd &&  ls -l && terragrunt init && terragrunt plan  && popd"
done |parallel -j0 --result /tmp/terragrunt
```
