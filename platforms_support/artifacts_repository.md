---
layout: default
title: Artifacts Repository
parent: Platforms support
nav_order: 4
has_children: false
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

You can read below a study to choose the best artifacts repository platform which will fit your needs. For this study, I compare several artifacts repository platform against a known platform reference **[Artifactory](https://www.jfrog.com/confluence/display/JFROG/Get+Started)**.

Jfrog Artifactory is a full platform with lot of features to store, distribute, manage artifacts and scan vulnerabilities. Jfrog Artifactory is a leader in this domain but comes with a lot of features that maybe you will never use and has also important cost if you want to use the complete tools.

## Requirements Definition
{: .text-blue-300 }

Before choosing the platform, it is mandatory to have a complete analysis of uses cases to be covered:

* Why do i need repositories ?
* Do we have Internet access from workstations, from servers ?
* How do we want manage external packages, in-house packages ?
* Can we trust downloaded packages ?
* What is the estimated size of storage ?
* How do we manage obsolescence ?

{: .important-title }
> Contoso requirements
>
> **Development activities:**
>
> * Dotnet : Need nuget repositories
> * Java : Need maven or graddle repositories.
>
>   Java developers choose IntelliJ as IDE and selected maven as project management
>
> * Javascript : Need npm repositories
> * Direct Business / scientific development : Need Python and CRAN repositories
> * Architects follow RUST and GO languages : Cargo and Go repositories
>
> **Infrastructure environment:**
>
> * Linux Ubuntu : Need debian and Python repositories
> * Windows : Need nuget repositories for powershell modules and scripts
> * Automation based on Ansible and powershell DSC : Need python and nuget repositories
>
> **Container orchestration:**
>
> * Containers : Need Docker and alpine repositories
> * Kubernetes : Need Helm repositories
>
> **Security:**
>
> * Internet access is blocked for all servers
> * Internet download is blocked for all developers
> * Downloaded packages must be approved and scanned for security vulnerabilities
> * Existing used packages must be scanned for security vulnerabilities and blocked when vulnerables
>
> **Governance:**
>
> * Packages or application must be validated before production deployment

### Package types

| Package Type             | Description                                                                                                                                                         |
| ------------------------ |:------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Alpine                   | Use Artifactory to gain full control of your deployment and resolution process of Alpine Linux (*.apk) packages.                                                    |
| Helm                     | Manage your Helm Charts in Artifactory and gain control over deployments to your Kubernetes cluster.                                                                |
| Maven                    | Artifactory is both a source for Maven artifacts needed for a build, and a target to deploy artifacts generated in the build process.                               |
| npm                      | Host your own node.js packages, and proxy remote npm repositories like npmjs.org through Artifactory.                                                               |
| NuGet                    | Host and proxy NuGet packages in Artifactory, and pull libraries from Artifactory into your various Visual Studio .NET applications.                                |
| PyPI                     | Host and proxy PyPI distributions with full support for pip.                                                                                                        |
| Generic                  | Generic binaries storage.                                                                                                                                           |
| CRAN                     | Deploy and resolve CRAN packages for the R language using dedicated CRAN repositories.                                                                              |
| Debian                   | Host and provision Debian packages complete with GPG signatures.                                                                                                    |
| Docker                   | Host your own secure private Docker registries and proxy external Docker registries such as Docker Hub.                                                             |
| Visual studio extensions |                                                                                                                                                                     |
| Cargo                    | Enhance your capabilities for configuration management with Cargo using all the benefits of a repository manager.                                                   |
| Go Registry              | Build Go projects while resolving dependencies through Artifactory, and then publish the resulting Go packages into a secure, private Go registry                   |
| Conan                    | Artifactory is the only secure, private repository for C/C++ packages with fine-grained access control.                                                             |
| Conda                    | Artifactory natively supports Conda repositories for Python, R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, FORTRAN.                                                |
| Git LFS                  | Optimize your workflow when working with large media files and other binary resources.                                                                              |
| Gradle                   | Resolve dependencies from and deploy build output to Gradle repositories when running Gradle builds.                                                                |
| Opkg                     | Optimize your work with OpenWrt using Opkg repositories. Proxy the official OpenWrt repository and cache remote .ipk files.                                         |
| RubyGems                 | Use Artifactory to host your own gems and proxy remote gem repositories like rubygems.org.                                                                          |
| Bower                    | Boost your front end development by hosting your own Bower components and proxying the Bower registry in Artifactory.                                               |
| Chef                     | Enhance your capabilities for configuration management with Chef using all the benefits of a repository manager.                                                    |
| CocoaPods                | Speed up development with Xcode and CocoaPods with fully fledged CocoaPods repositories.                                                                            |
| P2                       | Proxy and host all your Eclipse plugins via an Artifactory P2 repository, allowing users to have a single-access-point for all Eclipse updates.                     |
| PHP Composer             | Provision Composer packages from Artifactory to the Composer command line tool, and access Packagist and other remote Composer metadata repositories.               |
| Pub Repositories         | Artifactory natively supports Dart packages, giving you full control of your deployment and resolution process of Flutter, Angular Dart, and general Dart programs. |
| Puppet                   | Configuration management meets repository management with Puppet repositories in Artifactory.                                                                       |
| RPM                      | Distribute RPMs directly from your Artifactory server, acting as fully-featured YUM repository.                                                                     |
| SBT                      | Resolve dependencies from and deploy build output to SBT repositories when running SBT builds.                                                                      |
| Vagrant                  | Securely host your Vagrant boxes in local repositories.                                                                                                             |
| VCS                      | Consume source files packaged as binaries.                                                                                                                          |


## Packages types support
{: .text-blue-300 }

1. Determine the package's type that you need to support in your enterprise for development activities and for infrastructure automation.
2. Define a weight of importance

    1. Package type is mandatory because these technologies are currently used, belong of supported technologies and are not covered by an other tool

    2. Package type could become a futur official technologie due to its popularity

    3. Package type could be a plus for your enterprise to handle specific use case which are not covered

    4. You will not plan to use this technologie


| Package Type             | Priority | Artifactory | Azure DevOps Service | Nexus | ProGet |
| ------------------------ |:--------:|:-----------:|:--------------------:|:-----:|:------:|
| Alpine                   | 1        | ✅          | ❌                  | ?     | ❌     |
| Helm                     | 1        | ✅          | ❌                  | ?     | ✅     |
| Maven                    | 1        | ✅          | ✅                  | ?     | ✅     |
| npm                      | 1        | ✅          | ✅                  | ?     | ✅     |
| NuGet                    | 1        | ✅          | ✅                  | ?     | ✅     |
| PyPI                     | 1        | ✅          | ✅                  | ?     | ✅     |
| Generic                  | 1        | ✅          | ☑️ (1)              | ?     | ✅     |
| CRAN                     | 1        | ✅          | ❌                  | ?     | ❌     |
| Debian                   | 1        | ✅          | ❌                  | ?     | ✅     |
| Docker                   | 1        | ✅          | ❌                  | ?     | ✅     |
| Visual studio extensions | 2        | ❌          | ❌                  | ?     | ✅     |
| Cargo                    | 2        | ✅          | ❌                  | ?     | ❌     |
| Go Registry              | 2        | ✅          | ❌                  | ?     | ❌     |
| Conan                    | 3        | ✅          | ❌                  | ?     | ❌     |
| Conda                    | 3        | ✅          | ❌                  | ?     | ❌     |
| Git LFS                  | 3        | ✅          | ❌                  | ?     | ❌     |
| Gradle                   | 3        | ✅          | ❌                  | ?     | ❌     |
| Opkg                     | 3        | ✅          | ❌                  | ?     | ❌     |
| RubyGems                 | 3        | ✅          | ❌                  | ?     | ✅     |
| Bower                    | 4        | ✅          | ❌                  | ?     | ❌     |
| Chef                     | 4        | ✅          | ❌                  | ?     | ❌     |
| CocoaPods                | 4        | ✅          | ❌                  | ?     | ❌     |
| P2                       | 4        | ✅          | ❌                  | ?     | ❌     |
| PHP Composer             | 4        | ✅          | ❌                  | ?     | ❌     |
| Pub Repositories         | 4        | ✅          | ❌                  | ?     | ❌     |
| Puppet                   | 4        | ✅          | ❌                  | ?     | ❌     |
| RPM                      | 4        | ✅          | ❌                  | ?     | ❌     |
| SBT                      | 4        | ✅          | ❌                  | ?     | ❌     |
| Vagrant                  | 4        | ✅          | ❌                  | ?     | ❌     |
| VCS                      | 4        | ✅          | ❌                  | ?     | ❌     |

**(1)** Universal Packages are not supported on Azure DevOps Servers on premise.

{: .warning-title }
> Artifactory feedbacks for NPM
>
> In the real life, we met somes packages corruptions issues with NPM and Artifactory V6.x and V7.x. Normally they have been fixed but we continue to have this kind of issues.

:point_right: **Package type support**

* Even if we don't need to have all packages types, the platform selection must be oriented on a specialized product for artifacts management.
* Azure DevOps service is more oriented to development activities and can't fit Infrastructure needs**


### Sources

* [Artifactory Package Management](https://www.jfrog.com/confluence/display/JFROG/Package+Management){:target="_blank"}
* [Azure DevOps Package Management](https://docs.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops){:target="_blank"}
* [ProGet](https://docs.inedo.com/docs/proget-packages-what-is-a-package#what-package-types-does-proget-support-){:target="_blank"}

### Artifactory

### Azure DevOps Server


## Artifacts Repository
