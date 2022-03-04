# Control Plane - Environment Promotion Example

This example project demonstrates CI/CD for releasing an app to multiple environments. Upon the creation of a pull request, a 'Review Workload' is deployed to a specified "Dev Org". It is prefixed by the branch name. In this manner, multiple branches can be worked on simultaneously. Whenever a pull request is merged, the app is deployed to the same environment with a dev- prefix.

An image is only built once. A short hash is added to the image name to ensure uniqueness. That same image is then deployed to all environments, depending on the action being executed.

While this example utilizes two orgs, it illustrates there's no limitation to the number of orgs you can use.

## GitHub Actions

This project contains three GitHub Actions (in the `./.github/workflows` directory):

1. On a pull request (or updates to an existing pull request) to the `main` branch, the application is containerized and pushed
   to the dev Org's private repository. The GVC and `Review Workload` is created (if it not already created) by applying the YAML contents of the files `./cpln/cpln-gvc.yaml` and `./cpln/cpln-workload.yaml`. The name of the `Review Workload` is prefixed by the name of the branch that created the pull request and a dash.
2. When a pull request is accepted and the code is merged into the `main` branch, a `dev` workload is
   updated (or created if it doesn't exists) in the dev Org by applying the same files as step 1, except that the workload name is prefixed with `dev-`. This allows the application to be reviewed and tested before being pushed to the production Org.
3. The promotion to the `stage` and `prod` workload is accomplished by manually executing the `Deploy-To-Stage-or-Prod`
   workflow and selecting the desired workload type (stage or prod). The target GVC and Workload in the production Org is updated (or created if it doesn't exist) by applying the YAML contents of the files `./cpln/cpln-gvc-prod.yaml` and `./cpln/cpln-workload.yaml`. The main difference between the two GVC files is that the `prod` version contains the `Pull Secret` that is needed the pull the image from the dev Org. The `stage` and `prod` Workloads reuse the image that was initially pushed to the development Org.

## Release Phase

Each action contains a release phase that will execute a script named `release-tasks.sh` within the path
`release/ENVIRONMENT_NAME`. If this phase fails, the remainder of the action will not execute.

## Control Plane Prerequisites

The following Control Plane resources are required:

- Two Control Plane Orgs representing a `dev` and `prod` environments. The `dev` org will host the review and dev Workloads. The `prod` org will host the stage and prod Workloads.

Note: The `stage` and `prod` Workloads are deployed to the `prod` org since these environments should mirror each other.

## Control Plane Authentication Set Up

This example uses the Control Plane CLI to perform the necessary actions within the GitHub Actions. The CLI requires a `Service Account` with the proper permissions to perform actions.

Since actions will be performed against multiple Orgs, a `Service Account` is required in each Org.

**Perform the following steps in each Org:**

1. Follow the Control Plane documentation to create a `Service Account` and create a key. Take a note of the key. It will be used in the next section.
2. Add the Service Account to the `superusers` group. Once the GitHub Action executes as expected, a policy can be created with a limited set of permissions and the `Service Account` can be removed from the `superusers` group.

## Example Set Up

**Perform the following steps to set up the example:**

1. Fork this repo to your own GitHub account.

2. The following variables are required and must be added as GitHub repository secrets.

Browse to the Secrets page by clicking `Settings` (top menu bar), then `Secrets` (left menu bar), and finally click `Actions`.

Add the following variables:

- `CPLN_ORG_DEVELOPMENT`: Name of the `dev` Control Plane Org.
- `CPLN_ORG_PRODUCTION`: Name of the `prod` Control Plane Org.
- `CPLN_GVC`: GVC Name
- `CPLN_WORKLOAD`: Workload Name
- `CPLN_TOKEN_DEVELOPMENT`: Development Org's Service Account Key from the previous step.
- `CPLN_TOKEN_PRODUCTION`: Production Org's Service Account Key from the previous step.
- `CPLN_IMAGE`: The name of the image that will be deployed to the development Org's private repository. The GitHub Action workflow will append the short SHA of the commit as the image tag.

3. Review the `.github/workflow/*` files. These actions will be triggered as described in the `GitHub Actions` section above.

4. Review the Control Plane YAML files that are located in the `/cpln` directory. No changes are required to execute the example.

   - The `cpln-gvc.yaml` file defines the GVC to be created/updated for the `dev` Org.
   - The `cpln-gvc-prod.yaml` file defines the Pull Secret and GVC to be created/updated for the `prod` Org.
   - The `cpln-workload.yaml` file defines the Workload to be created/updated corresponding to the review/dev/staging/prod Workloads.

5. IMPORTANT: In order for the GitHub Actions to execute, they need to be enabled by clicking the `Actions` tab and clicking the `I understand my workflows, go ahead and enable them` button.

## Running the App

After the GitHub Action has successfully deployed the application, it can be tested by following these steps:

1. Browse to the Control Plane Console (https://console.cpln.io/).
2. If necessary, select a different Org by clicking your profile circle in the upper right corner, click the Org pull-down, and select the target Org.
3. Select the GVC that was set in the `CPLN_GVC` variable.
4. Select the desired Workload.
5. Click the `Open` button. The app will open in a new tab and the output will be displayed.

## Notes

- The `cpln apply` command creates and updates the resources defined within the YAML file. If the name of a resource is changed, `cpln apply` will create a new resource. Any orphaned resources will need to be deleted manually.

- The Control Plane CLI commands use the `CPLN_ORG` and `CPLN_TOKEN` environment variables when needed. There is no need to add the --org or --token flags when executing CLI commands.

- The GVC definition must exists in its own YAML file. The `cpln apply` command executing the file that contains the GVC definition must be executed before any child YAML files (workloads, identities, etc.) are executed.

## Permissions

To control which users have the ability to perform sensitive actions, such as merging a pull request, you can utilize the built-in capabilities of GitHub by creating a custom repository role.

Review these <a href="https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-repository-roles-for-an-organization" target="_blank">instructions</a> on how to create a role.

## Helper Links

GitHub

- <a href="https://docs.github.com/en/actions" target="_blank">GitHub Actions Docs</a>
