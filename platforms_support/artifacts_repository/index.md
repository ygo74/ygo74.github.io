---
layout: default
title: Artifacts Repository
parent: Platforms support
nav_order: 4
has_children: true
mermaid: true
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

The study will compare:

* JFrog Artifactory : Reference platform in **V7.x**
* Azure DevOps Services Artifacts in version **2022-04**
* Sonatype Nexus Repository in **V3.x**
* Indeo ProGet in **V6.x**

## Requirements Definition

Before choosing the platform, it is mandatory to have a complete analysis of uses cases to be covered:

* Why do I need repositories ?
* Do I have Internet access from workstations, from servers ?
* How do I want to manage external packages, in-house packages ?
* Can I trust downloaded packages ?
* What is the estimated size of storage ?
* How do I want to manage packages / applications lifecycle ?

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
> * Packages storage, naming conventions and versioning schemes must be enforced

## Packages types support

1. Determine the package's type that you need to support in your enterprise for development activities and for infrastructure automation.
2. Define a weight of importance

    1. Package type is mandatory because these technologies are currently used, belong of supported technologies and are not covered by an other tool

    2. Package type could become a futur official technologie due to its popularity

    3. Package type could be a plus for your enterprise to handle specific use case which are not covered

    4. You will not plan to use this technologie

### Comparison analysis

| Package Type             | Priority | Artifactory | Azure DevOps Service | Nexus  | ProGet  |
| ------------------------ |:--------:|:-----------:|:--------------------:|:------:|:-------:|
| Alpine                   | 1        | ✅          | ❌                  | ☑️ (2) | ❌     |
| Helm                     | 1        | ✅          | ❌                  | ✅     | ✅     |
| Maven                    | 1        | ✅          | ✅                  | ✅     | ✅     |
| npm                      | 1        | ✅          | ✅                  | ✅     | ✅     |
| NuGet                    | 1        | ✅          | ✅                  | ✅     | ✅     |
| PyPI                     | 1        | ✅          | ✅                  | ✅     | ✅     |
| Generic                  | 1        | ✅          | ☑️ (1)              | ✅     | ✅     |
| CRAN                     | 1        | ✅          | ❌                  | ✅     | ❌     |
| Debian                   | 1        | ✅          | ❌                  | ✅     | ✅     |
| Docker                   | 1        | ✅          | ❌                  | ✅     | ✅     |
| Visual studio extensions | 2        | ❌ (3)      | ❌ (3)              | ❔ (3)  | ✅     |
| Cargo                    | 2        | ✅          | ❌                  | ☑️ (2) | ❌     |
| Go Registry              | 2        | ✅          | ❌                  | ✅     | ❌     |
| CPAN                     | 3        | ❔           | ❌                  | ☑️ (2) | ❔     |
| Conan                    | 3        | ✅          | ❌                  | ✅     | ❌     |
| Conda                    | 3        | ✅          | ❌                  | ✅     | ❌     |
| Git LFS                  | 3        | ✅          | ❌                  | ✅     | ❌     |
| Gradle                   | 3        | ✅          | ✅                  | ❔      | ❌     |
| Opkg                     | 3        | ✅          | ❌                  | ❔      | ❌     |
| RubyGems                 | 3        | ✅          | ❌                  | ✅     | ✅     |
| Bower                    | 4        | ✅          | ❌                  | ✅     | ❌     |
| Chef                     | 4        | ✅          | ❌                  | ☑️ (2) | ❌     |
| CocoaPods                | 4        | ✅          | ❌                  | ✅     | ❌     |
| ELPA                     | 4        | ❔           | ❌                  | ☑️ (2) | ❔     |
| Flex                     | 4        | ❔           | ❌                  | ☑️ (2) | ❔     |
| P2                       | 4        | ✅          | ❌                  | ✅     | ❌    |
| PHP Composer             | 4        | ✅          | ❌                  | ☑️ (2) | ❌    |
| Pub Repositories         | 4        | ✅          | ❌                  | ❔      | ❌    |
| Puppet                   | 4        | ✅          | ❌                  | ☑️ (2) | ❌    |
| RPM                      | 4        | ✅          | ❌                  | ✅     | ❌    |
| SBT                      | 4        | ✅          | ❌                  | ❔     | ❌     |
| Vagrant                  | 4        | ✅          | ❌                  | ❔     | ❌     |
| VCS                      | 4        | ✅          | ❌                  | ❔     | ❌     |

