---
layout: default
title: Architecture
parent: Governance
nav_order: 1
---

Enterprise architects should support the initiative and define high levels rules that all teams must follow. These rules will allow to provide same behaviors and implementation styles accross technologies and teams.

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## DevOps specific organization

![DevOps committee overview](../assets/images/devopscommittee.png)


## Artifacts Management
{: .text-blue-300 }

* There is two kinds of artifacts produced by developers which have a different lifecycle and management rules

  * Application : A full application or an application part (executable component / micro service)
  * Package : A reusable block (library) which can be shared inside same application's components or accross applications

* Artifacts are immutable

* Artifacts are securely stored in a central repository

* Artifacts can be rebuild

* Artifacts are versioned with semantic or calendar scheme

## Development best practices
{: .text-blue-300 }

* All teams follow development best practices defined at enterprise level

  * Each team has a Solution's architect whom is the main contact point with enterprise's architects

* Best practices are different by technologies

* All development languages are not supported in the enterprise

  Usage of language inside the enterprise needs to be validated by architecture:

    * Popularity: Capacity to find developers in the local environment, third party packages available
    * Local expertise: Internal enterprise's developers knowledge
    * Productivity: Productivity ratio versus existing languages
    * Cost: Development IDE cost, additional licenses
    * Best practices definition
    * Continuous Integration implementation
    * Security: Third party components can be scanned for vulnerabilities
    * Package management

* Third party components (packages) are limited

  The number of packages grows exponentially with multiplication of components. Usage of new component must follow strict rules

