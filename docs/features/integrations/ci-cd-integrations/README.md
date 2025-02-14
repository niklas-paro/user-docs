# Snyk CI/CD Integration

{% hint style="info" %}
All these methods provide the same results, as they all rely on the same Snyk engine. Thus the same arguments or options apply regardless of the deployment method you select.
{% endhint %}

### CI/CD deployment methods

There are various ways to configure Snyk within your pipeline. Choose a method depending on your environment and preference. You can expect all methods to lead to a successful run.

#### **Use Snyk native plugins**

Snyk native plugins are available for most common CI/CD tools. Using these plugins is the easiest way to set up and get started. The plugins include the most common parameters and options in the user interface (UI).

#### **Deploy Snyk CLI using the npm method**

Follow similar steps as those for installing the CLI locally. You must be able to run an npm command in the pipeline script. This method has the advantage of completely aligning with the CLI experience so you can easily troubleshoot and configure.

#### **Deploy Snyk CLI binary version**

The advantage of the binary setup is that it has no dependency with the local environment. For example, it is useful if you cannot run an npm command in your pipeline.

CLI binaries are available here: [https://github.com/snyk/snyk/tags](https://github.com/snyk/snyk/tags)

Snyk has Linux, Windows and other versions.

#### **Deploy a Snyk container**

You may also deploy Snyk in your pipeline using one of the Snyk images in Dockerhub: [https://hub.docker.com/r/snyk/snyk](https://hub.docker.com/r/snyk/snyk)

#### Examples

This repo shows some examples of binary and npm integrations for various CI/CD tools: [CI/CD examples](https://github.com/snyk-labs/snyk-cicd-integration-examples)

### Typical stages of adoption

Developer teams typically adopt Snyk in the following stages:

1. [Expose vulnerabilities](./#stage-1-expose-vulnerabilities-snyk-monitor) (`snyk monitor`)
2. [Use Snyk as a gatekeeper](./#stage-2-use-snyk-as-a-gatekeeper-snyk-test) (`snyk test`)
3. [Continuous monitoring](./#stage-3-continuous-monitoring-snyk-test-and-snyk-monitor) (`snyk test` and `snyk monitor`)

#### **Stage 1: Expose vulnerabilities (snyk monitor)**

This is a typical initial approach, using Snyk results to expose vulnerabilities during the development process, which increases visibility of these vulnerabilities among members of your team.

When you first implement Snyk in your pipeline, using only the `snyk monitor` command is recommended. If you use one of the Snyk CI plugins, it is recommended that you configure the plugin to _not_ fail the build.

This is because all projects are vulnerable, and after you set Snyk to fail the build, every build fails because of Snyk. This may cause problems with your team being quickly overwhelmed with failure messages.

Using `snyk monitor` to expose results provides information without disrupting processes.

#### **Stage 2: Use Snyk as a gatekeeper (snyk test)**

Using Snyk as a gatekeeper prevents the introduction of new vulnerabilities (sometimes known as "stopping the bleeding").

After your teams understand the vulnerabilities in their applications, and develop a process for fixing them early in the development cycle, you can configure Snyk to fail your builds, to prevent introducing vulnerabilities into your applications..

Add `snyk test` to your build or enable the fail functionality to make Snyk fail your builds, providing the results output to the console. Your Devs or DevOps teams can use the results to decide whether to stop or continue the build.

#### **Stage 3: Continuous monitoring (snyk test** and **Snyk monitor)**

After you configure Snyk to fail the build when vulnerabilities are detected, you can configure Snyk to send a snapshot of your project's successful builds to Snyk for ongoing monitoring.

To do this, configure your pipeline to run `snyk monitor` if your `snyk test` returns a successful exit code.

### **Technical implementation**

#### Prerequisites

To configure Snyk to run in a pipeline, retrieve key configuration inputs from your Snyk account.

#### Define target organization

When you run Snyk in your CI/CD platform, you typically want to post the test results to Snyk for review and ongoing monitoring.

You can define the target organization in the Snyk CLI, by either URL slug or organization ID, using the `--org` CLI option:

* You can define the target organization using its URL slug, as displayed in the address bar of the browser in the Snyk UI.
* Alternatively you can define the target organization using its `org id` in the organization's settings page.

![](../../../.gitbook/assets/image1.png)

#### **Default organization**

If you do not define a target organization, Snyk uses the default organization for the authentication token you use:

* For user accounts, this is the user's preferred organization (configurable in the user's settings).
* For organization service accounts, this is the organization in which the account was created.

### Snyk authentication token

To run `snyk test`, you need an authentication token with access to the target organization . While you can use any valid authentication token, using a service account is recommended. For more details, see [Service accounts](https://docs.snyk.io/integrations/managing-integrations/service-accounts).

### Setting up

Snyk supports the following approaches to add tests to a build pipeline:

* **Snyk integration plugins**: Snyk provides pre-built plugins for several CI servers, including [Jenkins](https://docs.snyk.io/integrations/ci-cd-integrations/jenkins-integration-overview), [Team City](https://docs.snyk.io/integrations/ci-cd-integrations/teamcity-integration-overview)[, Bitbucket Pipelines](https://docs.snyk.io/integrations/ci-cd-integrations/bitbucket-pipelines-integration-overview) and [Azure Pipelines. ](https://docs.snyk.io/integrations/ci-cd-integrations/azure-pipelines-integration)See the [Continuous Integration](https://docs.snyk.io/integrations/ci-cd-integrations) documentation for more details
* **Snyk CLI:** Teams with more complex workflows, or using a build system without a Snyk pre-built plugin, can use the Snyk CLI tool during CI/CD setups. See [Setting up using Snyk CLI](./#setting-up-using-snyk-cli) for details.
* **Snyk API**: For teams with complex requirements Snyk provides a REST API, which you can use for functions including initiating scans, onboarding new projects, and testing arbitrary libraries. See the [Snyk API documentation](../../snyk-api-info/) for more details.

### Setting up using Snyk CLI

Snyk CLI is a NodeJS application that can be scripted directly by developers for easy integration into most CI/CD environments, and is available as an npm application, pre-packaged binary, or container image. See [Install the Snyk CLI](https://docs.snyk.io/snyk-cli/install-the-snyk-cli) for more details

Snyk CLI can be configured to:

* Return non-zero error codes only when certain criteria are met, for example, exit with an error code only if vulnerabilities of high severity are present.
* Output all of its data into JSON for more flexibility.

#### **Exit Codes**

The `snyk test` command is synchronous; it ends with an exit code. Your build system can use exit codes to either pass or fail the build based on the test results. See the [CLI reference](../../../snyk-cli/cli-reference/) for the meaning of the exit codes.

The `snyk monitor` command posts The snyk monitor command posts the dependency tree to of a project to your Snyk account and monitors it for vulnerabilities there.. It is an asynchronous command, that does not end with an exit code based on the vulnerability status.

To silence Snyk CLI exit codes for the `snyk test` command to avoid failing the build step, use `|| true` at the end of the command. `|| true` sets the exit code of the scan to 0. This can be used to continue with a CI/CD pipeline even when there are vulnerabilities.

Depending on your approach and goals, you may need to use both commands in your pipeline.

#### **CLI examples in a build pipeline**

Use `snyk monitor` to surface vulnerabilities and post to the Snyk UI for ongoing monitoring:

```
snyk monitor --all-projects --org=snyk-apps
```

Use `snyk test` to fail the build on high severity issues:

```
snyk test --all-projects --org=snyk-apps --severity-threshold=high
```

To see the full list of options in the CLI, run the snyk test --help, snyk monitor --help, and snyk container --help commands or see the [help docs](../../../snyk-cli/commands/).

### Configuring options for failing builds

You can add options to the `snyk test` command to refine parameters that can result in a failed build:

* `--severity-threshold=high`: Fail the build only for high severity issues,
* `--fail-on=upgradable`: fail the build only for issues that are upgradable (can be fixed with Snyk fix advice)

You can also fail the build for any other parameter in the Snyk JSON output (such as CVSS score), using a wrapper like [snyk-filter](https://github.com/snyk-tech-services/snyk-filter), or using additional tooling like [snyk-delta](https://github.com/snyk-tech-services/snyk-delta) to fail the build only for issues found since the last build. For information on using snyk-delta see [Advanced failing of builds in the Snyk CLI](../../../snyk-cli/test-for-vulnerabilities/advanced-failing-of-builds-in-snyk-cli.md).

### Creating custom build artifacts

You can use Snyk's JSON output to create custom test reports as build artifacts, using the [snyk-to-html](https://github.com/snyk/snyk-to-html) utility, or other custom processing you develop.

### Creating work items for new vulnerabilities

Snyk allows you to automatically create new work items in JIRA (see [Jira integration](https://docs.snyk.io/integrations/untitled-3/jira) documentation). You can customize this code for your specific requirements, or adapt it to work with other work management systems.

See [Jira tickets for new vulns](https://github.com/snyk-tech-services/jira-tickets-for-new-vulns) to get started, or review the [API to create Jira tickets](https://snyk.docs.apiary.io/#reference/projects/project-jira-issues).

### Ignoring issues

By default if issues are not ignored, or if you are not using [snyk-delta](https://github.com/snyk-tech-services/snyk-delta), a `snyk test` in your pipeline fails the build when issues are found. To allow builds to continue without resolving these issues, you can:

* [Ignore issues using a .snyk policy file](https://docs.snyk.io/snyk-cli/fix-vulnerabilities-from-the-cli/ignore-vulnerabilities-using-snyk-cli)
* [Ignore issues from the Snyk UI](https://support.snyk.io/hc/en-us/articles/360000923498-How-can-I-ignore-a-vulnerability-)
* [Ignore issues from the Snyk API](https://snyk.docs.apiary.io/#reference/projects/project-ignores-by-issue/add-ignore)
* Use the Snyk Python API for bulk ignores: see [https://github.com/snyk-labs/pysnyk](https://github.com/snyk-labs/pysnyk) and [https://github.com/snyk-labs/pysnyk/blob/master/examples/api-demo-9c-bulk-ignore-vulns-by-issueIdList.py](https://github.com/snyk-labs/pysnyk/blob/master/examples/api-demo-9c-bulk-ignore-vulns-by-issueIdList.py)

### Snyk Open Source-specific strategies

These strategies are useful to teams using the Snyk SCA ([Software Composition Analysis](https://snyk.io/blog/what-is-software-composition-analysis-sca-and-does-my-company-need-it/)) testing features.

#### Gradle and Scala

* For "multi-project" configurations, test all sub-projects. Use this option with the `monitor` or `test` command: `--all-sub-projects`.
* To scan specific configurations, select certain values of configuration attributes to resolve the dependencies. Use this option with the `test` or `monitor` command: `--configuration-attributes=`.

#### Python

*   Snyk uses Python to scan and find your dependencies. Snyk needs the Python version to start scanning, and defaults to "python." If you are using multiple Python versions, use this option with the `test` or `monitor` command to specify the correct Python command for execution: `--command=`. Example:

    ```
    snyk test --command=python3
    ```
* If you scan a Pip project and use the `--file=` option because your manifest file is not the standard `requirement.txt`, then the next option is mandatory to specify Pip as the package manager `--package-manager=pip.`

#### .Net

If you use a `.sln` file, you can specify the path to the file, and Snyk scans all the sub projects that are part of the repo, for example:

```
snyk test --file=sln/.sln
```

#### Yarn

For Yarn workspaces use the `--yarn-workspaces` option to test and monitor your packages. The root lockfile will be referenced for scans of all the packages. Use the `--detection-depth` parameter to find sub-folders which are not auto discovered by default.

{% hint style="info" %}
**Note**\
Yarn workspaces support is for the `snyk test` and `snyk monitor` commands only at this time.
{% endhint %}

Example: scan only the packages that belong to any discovered workspaces in the current directory and 5 sub-directories deep.

```
snyk test --yarn-workspaces --detection-depth=6
```

You can use a common `.snyk` policy file if you maintain ignores and patches in one place to be applied for all detected workspaces, by providing the policy path as follows:

```
snyk test --yarn-workspaces --policy-path=src/.snyk
```

#### Monorepo

Some customers have complex projects, with multiple languages, package managers, and projects in a single repository. To facilitate this, you can take different approaches:

*   As you build each project and language, add a directive to run `snyk test` and target a specific project file, for example:

    ```
    snyk test --file=package.json
    ```

    After you install the dependencies of each project, make a similar call pointing to the specific artifact (such as `pom.xml`). This is fast and efficient, but can be difficult to scale, especially if you are not familiar with the project.
* When you use the `--all-projects` and `--detection-depth` options, the Snyk CLI or CI/CD plugin searches up to the `--detection-depth` in the folder structure for any manifests that match the supported file types. Each project is scanned and has its own result. Similarly, if you are using `snyk monitor,` a separate result is created for each project. This proides a good way to automate scanning, especially if you have projects spanning node, .net, python, and so on.
* For most **Gradle projects**, using `--all-projects` works, as it invokes gradle-specific options behind the scenes in the form of: `snyk test --file=build.gradle --all-sub-projects` when it finds the build file as part of the `--all-projects` search.
* Gradle may require additional configuration parameters. If so, target the other artifacts using `--file=` for each manifest of the other languages and package-managers. You must then use `--all-sub-projects` and potentially `--configuration-matching` to scan a complex Gradle project.

See [Snyk for Java (Gradle, Maven)](../../../products/snyk-open-source/language-and-package-manager-support/snyk-for-java-gradle-maven.md) for more information.

### Snyk Container-specific strategies

The best stage to implement Snyk Container in your pipeline is after the container image is built (after running the equivalent of “docker build”), and before your image is either pushed into your registry (“docker push”) or deployed to your running infrastructure (“helm install”, “kubectl apply” and so on).

Typically, the way you run your container build-test-deploy pipeline depends on whether or not a Docker daemon is available to the build agent.

#### **Running pipeline if a Docker daemon is available**

If the following circumstances exist:

* You are running your build tooling (such as Jenkins) directly on a host that has Docker natively installed.
* Your pipeline tasks are run inside containers which have the Docker socket \[/var/run/docker.sock] bind-mounted to the host.
* You are running a Docker-inside-Docker setup.

Snyk can help as follows:

* When you run `snyk container test $IMAGE_NAME`, Snyk looks for that image in your local daemon’s storage, and if it does not exist, does the equivalent of a **docker pull** to download it from your upstream registry.
* For registry authentication, Snyk uses the credentials you already configured (with something like **docker login**).
* You can specify `--file=Dockerfile` on the command line to link the image vulnerability results with the Dockerfile that built it, to receive inline fix advice and alternate base image suggestions.

#### **Running pipeline if a Docker daemon is not available**

If the following circumstances exist:

* You containerize each build task but do not mount the Docker socket for security and performance reasons.
* Pipeline tasks are split across hosts (or even clusters) and rely on artifacts to be handed off though a central volume or intermediate registry/object store.
* You work exclusively in an ecosystem that only uses OCI-compliant container images.

Snyk can help as follows:

* Run either `snyk container test docker-archive:archive.tar` or `snyk container test oci-archive:archive.tar` to get Snyk vulnerability results against tar-formatted container images (either in Docker or OCI format) without relying on the Docker daemon.
* The tar archive can be generated by your build process using the equivalent of **docker save** and stored in a shared volume or object store. This can be accessed by the build agent container running the Snyk binary, with no other dependencies required.

### Good practice recommendations

* Regardless of how you integrate with container images during CI, run a Snyk Container scan as a separate build step from your Snyk Open Source (application SCA) test. This allows you to isolate build failures to vulnerabilities within either the container/OS layer or the application layer, respectively. This also enables more easily containerized build tasks.
* Use CLI flags like `--fail-on` and `--sevety-threshold` to customize the failure status for the build task. For more advanced usage, you can use `--json` to generate a JSON file containing the full vulnerability report, and set your own build failure status based on the JSON data.
* Pass `--exclude-base-image-vulns` to report only vulnerabilities introduced by your user layers, rather than vulnerabilities that come from the container’s base image (the image you specify in the FROM line in the Dockerfile).
* Run `snyk container monitor` following `snyk container test` (or simply check the **Monitor** box on your plugin settings), to keep a record of this container’s bill of materials within the Snyk UI and proactively monitor for new vulnerabilities on a daily basis. This is useful when pushing new releases into production environments. You can use `--project-name` to specify a unique identifier for the release to ensure production containers are tracked separately from others in your build process.

### Snyk IaC-specific strategies

The best stage to implement Snyk Infrastructure As Code in your pipeline as part of the stages, but after the SCA and the Containers stage.

Snyk Infrastructure as Code supports:

* Deployments, Pods, and Services.
* CronJobs, Jobs, StatefulSet, ReplicaSet, DaemonSet, and ReplicationController.

See [Test your Kubernetes files with our CLI tool](https://docs.snyk.io/snyk-infrastructure-as-code/snyk-cli-for-infrastructure-as-code/test-your-kubernetes-files-with-our-cli-tool) for more details.

### CI/CD troubleshooting and advanced tips

This section provides a few tips to help troubleshoot or scale CI/CD integrations.

#### Step 1: Try to replicate with Snyk CLI

If CLI and pipeline are running the same engine, try to clone the project and scan with CLI.

Play with the CLI flags. Use the Snyk CLI tool to find and fix known vulnerabilities as you run it in the pipeline. For more information see the [CLI reference](../../../snyk-cli/cli-reference/).

#### Step 2: Get logs

If you are able to replicate with CLI and the problem still exists, ask the CLI to output the debug logging using the following command: `DEBUG=*` or use the `-d` option to capture logs:

```
snyk test -d
```

or

```
DEBUG=* snyk test
```

#### Step 3: Use the CLI instead of the plugin

Try to replace the native plugin with the CLI by installing the CLI. See [Install the Snyk CLI ](../../../snyk-cli/install-the-snyk-cli/)for instructions.

### Common flags in a CI/CD integration

Among the most common flags used in a CI/CD integration are the following:

`-- all-projects`: Auto-detect all projects in working directory

`-p`: Prune dependency trees, removing duplicate sub-dependencies. Continues to find all vulnerabilities, but may not find all of the vulnerable paths.

\--org=\<ORG\_ID>: Specify the ORG\_ID to run Snyk commands tied to a specific organization. This influences where new projects are created after running the `monitor` command, some features availability, and private tests limits. If you have multiple organizations, you can set a default from the CLI using:

```
$ snyk config set org=<ORG_ID>
```

Set a default to ensure all newly tested or monitored projects are tested under your default organization. If you need to override the default, use the `--org=<ORG_ID>` option.

Default: `<ORG_ID>` that is the current preferred organization in your [Account settings](https://app.snyk.io/account).

### **Useful resources**

The following repo shares some examples of binary and npm integrations for various CI/CD tools: [GitHub CI/CD examples](https://github.com/snyk-labs/snyk-cicd-integration-examples)

## Configure your continuous integration

To continuously avoid known vulnerabilities in your dependencies, integrate Snyk into your continuous integration (a.k.a. build) system. In addition to the documentation here, you're also invited to check out our integration configuration examples in our GitHub repository.

### Set up automatic monitoring

If you monitor a project with Snyk, you’ll get notified if your project’s dependencies are affected by newly disclosed vulnerabilities. To make sure the list of dependencies we have for your project is up to date, refresh it continuously by running Snyk monitor in your deployment process. Configure your environment to include the SNYK\_TOKEN environment variable. You can find your API token on the dashboard after logging in.

### API token configuration

Make sure you don’t check your API token into source control, to avoid exposing it to others. Instead, use your CI environment variables to configure it.

See guidance for how to do this on:

* [Travis](https://docs.travis-ci.com/user/environment-variables/)
* [Circle](https://circleci.com/docs/environment-variables/)
* [Codeship](https://codeship.com/documentation/continuous-integration/set-environment-variables/)

You can find others through an easy [Google search](https://www.google.co.uk/search?q=setting+up+env+variables+in+CI).\
Learn more about [CI/CD](https://snyk.io/learn/what-is-ci-cd-pipeline-and-tools-explained/).
