# Shifting Left

In this lab we'll look at two new command line tools that are available to add to CI/CD pipelines.  You can of course run tests "pre-build" in CI using providers like Codepipeline, Codebuild, Github Actions, and more.  For the purpose of this lab we're going to simulate that locally to learn what checks might be useful to add in CI/CD and how we can use those to reduce developer friction and shorten feedback loops.

> **Pro tip**: the easier it is to run your CI/CD steps locally the more time you'll save adding and tuning checks. Developers want to do the right thing when deploying code so help them shorten feedback loops whenever possible. 

## Lab Prerequisites

For this lab we're going to use several tools.  You'll need to download Visual Studio Code and install it for your operating system. [Download it here](https://code.visualstudio.com/)

You'll also need to ensure you have the latest version of the AWS CLI setup with a pair of credentials for this lab. That's available [here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

It is **highly reccomended** that you do this on *nix or MacOS vs Windows as the lab was built on MacOS and has not been tested on Windows.  

## Lab Instructions

1. Create a directory in your home folder called workspace by typing in `mkdir -p ~/workspace`
2. Clone the lab materials from this repository `cd ~/workspace && git clone https://github.com/andrewkrug/wrangling-cloud-identity.git --depth 1`
3. Open "wrangling-cloud-identity" in Visual Studio Code
4. Configure your credentials on the CLI for the target environment. **Note:** *If this is self paced you should use a set of read only credentials setup as the default profile*
5. Install the AWS IAM Validator `pip install cfn-policy-validator`
6. Install the [IAMLegend](https://marketplace.visualstudio.com/items?itemName=SebastianBille.iam-legend) VSCode Extension
7. While authenticated, run the policy validator against the iam-role.yml file `cfn-policy-validator validate --template-path ./iam-role.yml --region us-west-2`

You should see the following:

```bash
cfn-policy-validator validate --template-path ./iam-role.yml --region us-west-2
{
    "BlockingFindings": [],
    "NonBlockingFindings": []
}
```

8. Now let's check for public access against the s3 bucket stack
`cfn-policy-validator check-no-public-access --template-path ./s3-stack.yml --region us-west-2`

You should see the following:

```bash
cfn-policy-validator check-no-public-access --template-path ./s3-stack.yml --region us-west-2
{
    "BlockingFindings": [
        {
            "findingType": "SECURITY_WARNING",
            "code": "policy-analysis-CheckNoPublicAccess",
            "message": "The resource policy grants public access for the given resource type.",
            "resourceName": "AnalyticsBucket",
            "policyName": "BucketPolicy",
            "details": {
                "result": "FAIL",
                "message": "The resource policy grants public access for the given resource type.",
                "reasons": [
                    {
                        "description": "Public access granted in the following statement with sid: GrantReportAccess.",
                        "statementIndex": 0,
                        "statementId": "GrantReportAccess"
                    }
                ]
            }
        }
    ],
    "NonBlockingFindings": []
}
```

9. Now let's try adding a new substantial and dangerous permission.  Run the command

Note that this policy has a _tricky_ way this access was included that might bypass a static check.

```cfn-policy-validator check-access-not-granted --template-path ./iam-role-new.yml --region us-west-2 --actions "iam:CreateAccessKey"```

You should see the following:

```bash

cfn-policy-validator check-access-not-granted --template-path ./iam-role-new.yml --region us-west-2 --actions "iam:CreateAccessKey"

{
    "BlockingFindings": [
        {
            "findingType": "SECURITY_WARNING",
            "code": "policy-analysis-CheckAccessNotGranted",
            "message": "The policy document grants access to perform one or more of the listed actions.",
            "resourceName": "DeliveryRole",
            "policyName": "DeliveryPolicy",
            "details": {
                "result": "FAIL",
                "reasons": [
                    {
                        "description": "One or more of the listed actions in the statement with index: 2.",
                        "statementIndex": 2,
                        "accessInput": [
                            {
                                "actions": [
                                    "iam:CreateAccessKey"
                                ]
                            }
                        ]
                    }
                ],
                "message": "The policy document grants access to perform one or more of the listed actions."
            }
        }
    ],
    "NonBlockingFindings": []
}
```
