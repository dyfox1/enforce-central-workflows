# enforce-central-workflows
Sample setup for enforcing the use of centralized workflows when deploying to AWS.

In this simple example, the calling workflow and the re-usable workflow are in the same repo.  This is not a requirement for this pattern as outlined here in the GitHub documentation: https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow.

In practice, the calling workflow would live in a product or workload team's repository and the re-usable workflow would be hosted in a centrally managed repository.

# Setup

To set up our test deployment role, we deploy the `github-assume-role-stack.yaml` via the AWS cli or the AWS console.

Some parameters are required to use this stack and they are as follows:

```
Parameters:
  github_org_name:
    Type: String
    Description: The GitHub Organization that holds our repo(s)
  calling_repo_name:
    Type: String
    Description: The name of the repo that will call our re-usable workflow
  central_repo_name:
    Type: String
    Description: The name of the repo that will host our centrally managed workflow
  reusable_workflow_file:
    Type: String
    Description: The name of our re-usable workflow file (e.g. workflow.yaml)
```

These can be subbed in as follows using example values relevant to this repo:

```
aws cloudformation create-stack \
    --stack-name test-github-role-stack \
    --template-body file://github-assume-role-stack.yaml \
    --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
    --parameters file://github-assume-role-stack-parameters.json \
    --region ca-central-1
```