# GHPackages-Demo
Instructions to automate CI/CD w/ GitHub Workflows and Packages :) by [jstnf](https://github.com/jstnf)

This is a simple tutorial on using GitHub's Workflows to deploy and host Maven artifacts on GitHub packages.

## Contents
* [Setup](#setup)
* [1. Create a GitHub PAT (personal access token)](#1-create-a-github-pat-personal-access-token)
* [2. Add the workflow files to your repository](#2-add-the-workflow-files-to-your-repository)
* [3a. Modify your parent `pom.xml` (multi-module projects only!)](#3a-modify-your-parent-pomxml-multi-module-projects-only)
* [3b. Modify your project/module `pom.xml` (both single and multi-module projects)](3b-modify-your-projectmodule-pomxml-both-single-and-multi-module-projects)
* [4. Add the `PERSONAL_GITHUB_TOKEN` secret to the repository](#4-add-the-personal_github_token-secret-to-the-repository)
* [5. (optional) Create a new release and upload your first package](#5-optional-create-a-new-release-and-upload-your-first-package)
* [6. Admire your work!](#6-admire-your-work)
* [Troubleshooting](#troubleshooting)
  * [Connection time out when downloading dependencies in the workflow](#connection-time-out-when-downloading-dependencies-in-the-workflow)
  * [`Error 409: Conflict`](#error-409-conflict)
  * [`Error 422: Unprocessable Entity`](#error-422-unprocessable-entity)

## Setup
To begin, you'll need:
* A Maven project (either multi-module w/ a parent pom or single module)
* A GitHub repository associated with the project
* A GitHub PAT (personal access token) which we will create later

## 1. Create a GitHub PAT (personal access token)
Navigate to your GitHub profile settings. Select **Developer Settings** at the bottom of the left menu, and then **Personal access tokens**.

Select **Generate new token** to create a new PAT. Check the `write:packages` permission in the list below. This will automatically grant the `repo` permission to your personal access token.

Copy this token and save it somewhere. You will recieve this token from GitHub only once; if forgotten you will need to deactivate it, so keep it safe!

## 2. Add the workflow files to your repository
Copy the contents of the .github folder to your repository. Specifically, you want to make sure that the `maven-publish.yml` workflow YAML file is in your repository in the `workflows` folder.

## 3a. Modify your parent `pom.xml` (multi-module projects only!)
*(skip this step if your project only contains a singular `pom.xml` file)*

In a normal multi-module project on Ataraxia, your Maven structure will be composed of a parent `pom.xml` file at the root of your repository, and a `pom.xml` file for each module in your project. For this step, we will enter the parent `pom.xml` file.

Merge this into your parent `pom.xml`.
```lang=xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.7</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

*What's happening here?* Well, we don't want to publish the parent module to GitHub packages. This will cause the build to fail in the end.

## 3b. Modify your project/module `pom.xml` (both single and multi-module projects)
Now, open the `pom.xml` file of artifacts you wish to upload to GitHub packages.

Firstly, you need to ensure that the `artifactId` of your `pom.xml` is completely lowercase! This is an important step. If you do not make this correct, you may run into `Error 422: Unprocessable Entity` when deploying to GitHub packages.

Next, merge this into your `pom.xml` file **of any artifacts you wish to upload to GitHub Packages**.
```lang=xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.7</version>
                <configuration>
                    <skip>false</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
    <distributionManagement>
        <repository>
            <id>github</id>
            <name>GitHub AtaraxiaMC Apache Maven Packages</name>
            <url>https://maven.pkg.github.com/AtaraxiaMC/{REPO}</url>
        </repository>
    </distributionManagement>
```

Here, we ensure the deploy plugin does not skip this module. In the `distributionManagement` section, be sure to change the `{REPO}` with the name of your repository.

Now, merge this into your `pom.xml` file **of any artifacts you wish to not upload to GitHub Packages**.
```lang=xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.7</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

We don't need to create a `distributionManagement` here, and whether the `artifactId` of these modules are upper or lowercase does not matter.

Commit all of these changes back to the repository.

## 4. Add the `PERSONAL_GITHUB_TOKEN` secret to the repository
Remember that **personal access token** that we created before? We need to take this token and create a new repository secret that contains it.

Go to your repository's settings. On the left, click **Secrets**, and then **Actions**. Click **New repository secret**.

Now, name the secret `PERSONAL_GITHUB_TOKEN`. The workflow uses the secret with this name to authenticate into GitHub Packages. For the secret's contents, paste in the personal access token you created earlier.

## 5. (optional) Create a new release and upload your first package
This workflow only runs when a new release is published in your GitHub repository. Therefore, we need to create and publish a new release.

Go to your repository's homepage. Click **Releases** on the right. Depending on whether your repository already contains releases, you'll need to click either **Create a new release** or **Draft a new release**.

Create a new tag for your release. Feel free to customize the name and description of your release to your liking, uploading binaries or JARs if you wish. The contents of your release do not impact the workflow's results.

Finally, publish your release. The workflow will begin automatically once you do this.

## 6. Admire your work!
You're done! Your repository is now able to upload and host artifacts with GitHub Packages. Use standard authentication steps to get these packages in your Maven projects.

## Troubleshooting
### Connection time out when downloading dependencies in the workflow
This is a random bug that occurs with GitHub. Delete any packages that your workflow has created and then re-trigger the workflow in Actions.

### `Error 409: Conflict`
Error 409 indicates a package with the same version exists. You may need to delete this package in order to perform the workflow.

### `Error 422: Unprocessable Entity`
Ensure your artifact is completely lowercase. If it is, try adding additional characters to the name of your artifact (in turn, changing the artifact name and thus the dependency in any dependent `pom.xml` files!).

Another reason this may occur is that your package is uploaded in another repository somewhere (identical `groupId` and `artifactId`. GitHub will prevent you from uploading the package if this is the case. Attempt to find and delete these offending packages.
