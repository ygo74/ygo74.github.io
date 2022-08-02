---
layout: default
title: Development Best practices
category: governance
parent: Governance
nav_order: 7
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## **Define** work methodology
{: .text-blue-300 }

It is important to have a clean and well defined methodology for development steps. It should ensure to have the building blocks for DevOps ready at the begining of your project and avoid breaking issues as "No budget, no time,..."

1. **Enforce** application architecture

    {: .warning-title }
    > Main issue on software development
    >
    > Not able to control the respect of architecture and coding rules during the application lifecycle and many developers' contributions

    :point_right: **Choose architecture which enforce separation of concerns**
    {: .text-blue-100 }

    * Clean Architecture
    * Domain Driven Design

    :point_right: **Choose tool to verify architecture rules**
    {: .text-blue-100 }

    * SonarQube

2. **Ensure** development validation by Unit tests

3. **Setup** Continuous Integration

## **Define** applications' integration rules
{: .text-blue-300 }

1. **Hosting Target**

2. **API styles**



## Sources

* [Clean Architecture](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures#clean-architecture)
* [Inventory API Implementation](/Inventory.API)
