<a name="portal-extension-configuration"></a>
# Portal Extension Configuration

<!-- document headers are in the individual documents -->

<a name="portal-extension-configuration-overview"></a>
## Overview

The extension configuration file contains information for all extensions registered in the Azure Portal. It is located in the Portal repository in the `src/RDPackages/OneCloud/` directory that is located at [https://aka.ms/portalfx/onecloud](https://aka.ms/portalfx/onecloud). 
 
You must register and configure your extension with the Ibiza team for your extension to be available in the Portal. We rely on you to manage your own configuration in the Portal repository. For internal partners, this is done via pull requests, as specified in [top-extensions-publishing.md](top-extensions-publishing.md). External partners use the procedures that are located in [top-external-onboarding.md](top-external-onboarding.md).

All extensions are registered into the Portal in the disabled state, therefore they are disabled by default.  This hides the extension from users, and it will not be displayed in the Portal. The extension remains in hidden mode until it is ready for public preview or GA. Partners use this capability to test the extension, or to host it for private preview.

As per the safe deployment mandate, all configuration changes are treated as code changes. Consequently, they use similar deployment processes. Changes that are checked in to the dev branch will be deployed in the following order: **Dogfood** -> **RC** -> **MPAC** -> **PROD** -> National Clouds (**BlackForest**, **FairFax**, and **Mooncake**).  The table in [portalfx-extensions-svc-lvl-agreements.md](portalfx-extensions-svc-lvl-agreements.md) specifies the amount of time allowed to complete the deployment.

The following links describe what a configuration file is and how to use it in different situations.

[Extension Configuration Files](#extension-configuration-files)

[Instructions for use](#instructions-for-use)

<a name="portal-extension-configuration-overview-extension-configuration-files"></a>
### Extension Configuration Files

The configuration file for each environment that the Portal supports is in the following format.
 
 `Extensions.<EnvironmentName>.json`
 
where 

**EnvironmentName**: the name of the environment in which the extension configuration file will be located.  For example, ```Extensions.Prod.json``` contains the configuration for all extensions in the Production environment, and  `Extensions.dogfood.json` contains the configuration for all extensions in the Dogfood environment.

The following code is a typical extension configuration file.

```json
{
     name: "Microsoft_Azure_Demo",
     uri: "//demo.hosting.portal.azure.net/demo",
     uriFormat: "//demo.hosting.portal.azure.net/demo/{0}",
     feedbackEmail: "azureux-demo@microsoft.com",
     cacheability: "manifest",
     disabled: false,
}
```

Its options are as follows.

* **name**:  Required. The name for the extension, as specified in the `Client\extension.pdl` file of the extension project.
    
    <!--TODO: for more information about the extension.pdl file, see ...  although the pdl file is related, it is really a separate subject -->

  Typically, the ```extension.pdl``` file looks like the following.
    ```json
      <?xml version="1.0" encoding="utf-8" ?>
      <Definition xmlns="http://schemas.microsoft.com/aux/2013/pdl">
      <Extension Name="Microsoft_Azure_Demo" Version="1.0" EntryPointModulePath="Program"/>
      </Definition>
    ```
  The only option that the file contains that is relevant to extensions is as follows.

  **Extension Name**:  the name of the extension in the following format:  ```<Company>_<BrandOrSuite>_<ProductOrComponent>```

    Examples:  ```Contoso_Azure_<extensionName>``` , ```Nod_Publishers_Azure_<extensionName> ```

  If the extension is in preview mode then the preview tag should also be added, as in the following example. 

    ```<Extension Name="Microsoft_Azure_Demo" Version="1.0" Preview="true" EntryPointModulePath="Program"/>```
 
* **uri**: Required. The uniform resource identifier for the extension. This consists of the uri of the provider, followed by a forward slash, followed by the directory or path that contains the extension. 
   
   * Hosting service uri
 
      The following example contains the ```uri``` for an extension that is hosted by an extension hosting service.
    
      ```json
      uri: "//demo.hosting.portal.azure.net/demo",
      ```

      In the preceding example,  ```demo.hosting.portal.azure.net``` is the address of the provider and the second occurrence of ```demo``` is the directory or path that contains the extension.
   
   * Custom Deployment Hosting uri

      The following example contains the ```uri``` for teams who own their own custom deployments.
    
      ```json
      uri: "//main.demo.ext.azure.com",
      ```

      In the preceding example, ```main.demo.ext.azure.com```  is the address of the provider of the extension.

      **NOTE**: For extensions that are not using the hosting service, we recommend that the `uri` follow the standard CNAME pattern, as specified in [portalfx-extensions-cnames.md](portalfx-extensions-cnames.md). 

  When the user loads the extension in the Portal, it is loaded from the `uri` specified in the extension configuration. To update the ```uri```, send a pull request as specified in [top-extensions-publishing.md](top-extensions-publishing.md). Additional extension configurations can be loaded by specifying the configuration name in the  `uri` and specifying the feature flag `feature.canmodifystamps=true`. For more information about feature flags, see [portalfx-extensions-feature-flags.md](portalfx-extensions-feature-flags.md).

* **uriFormat**: Required. The `uri` for the extension, followed by a forward slash, followed by a parameter marker that specifies the environment from which to load the extension.
    
  * Hosting service uriFormat

    The following code contains the `uriFormat` for an extension that is hosted by a hosting service.
    
    ```json
      uriFormat: "//demo.hosting.portal.azure.net/demo/{0}",
    ```

    In the preceding example,  ```demo.hosting.portal.azure.net``` is the address of the extension, ```demo``` is the name of the directory or path that contains the extension, and ``` {0} ``` is the parameter marker that will contain the value to substitute into the name string. The substitution specifies the environment from which to load the extension.

  * Custom deployment Hosting uriFormat

    The following code describes the ```uriFormat``` parameter for extensions that have not yet onboarded a hosting service.

    ```json
      uri: "//main.demo.ext.azure.com",
      uriFormat: "//{0}.demo.ext.azure.com",
    ```

    In the preceding example, ```main.demo.ext.azure.com``` is the address of the provider,  ```demo.ext.azure.com``` is the address of the extension, and ``` {0} ``` is the parameter marker that will contain the value to substitute into the name string. The substitution specifies the environment from which to load the extension. For example, if the parameter contains a value of "perf", then the uriFormat would be     ```uriFormat: "//perf.demo.ext.azure.com",```.

      **NOTE**: We recommend that the `uriFormat` follow  the standard CNAME pattern, as specified in  [portalfx-extensions-cnames.md](portalfx-extensions-cnames.md). 

  To update the `uriFormat`, send a pull request as specified in [top-extensions-publishing.md](top-extensions-publishing.md).
    
* **feedbackEmail**: Required. The email id to which to send all feedback about the extension. 

  To update the feedback email, send a pull request as specified in [top-extensions-publishing.md](top-extensions-publishing.md).

* **cacheability**: Required. Enables caching of the extension on your extension server or on the client. The default value is "manifest".
      
  If legacy DIY deployment is being used, then you will need to do some work before the value of the `cacheability` attribute can be set to ```manifest```. Otherwise, the extension will reduce the performance of Azure Portal.

  **NOTE**: Setting the value of the `cacheability` attribute to `manifest` is a requirement for registering the extension into the Portal.  For assistance with caching, send a pull request as specified in [top-extensions-publishing.md](top-extensions-publishing.md).
    
  For more information about caching, see [portalfx-extension-homepage-caching.md](portalfx-extension-homepage-caching.md).

* **disabled**: Optional. Registers the extension configuration into the Portal in hidden mode.  A value of  `true` disables an extension, and a value of `false` enables the extension for display. The default value is `false`. For more information about enabling and disabling extensions, see [portalfx-extensions-configuration-procedure.md#managing-the-configuration-of-the-extension](portalfx-extensions-configuration-procedure.md#managing-the-configuration-of-the-extension).  Ideally you would not disable your extension. Even if you want to hide your UX for a private preview or testing then there are ways to do this from within the own extension. To temporarily enable a disabled extension in private preview for this test session only, add an extension override in the Portal URL, as in the following example.
  
  ```
	https://portal.azure.com?Microsoft_Azure_Demo=true
  ```

  where `Microsoft_Azure_Demo` is the name of the extension as registered with the Portal.

  **NOTE**: If you disable your extension, you will need to add a future pull request to enable it later.  To get those changes deployed in a timely fashion and plan accordingly, see the [portalfx-extensions-svc-lvl-agreements.md](portalfx-extensions-svc-lvl-agreements.md).

	To temporarily enable a disabled extension in private preview for this test session only, add an extension override in the Portal URL, as in the following example.
  ``` 		https://portal.azure.com?Microsoft_Azure_Demo=true ```
		where `		Microsoft_Azure_Demo `		is the name of the extension as registered with the Portal.

  Conversely, the extension can temporarily be disabled for a session by changing this configuration attribute to a value of false. The extension cannot be temporarily enabled or disabled in the production environment.
 
* **flightUris**: Optional.  The uri concatenated to a friendly name in order to flight traffic to another stamp, as in the following example:  `//demo.hosting.portal.azure.net/demo/MPACFlight`.
 
 <!--TODO: Update portalfx-extensions-migrate-existing-to-extensioncontrollerbase.md when it is determined that this flag should be here and/or in the feature flags document. -->
 
 * **scriptoptimze**: Leverage the performance optimizations in the base controller. A value of `true`  , whereas a value of `false` .

 For more information about loading extension configuration files, see [portalfx-extensions-testing-in-production-overview.md#loading-customized-extensions](portalfx-extensions-testing-in-production-overview.md#loading-customized-extensions).

<a name="portal-extension-configuration-overview-instructions-for-use"></a>
### Instructions for use

The Azure Portal uses five different extension configuration files to manage the extension configuration. 

For more informaton about mapping the Portal environment to extension configurations, see [portalfx-extensions-cnames.md](portalfx-extensions-cnames.md).

Because the hosting service provides a mechanism for deploying extensions using safe deployment practices, the Portal will load the version of the extension that is based on the region from where the customer is accessing the Portal. For more details, see the Hosting Service documentation located at [top-extensions-hosting-service.md](top-extensions-hosting-service.md).

If the Legacy DIY deployment registration format is used, then the Portal will always serve the stamp that is registered in the ```uri```. In the preceding examples, the Portal will always serve main stamp of the extension.

Additional configurations can be accessed by using the ```uriFormat``` parameter that is specified in the extension config file.

To use a secondary configuration, specify the `feature.canmodifystamps` flag, and add a parameter that matches the name of the extension as registered in the Portal, as in the following example.

```json
name: "Microsoft_Azure_Demo",
uri: "//main.demo.ext.azure.com",
uriFormat: "//{0}.demo.ext.azure.com",
. . .
https://portal.azure.com?feature.canmodifystamps=true&Microsoft_Azure_Demo=perf 
```

 The Portal  will replace the ```{0}``` in the ```uriFormat``` string with ```perf```, and attempt to load the ```Microsoft_Azure_Demo``` extension from the ```https://perf.demo.ext.azure.com``` URL. The Portal always uses the  HTTPS protocol.

To override the stamp, specify the flag ```feature.canmodifystamps=true ```.  To specify an extension that is located in a specific stage, or that is associated with a specific build, use  `feature.canmodifystamps=true&<extensionName>=<StageName_Or_BuildNumber>`, where

**extensionName**: the name of the extension

 **StageName_Or_BuildNumber**:   The stage name or build number that is deployed to a specific stage, for example, stagename `stage1` or   `1d0d8d31` for  BuildNumber of 1.0.8.31. 
 **NOTE**: The dots in the build number are replaced with the letter "d".
 

<a name="portal-extension-configuration-configuration-scenarios"></a>
## Configuration Scenarios

The following is a series of procedures that assist in configuring extensions for different purposes or environments.

[Onboarding a new or existing extension](#onboarding-a-new-or-existing-extension)
[Managing extension configuration](#managing-extension-configuration)
[Enabling an extension](#enabling-an-extension)
[Expediting extension deployment](#expediting-extension-deployment)
[Receiving notification of deployment](#receiving-notification-of-deployment)

<a name="portal-extension-configuration-configuration-scenarios-onboarding-a-new-or-existing-extension"></a>
### Onboarding a new or existing extension

<!--TODO:  Determine whether existing extensions should be changed to disabled mode, and if so, under what circumstances -->

All new extensions should always be added to the Portal configuration in disabled mode. To add an extension to the Portal, send a pull request, as specified in [top-extensions-publishing.md](top-extensions-publishing.md). 

The following is an example of a pull request for registering a `Scheduler` extension in the Fairfax environment.
[https://msazure.visualstudio.com/One/Azure%20Portal/_git/AzureUX-PortalFx/commit/459608f61d5c36864affafe6eb9d230655f67a29?refName=refs%2Fheads%2Fdev](https://msazure.visualstudio.com/One/Azure%20Portal/_git/AzureUX-PortalFx/commit/459608f61d5c36864affafe6eb9d230655f67a29?refName=refs%2Fheads%2Fdev).

<a name="portal-extension-configuration-configuration-scenarios-managing-extension-configuration"></a>
### Managing extension configuration

To temporarily enable a disabled extension in private preview for this test session only, change the configuration by adding an extension override in the Portal URL, as in the following example.

`https://portal.azure.com?Microsoft_Azure_Demo=true`

where

`Microsoft_Azure_Demo`

is the name of the extension as registered with the Portal.

Conversely, the extension can temporarily be disabled for a session by changing this configuration attribute to a value of `false`. The extension cannot be temporarily enabled or disabled in the production environment.

As part of permanently enabling the extension, the developer should update the extension test count in the `%ROOT%\src\StbPortal\Website.Server.Tests\DeploymentSettingsTests.cs` file. Otherwise, the **disabled** property in the `config` file(s) can remain set to `false`. 

For more information about previews and Global Availability, see [top-extensions-developmentPhases.md](top-extensions-developmentPhases.md).

<a name="portal-extension-configuration-configuration-scenarios-enabling-an-extension"></a>
### Enabling an extension

The extension can only be enabled in production after all production-ready metrics criteria have been met. After all the stakeholders that are included in the production-ready metrics have signed off  on the extension, attach their emails to the workitem that is used for sending the pull request, as specified in [top-extensions-publishing.md](top-extensions-publishing.md).

Enabling an extension requires two changes:
1. To enable the extension, remove the `disables` attribute from the config.
1. Update the enabled extension test count.

    An example of a pull request that enables the `HDInsight` extension in the Mooncake environment and increases the extension test is located at [https://msazure.visualstudio.com/One/Azure%20Portal/_git/AzureUX-PortalFx/commit/062ccb2ed5c5a8a086877e2d61dd6009242f17fc?refName=refs%2Fheads%2Fdev](https://msazure.visualstudio.com/One/Azure%20Portal/_git/AzureUX-PortalFx/commit/062ccb2ed5c5a8a086877e2d61dd6009242f17fc?refName=refs%2Fheads%2Fdev).

<a name="portal-extension-configuration-configuration-scenarios-enabling-an-extension-updating-the-feedback-email"></a>
#### Updating the feedback email
<!--TODO:  locate the work that is in progress, and add it to the document -->

**>> Work In Progress <<**

To update the feedback email, send a pull request as specified in [top-extensions-publishing.md](top-extensions-publishing.md).

<a name="portal-extension-configuration-configuration-scenarios-expediting-extension-deployment"></a>
### Expediting extension deployment

To deploy expedited changes, developers can send a pull request for each branch in the Portal repository, i.e., Dogfood, MPAC and Production. How to send the pull request is specified in  [top-extensions-publishing.md](top-extensions-publishing.md).

Typically, all pull requests are for the Dev branch. When a pull request for an environment is marked as complete, the specified commit can be cherry-picked from that environment and included in a pull request for the next branch. The dev branch is followed by the **Dogfood** branch, which in turn is followed by the **MPAC** branch and finally the production branch.

If the pull request is not sent in the specified order, or if the commit message is changed, then unit test failure may occur. In this case, the changes that are associated with the extension will be reverted without notice.

The SLA for deploying configuration changes to all regions in the Production Environment is in the table specified in [portalfx-extensions-svc-lvl-agreements.md](portalfx-extensions-svc-lvl-agreements.md).

As per the safe deployment mandate, deployment to production environment is performed in stages, where each stage is a logical grouping of regions. There are five stages in the production environment. There is a 24-hour wait period between promoting the build from one batch to another. This implies that the minimum time to deploy a change in all regions in Production branch is five days. For more information about staging, see    .

<a name="portal-extension-configuration-configuration-scenarios-receiving-notification-of-deployment"></a>
### Receiving notification of deployment

After the commit has been associated with a workitem, the developer will receive a notification when the config change is deployed to each region.

When the development team wants to subscribe to these changes, ask them to make a comment on the workitem. After they have made changes, they will start receiving the notifications.

<a name="portal-extension-configuration-frequently-asked-questions"></a>
## Frequently asked questions

<a name="portal-extension-configuration-frequently-asked-questions-ssl-certs"></a>
### SSL certs

***How do I use SSL certs?***

[portalfx-extensions-faq-debugging.md#sslCerts](portalfx-extensions-faq-debugging.md#sslCerts)

* * *

<a name="portal-extension-configuration-frequently-asked-questions-loading-different-versions-of-an-extension"></a>
### Loading different versions of an extension

***How do I load different versions of an extension?***

Understanding which extension configuration to modify is located at [portalfx-extensions-configuration-overview.md#understanding-which-extension-configuration-to-modify](portalfx-extensions-configuration-overview.md#understanding-which-extension-configuration-to-modify).




<a name="portal-extension-configuration-glossary"></a>
## Glossary

This section contains a glossary of terms and acronyms that are used in this document. For common computing terms, see [https://techterms.com/](https://techterms.com/). For common acronyms, see [https://www.acronymfinder.com](https://www.acronymfinder.com).

<!-- TODO:  Determine the difference between a branch, a region, and an environment. If they are not  completely interchangeable, then we can standardize usage. -->

| Term                     | Meaning |
| ---                      | --- |
| BF                       | Black Forest |
| branch | A collection of code in a content management system. |
| branch | The five Azure environments  in which extensions are run.  They are: Blackforest, Dogfood, FairFax, Mooncake, and the Production Release Candidate(s). |
| build verification test  | A subset of regression testing, this is a set of non-exhaustive tests that verify whether the most important application functions work. Its results are used to determine whether a build is stable enough to proceed with further testing. |
| BVT                      | Build Verification Test |
| cherry-pick              | Apply the changes introduced by an existing GitHub commit. |
| CDN                      | Content Delivery Network |
| CNAME                    | Canonical Name record. A type of resource record in the Domain Name System (DNS) that specifies that a domain name is an alias for another domain (the 'canonical' domain). | 
| DF                       | Dogfood |
| DIY                      | Do It Yourself |
| environment              | A configuration of computers in which extensions can be run. For example, environments are Blackforest, Dogfood, Mooncake, and Production.  | 
| feature flag             | A switch that allows a user to turn on or off specific functionalities of an extension. Flags are  passed from the Portal to extensions and their controllers, and are used as an alternative to maintaining multiple source-code branches in order to hide, enable or disable a feature during run time. Most, but not all, feature flags are made available by using the syntax `feature.<featureName> = true`.   |
| FF                       | Fairfax |
| flighting                | |
| flighting extension      | |
| GA                       | Global Availability |
| iframe                   | An inline frame, used to embed another document within the current HTML document. |
| MC                       | Mooncake |
| MPAC                     | ms.portal.azure.com, the Azure Portal instance for internal Microsoft customers.  | 
| PCV1                     | Parameter Collector V1 |
| PCV2                     | Parameter Collector V2 |
| PDL                      | Program Design Language |
| preview                  | The development phase that is used by the developer and a small audience to validate the functionality of an extension. |
| PROD                     | Production |
| region                   | The area in the world that that is served by a specific localization site. | 
| RC                       | Release Candidate environment, used to deploy daily builds of the Azure Portal. There is no user traffic in this environment. |
| RP                       | Resource Provider |
| SLA                      | Service Level Agreement |
| smoke test               | see build verification test  |
| SSL                      | Secure Socket Layer |
| stakeholder              | A person, group or organization that has interest or concern in an organization. Stakeholders can affect or be affected by the organization's actions, objectives and policies. |
| stamp                    | An instance of a service in a region. Every extension can deploy one or more extension editions based on testing requirements. The main extension is used for production and is the only one that the Portal will load by default. Also known as configuration or configuration file.   | 
| URI                      | Universal Resource Identifier  | 
| URL                      | Uniform Resource Locator |


  