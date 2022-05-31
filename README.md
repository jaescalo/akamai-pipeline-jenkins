# Akamai-Pipeline-CICD

This is a demo on how to manage existing Akamai properties as code by leveraging "Akamai Pipeline" CLI framework which allows to:

* Break an existing property into JSON snippets
* Parameterize the builds for the different environments by using the variables JSON files.
* Merge the changes to build the final rule tree. 

After the rule tree is built it will be pushed to Akamai and activated using the Akamai Property CLI commands (more flexibility for CI/CD) instead of the Akamai CLI pipeline promote commands.

## Prerequisites
- [Akamai API Credentials](https://developer.akamai.com/getting-started/edgegrid) for creating propertie, hostnames and CP codes.
- [Akamai Property Manager CLI](https://github.com/akamai/cli-property-manager)
- Go through the [Akamai Pipeline Runbook](https://developer.akamai.com/resource/whitepaper/akamai-pipeline-cli-framework-runbook/direct)

There are two main subjects: Property Preparation for Akamai as Code and GitLab CI/CD Configuration.

## Prepare Properties for Akamai as Code
Perhaps the most important step is to prepare the target properties for management via the pipeline framework.

* Pipeline should be based on a production property (i.e. www.example.com NOT qa.example.com)
* If desired, perform a configuration clean-up (i.e. remove duplicated rules/behaviors, parameterize behaviors/matches, etc) which can help reduce the code length.
* All advanced rules and matches need to be converted first to [Akamai Custom Behaviors](https://developer.akamai.com/blog/2018/04/26/custom-behaviors-property-manager-papi) or to the Property Manager behaviors/matches if possible. If there is an advanced override section this can also be converted to a custom advanced override.
* The Akamai pipeline framework will create build artifacts for all stages using a common template metadata. For this to work effectively, all rules and behaviors must be normalized across each environment so that they work consistently. If there are lower level environment differences these can be added under a hostname match.

### Akamai Pipeline Setup
The main idea is to build a template property that will be used for all the environments. Once a property is ready for "Akamai as Code" it can be used as a template. It is a good idea to make your production property the template property.

These steps should only need to be executed once when you're converting a property to code or when you're adding new environments to the pipeline.

1. Create your Akamai API credentials and install the Akamai Property Manager CLI.
2. Create a local pipeline referring to the template property. You can give any name to the pipeline and at this point is not necessary to specify all the environments, these can be added later. This command will actually create new properties, but these can be deleted later.
```
$ akamai pipeline new-pipeline -p gitlab-pipeline-demo -e <propertyId> prod -g <groupId> --variable-mode user-var-value
```
3. At this point there will be multiple files and a folder structure created locally. Edits are recommended to add more environments and to clean up some of the files.
4. To add more environments (i.e. stage, test, dev, qa) add them to the /projectInfo.json file, plus only the `environments` and `name` key are needed and the rest can be deleted. For example adding the "dev" environment:
```
{
    "environments": [
        "prod",
        "dev"
    ],
    "name": "gitlab-pipeline-demo"
}
```
5. For the newly added environment in the (pipeline)/environments/(dev)/ directory, edit the `envInfo.json` file and clean it up. Only keep the name key which specifies the environment name.
```
{
    "name": "dev"
} 
```
6. Because for an existing property it is rare to add/remove hostname the `hostnames.json` files can be deleted. These are located inside each environment folder.
7. Add any pipeline variables as needed through the JSON variables files to parameterize the different environments.
8. Build the rule tree file by submiting the merge command.
```
$ akamai pipeline merge  -n -v -p gitlab-pipeline-demo dev
```
Our file repository is set up! These can be uploaded to GitLab now and managed with CI/CD.

## Akamai CI/CD Setup in GitLab
This is a simple example that leverages the akamai/shell Docker container to build the .edgerc file in one job and execute the pipeline cli in another job. Check the `.gitlab-ci.yml` for more clarification on the following steps (the `--accountSwitchKey` flag can be disregarded as it is only for managing multiple accounts with one API client). In summary:

1. Store The Akamai {OPEN} API credentials in the project variables in GitLab to keep them masked.
2. A developer makes changes to JSON code and commits is to the repository. Git branching and collaboration concepts apply here.
4. Akamai CLI builds the rule file from the json snippets and variables
```
$ akamai pipeline merge  -n -v -p gitlab-pipeline-demo dev
```
5. Akamai CLI creates a new version of the property and updates it with the local rule tree file
```
$ akamai property-manager property-update -p <property-name> --file ./gitlab-pipeline-demo/dist/dev.gitlab-pipeline-demo message "Created By GitLab-$CI_JOB_NAME-$CI_JOB_ID; Commit $CI_COMMIT_REF_NAME"
```
6. Akamai CLI activates the property
```
$ akamai property-manager activate-version -p <property-name> --network staging --wait-for-activate
```

## Test Stage
This is optional but highly recommended to always test the changes through any testing framentworks. In this example Newman (CLI for Postman) is used to run a public collection from a URL.

## Resources
- [Akamai OPEN Edgegrid API Clients](https://developer.akamai.com/libraries)
- [Akamai Pipeline](https://developer.akamai.com/devops/use-cases/akamai-pipeline)
- [Akamai Property-Manager and Pipeline CLI](https://github.com/akamai/cli-property-manager)
- [Akamai as Code with Jsonnet](https://developer.akamai.com/blog/2021/04/28/akamai-code-jsonnet)
- [Akamai Docker](https://github.com/akamai/akamai-docker)
- [Test Changes With Akamai Sandbox & Docker](https://developer.akamai.com/blog/2020/12/11/test-changes-akamai-sandbox-docker)