* **(1)** Universal Packages are not supported on Azure DevOps Servers on premise.
* **(2)** Nexus repository support provided by the communauty
* **(3)** Work around to manage manually private gallery in a generic repository

    * [Jfrog support](https://www.jfrog.com/jira/browse/RTFACT-7471)
    * [Visual studio documentation](https://docs.microsoft.com/en-us/visualstudio/extensibility/private-galleries?view=vs-2022)

{: .warning-title }
> Artifactory feedbacks for NPM
>
> In the real life, we met somes packages corruptions issues with NPM and Artifactory V6.x / V7.x. Normally they have been fixed but we continue to have this kind of issues.

### Comparison conclusion

:point_right: Even if we don't need to have all packages types, the platform selection must be **oriented on a specialized product** for artifacts management.

:point_right: **Jfrog Artifactory and Sonatype Nexus are the only platform** which cover all Contoso's requirements.

:point_right: Sonatype Nexus also offers an important list of supported package types **thanks to the communauty**.

  {: .note-title }
  > Nexus communauty
  >
  > It could be interresting to check the communauty activites for these repositories and the complexity to provide a custom packag etype such as Visual Studio extensions.

:point_right: **Indeo Proget could be cover enterprises' requirement** It is near to fit all Contoso's requirements except for two particular uses cases:

* CRAN which becomes a popular language for financial developments
* Alpine package which is required to customize Docker Linux Alpine images. If Contoso chose Centos / Redhat, this requirement would not have been mandatory.

:point_right: **Azure DevOps service is more oriented to development activities** and can't fit Infrastructure needs

## Repository management features

### Features analysis

1. Virtual Repository

    Expose a virtual repository is a good choice to mask physical repository structures, to aggregate multiples repositories and allow platform administrators to re-arrange physical repositories without disturbing clients.

    ```mermaid
    flowchart BT
      subgraph Artifacts platform
        virtual-.->inhouse(In house packages)
        virtual-.->external(External packages)
      end
      client(client)-->virtual(Virtual Repository)
      style client fill:#bcee68,stroke:#333,stroke-width:4px
      classDef blue fill:#6666ff,stroke:#333,stroke-width:4px;
      class virtual,inhouse,external blue
    ```

2. Remote Repository

    This kind of repository allow to centralize internet access and to cache locally packages, therefore decrease latency and bandwith to download packages.
    They also act as a gateway which can be monitored for vulnerabilities and licences controls.

3. Repository Replication

    Capacity to replicate artifacts between repositories can be an important tools to manage platform, for sample to replicate artifacts between 2 distincts platforms which are seggregated for differents reasons (security, tests, geographic locations)

4. Repository cleaning

    Capacity to auto clean artifacts which are not downloaded since a long time can be helpfull to decrease the storage space

5. Artifacts storage

    Technology used to store artifacts can have an important impact on the storage size because we can find same artifacts in different repositories

6. Artifcats promotion

    Capacity to copy / move artifacts between repositories according the artifacts' lifecycle (development, tests, production ready) can help to track artifacts status, put in place a delivery workflow, ensure artifacts delivey to correct target depending their status

    ```mermaid
    flowchart LR
      subgraph Artifacts platform
        dev-.->tests(Tests repository)
        tests-.->prod(Production repository)
      end
      client(Build Servers)-->dev(Development Repository)
      style client fill:#bcee68,stroke:#333,stroke-width:4px
      classDef blue fill:#6666ff,stroke:#333,stroke-width:4px;
      class dev,tests,prod blue
    ```

7. Repository security

### Features comparison

| Repository features                | Sub features                             | Artifactory | Azure DevOps Service | Nexus | ProGet |
| ---------------------------------- | ---------------------------------------- |:-----------:|:--------------------:|:-----:|:------:|
| Virtual Repository                 | Virtual repository support               | ✅          | ✅                  | ❔     | ✅   |
|                                    | Resolution ordering                      | ✅          | ✅                  | ❔     | ❔    |
|                                    | Views filtering                          | ❌          | ✅                  | ❔     | ❔    |
|                                    | Include / exclude pattern                | ✅          | ❌                  | ❔     | ❔    |
|                                    | Underlying repository deployment         | ✅          | ❌                  | ❔     | ❔    |
| Remote Repository                  | Remote Repository support                | ✅          | ✅                  | ❔     | ✅   |
|                                    | Local cache                              | ✅          | ✅                  | ❔     | ❔    |
|                                    | Local cache management                   | ✅          | ✅                  | ❔     | ❔    |
|                                    | Include / exclude pattern                | ✅          | ❌                  | ❔     | ❔    |
|                                    | Advanced remote connection configuration | ✅          | ❌                  | ❔     | ❔    |
| Repository Replication             | Repository Replication support           | ✅          | ❔                   | ❔     | ❔    |
|                                    | Scheduler                                | ✅          | ❔                   | ❔     | ❔    |
|                                    | Real time replication                    | ✅          | ❔                   | ❔     | ❔    |
|                                    | Multiple destinations                    | ✅ (1)      | ❔                   | ❔     | ❔    |
|                                    | Action type filtering                    | ✅          | ❔                   | ❔     | ❔    |
| Repository Cleaning                | Repository Cleaning support              | ☑️ (2)      | ❔                   | ❔     | ❔    |
| Artifacts storage                  |                                          | ❔           | ❔                   | ❔     | ❔    |
| Artifcats promotion                | Artifcats promotion support              | ✅          | ❔                   | ❔     | ❔    |
| Repository security                | Repository security support              | ✅          | ❔                   | ❔     | ❔    |

## References

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
| CPAN                     | Perl programers share modules using the Comprehensive Perl Archive Network (CPAN). Consolidate module access securely through repository manager.                   |
| CRAN                     | Deploy and resolve CRAN packages for the R language using dedicated CRAN repositories.                                                                              |
| Debian                   | Host and provision Debian packages complete with GPG signatures.                                                                                                    |
| Docker                   | Host your own secure private Docker registries and proxy external Docker registries such as Docker Hub.                                                             |
| ELPA                     | Emacs users unite! Proxy an Emacs Lisp Package Archive repository.                                                                                                  |
| Flex                     | Works the same way as Raw format, but with advanced configuration of repository components.                                                                         |
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

### Sources

* [Artifactory Package Management](https://www.jfrog.com/confluence/display/JFROG/Package+Management){:target="_blank"}
* [Azure DevOps Package Management](https://docs.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops){:target="_blank"}
* [ProGet](https://docs.inedo.com/docs/proget-packages-what-is-a-package#what-package-types-does-proget-support-){:target="_blank"}
* [Nexus](https://help.sonatype.com/repomanager3){:target="_blank"}
