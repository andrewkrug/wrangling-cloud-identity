# Identity Risk with Static Analysis

Static Analysis gives you a point in time snapshot of what is 
really happening in your policy.

## The advantages of running a static analyzer are:

* Speed 
* No credentials are required

## The disadvantages of static analysis are:

* It's only as good as the signature
* No additional context from the environment 

In this lab you will run Semgrep and IAMCTL. Semgrep is a static analysis tool that has both open source and paid components to identify patterns in JSON, Cloudformation, and Terraform. 

IAMCTL is a command line tool that can be used to identify risky permissions or compare permissions across accounts.

## What is static analysis most useful for? 

Static Analysis is most useful for identifying specific IAM verbs or patterns that we want to actively avoid. 

## Lab Instructions 

1. Setup your credentials by copy+pasting them into your shell
2. Install [IAMCTL](https://github.com/aws-samples/aws-iamctl) `pip install git+https://github.com/BogdanSorlea/aws-iamctl@fix_aws-samples_issue6`
> Note: This branch addresses a bug in python3.10+ support this can be moved pack from this workaround as soon as the branch merges
3. Download all of the IAM Policies as JSON that have been deployed in the IAM Sandbox environment 

```bash

 ____    __    __  __  ___  ____  __
(_  _)  /__\  (  \/  )/ __)(_  _)(  )
 _)(_  /(__)\  )    (( (__   )(   )(__
(____)(__)(__)(_/\/\_)\___) (__) (____)


Harvesting IAM Roles from workshop █████████████████████∙∙∙∙∙∙∙∙∙∙∙ 43/64 - 13s
```

You should observe the gathering of 64 roles

This produces a CSV with all of the role arns and actions.

4. Use grep, excel, or a tool of your choice to identify roles with dangerous permissions.  

5. If you want to progress this to a set of detections you could run in CI/CD the first step is to make it work locally.
Install the semgrep cli and give this command a try from within this directory:
`semgrep scan --config=./rules.yaml .`

If for some reason you can not install semgrep locally there is a web version that will allow you to try the same thing
inside of the web [sandbox](https://semgrep.dev/playground/r/vdT2ED/terraform.aws.security.aws-iam-admin-policy.aws-iam-admin-policy?view=code&editorMode=advanced).

If successful you should see the following:
```bash
┌─────────────────┐
│ 4 Code Findings │
└─────────────────┘

    example-policy.tf
   ❯❯❱ aws-iam-admin-policy
          Detected admin access granted in your policy. This means anyone with this policy can perform
          administrative actions. Instead, limit actions and resources to what you need according to least
          privilege.

           39┆   policy = <<POLICY
           40┆ {
           41┆   "Statement": [
           42┆     {
           43┆       "Action": [
           44┆         "s3:HeadBucket",
           45┆         "*"
           46┆       ],
           47┆       "Effect": "Allow",
           48┆       "Resource": [
             [hid 10 additional lines, adjust with --max-lines-per-finding]
```

These tactics are the foundation for basic CI/CD checks. Anything that you can write as a signature you can use in a pre-commit or in ci as a gate to prevent dangerous policies from progressing in the software development lifecycle.
