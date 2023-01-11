# OIDC Setup for using AWS STS from GitHub Actions

## Step One - Create OIDC Provider in AWS

### Deploy CloudFormation template

I have included [`oidc_provider_cfn.yaml`](oidc_provider_cfn.yaml) as an example Cloudformation template derived from [GitHub's example](https://github.com/aws-actions/configure-aws-credentials#sample-iam-role-cloudformation-template). This can be deployed using the AWS Console or by command line.

```
aws cloudformation deploy \
    --template-file /path/to/oidc_provider_cfn.yaml \
    --stack-name "OIDC-Provider-Example" \
    --capabilities CAPABILITY_NAMED_IAM \
    --parameter-overrides \
      GitHubOrg='<github-user-or-organization-name>' \
      PermissionList='s3:PutObject,s3:ListBucket' \
      RepositoryName='<github-repo-name>' \
      ResourceList='*' \
      RoleName='<any-role-name>'
```

When the stack deploys, grab the full role name from the **Role** output parameter:

  * `arn:aws:iam::1234567891023:role/any-role-name-Username-RepoName-Role` => `any-role-name-Username-RepoName-Role`
 
### Important Note:

It is probably best to use this CF Template as an EXAMPLE, or add a `Condition` to the `AssumeRolePolicyDocument`'s `Principal` (See [GitHub's Example](OIDCProvider)) to use a passed in an existing `AWS::IAM::OIDCProvider` Arn since `OIDCProvider` URL's must be unique in every account. Rather than sharing roles across projects/repos, its probably better to add additional IAM roles / Policies for each project, and instead refer to the unique `OIDCProviderOIDCProvider`.

## Step Two - Add `.github/workflows/workflow.yml`

See [`oidc_example.yml`](.github/workflows/oidc_example.yml) for an example. 

### Important bits:

#### OIDC Token endpoint

You need to give the workflow permissions to access GH's OIDC Token endpoint. This part is easily missed (Speaking from experience!)

```yaml
    # These permissions are needed to interact with GitHub's OIDC Token endpoint.   
    permissions:
      id-token: write
      contents: read   
```

#### Getting STS Credentials in Workflow

I stored the AWS Account Number and Role name as Action Variables for opaqueness in the repo.

```yaml
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::${{vars.AWS_ACCOUNT_NUMBER}}:role/${{vars.AWS_IAM_ROLE_NAME}}
          aws-region: ${{env.AWS_DEFAULT_REGION}}
```
