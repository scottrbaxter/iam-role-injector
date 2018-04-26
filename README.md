# iam-role-injector

The IAM Role Injector is a tool for easily assuming an IAM Role with
Multi-Factor Authentication (MFA). It manipulates environment variables
to allow codebases already using AWS credentials to use IAM roles with minimal to no
refactoring. In the same vein, the Role Injector can also be used to help users using the
command line tools to assume a role.

## Assumptions
 - AWS CLI configured correctly, storing 'aws_access_key_id' and
   'aws_secret_access_key' in either environment variables -OR- in
   ~/.aws/credentials
 - One of the following Scenarios apply:

### Scenario One: Federated AWS Accounts
 - At least two AWS Accounts:
   - AWS Account 1 must have a policy that includes sts:AssumeRole to AWS Account 2
   - AWS Account 2 must have a Trust Relationship on a role that references AWS Account 1
 - AWS Account 1 may now assume the role in AWS Account 2 that has the Trust Relationship

### Scenario Two: Single AWS Account
 - An IAM User Account with a policy that include sts:Assume on an IAM
   Role.
 - The IAM Role has a trust relationship that allows entities in the
   account to assume it
 - In this case, AWS Account 1 and AWS Account 2 are the same.

## Installation

1. [Install AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
2. [Configure AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) with required credentials, either as Environment
   Variables or by running 'aws configure'
3. `wget -N https://raw.githubusercontent.com/civisanalytics/iam-role-injector/master/assume_role.sh -O ~/assume_role.sh`

## Increase AWS IAM Role max-session-duration
1 hour is used by default (following the AWS assume-role default), if no timeout time is specified.
To update an AWS IAM role beyond 1 hour using the sts assume-role call, you must initially update the IAM role with a max-session-duration (in seconds).
e.g. (for 12 hours):

`aws iam update-role --role-name administrator --max-session-duration 43200`

## Command Line Usage

`source /path/to/sts_assume_role.sh -d 1234567890 -r administrator`

*source /path/to/sts_assume_role.sh -d [destination_account__number] -r [rolename]*
### Otional features:
#### Help menu
`/path/to/sts_assume_role.sh -h`
#### Show current iam user or role info
`/path/to/sts_assume_role.sh -i`
#### Revert to iam user from role
`/path/to/sts_assume_role.sh -u`
#### Specify expiration (aws sts now supports from 1 hour, up to 12 hours)
`source /path/to/sts_assume_role.sh -d [destination_account__number] -r [rolename] -t 4h`

*works with seconds, minutes, or hours. e.g. `-t 2h` `-t 120m` `-t 7200s` `-t 7200`*

### Variables exported
```
AWS_ACCOUNT_NAME
AWS_ACCESS_KEY_ID
AWS_ENV_VARS
AWS_SECRET_ACCESS_KEY
AWS_SECURITY_TOKEN
AWS_SESSION_TOKEN
AWS_STS_EXPIRATION
AWS_STS_TIMEOUT
AWS_USER
OG_AWS_ACCESS_KEY_ID
OG_AWS_SECRET_ACCESS_KEY
```

### Expiration/Timeout use case
`AWS_STS_EXPIRATION` and `AWS_STS_TIMEOUT` are helpful to create a basic functions to determine how much time until the assume-role has
expired.

```
function timeToSTSExpiration(){
  echo $(( $(( ${AWS_STS_TIMEOUT} - $(date -u +%s) )) / 60 ))
}
```

It can even be added to your prompt. e.g.
```
function checkSTS() {
if [[ -n $AWS_STS_EXPIRATION && $AWS_STS_TIMEOUT -ge $(date +%s) ]]; then
  AWS_STS_MINUTES_REMAINING=$(( $(( AWS_STS_TIMEOUT - $(date -u +%s) )) / 60 ))
  echo "$AWS_ACCOUNT_NAME|$AWS_STS_MINUTES_REMAINING"
elif [[ $AWS_STS_TIMEOUT -le $(date +%s) ]]; then
  unset AWS_STS_EXPIRATION
fi
STS_PROMPT_="$STS_COLOR"'$(checkSTS)'
PROMPT="$STS_PROMPT >"
```

# Antiquated script
```
source ~/assume_role.sh {sourceAccountNumber} {username} {destinationAccountNumber} {rolename}
```

 - `sourceAccountNumber`: AWS Account Number of AWS Account 1
 - `username`: AWS Account 1 username
 - `destinationAccountNumber`: AWS Account Number of AWS Account 2
 - `rolename`: the name of the role to assume in AWS Account 2 that has the Trust Relationship to AWS Account 1

Calling the script with 'source' is required for the
environment variables to persist past the runtime of the script.

The script will also protect your original credentials if you chose to
store *them* as environment variables.

## Bugs

Please report any bugs to:
https://github.com/civisanalytics/iam-role-injector/issues

## Contributing

Open an issue or a pull request if you see how we can improve the
script!


## License

 iam-role-injector is released under the [BSD 3-Clause License](LICENSE.txt).
