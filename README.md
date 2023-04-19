# enforce-central-workflows
Sample setup for enforcing the use of centralized workflows when deploying to AWS.

In this simple example, the calling workflow and the re-usable workflow are in the same repo.  This is not a requirement for this pattern as outlined here in the GitHub documentation: https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow.

In practice, the calling workflow would live in a product or workload team's repository and the re-usable workflow would be hosted in a centrally managed repository.

# Setting up on the GitHub side
Clone this repo and then run the following GitHub CLI command to customize the OIDC claims that GitHub will send when assuming roles on AWS.

More info on this command, including the permissions required for completing the call is available here: https://docs.github.com/en/rest/actions/oidc?apiVersion=2022-11-28#set-the-customization-template-for-an-oidc-subject-claim-for-a-repository
```
curl -L \
  -X PUT \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR_TOKEN>"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/dyfox1/enforce-central-workflows/actions/oidc/customization/sub \
  -d '{"use_default":false,"include_claim_keys":["repo","job_workflow_ref"]}'
```

This step only needs to be performed once on a repo and can also be performed once for the entire org if needed.  If you are constructing repos via Terraform, you can set this property via this terraform resource: https://registry.terraform.io/providers/integrations/github/latest/docs/resources/actions_repository_oidc_subject_claim_customization_template.

# Setting up on the AWS side

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

The output from this stack is a role arn.  Replace the text `<your_role_arn_here>` in the `.github/workflows/caller-workflow.yaml` file in this repo once you've deployed the AWS stack into your target account.