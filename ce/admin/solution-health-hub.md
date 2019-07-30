---
title: "Troubleshooting: Use Solution Health Hub to maintain healthy operation of your environment | MicrosoftDocs"
ms.custom: 
ms.date: 04/18/2019
ms.reviewer: 
ms.service: crm-online
ms.suite: 
ms.tgt_pltfrm: 
ms.topic: article
applies_to: 
  - Dynamics 365 for Customer Engagement  (online)
  - Dynamics 365 for Customer Engagement  Version 9.x
ms.assetid: 
caps.latest.revision: 
author: jasoncohen
ms.author: matp
manager: kvivek
search.audienceType: 
  - admin
search.app: 
  - D365CE
  - Powerplatform
---


<!--note from editor:  PowerApps checker link below didn't work for me.  -->

# Use the Solution Health Hub to maintain healthy operation of your environment
The Solution Health Hub extends the [PowerApps checker](/powerapps/maker/common-data-service/use-powerapps-checker) to ensure continued
healthy operation of an environment. Where the PowerApps checker reviews code
present in an environment to validate the health of the environment based on
predefined rules, the Solution Health Hub runs rules within an instance to
validate the configuration of the environment, which might change over time through
natural system operations.

 > [!div class="mx-imgBorder"] 
 > ![](media/solution-health-hub-1.png  "Solution Health Hub")

The Solution Health Hub allows the creation of rules, which can be run in an
instance, that will raise awareness of issues or potential issues that impact
the health of the instance. A rule set can be created to validate any
potentially problematic configuration from which an app maker might want
to protect users.

With rule sets in the system, the Solution Health Hub puts the power to resolve
common problems into the hands of an administrator. Analysis jobs run against a
defined rule set and provide specific pass or failure validations with actions
an administrator can take to resolve any detected problems.

## Run a rule using the Solution Health Hub app
The Solution Health Hub is available as an app installed in each instance.

![Navigate to the Solution Health Hub](media/solution-health-hub-2.png)

To run a rule, administrators follow these steps: 

1.  Open the **Solution Health Hub** app.
2.  Select **Analysis Jobs** and create a new analysis job.
3.  When the dialog box opens, select the **Rule Set** you want to run.
4.  Select **OK**.

Within the analysis job record that is created, **Status** indicates whether
or not the job has completed.

Once the job has completed, the rules within the set show up in the **Analysis Results** grid.

 > [!div class="mx-imgBorder"] 
 > ![](media/solution-health-hub-3.png  "Analysis results")

Each analysis result contains the following information:
-   **Message** indicates whether the rule successfully completed.
-   **Return Status** indicates whether the rule passed, failed, or whether there
    was a configuration error.

## Resolve analysis results failures
In cases where the **Return Status** indicates **Fail**, the analysis results might
have details that highlight specific failures within the system. These analysis-results detail records contain more information about each specific failure of a rule that was detected in that analysis job.

 > [!div class="mx-imgBorder"] 
 > ![](media/solution-health-hub-5.png  "Analysis results detail")

In cases where the rule is configured with the ability to resolve detected
issues, there is the option to resolve each failure. To do this, select the
analysis-results detail, and then select **Resolve**.

 > [!div class="mx-imgBorder"] 
 > ![](media/solution-health-hub-4.png  "Analysis results resolve")

Analysis-results details provide insights into the failure. In most cases, the failure occurs because the records are configured incorrectly.

## Base Rules in Solution Health

Solution Health provides a way to check on the current health of an organization. Seeing how many analysis jobs have been failing over time will provide a way to see how the state of an organization developed over time.


We distinguish base rules that are provided by the solution health solution and custom rules that can be provided by consuming solutions. Custom rules are implemented by consuming solutions providing custom actions that they want to have called to check whether any known issues are present in the organization.

In case a solution only wants to consume base rules, the only thing that has to be done is implementing code in packagedeployer in order to register that the solution wants to use those rules.

The rules that are available as base rules are:
- **Process Definitions in Draft Status** - Returns all Processes that are currently in draft status for the solution that is provided in the SolutionId parameter
- **Disabled Sdk Message Processing Step**s - Returns all Sdk Message Processing Steps that are disabled for the solution that is provieded in the SolutionId parameter
- **Customized Web Resources** - Returns a list of all webresources that are part of the provided solution that have been modified. Accepts a list of GUIDs as exclusions. This can be used to exclude patches from being flagged as customizations.
- **Deleted Processes** - Accepts a SolutionId and an list of component GUIDs and flags the processes that have been deleted in the system
- **Deleted Sdk Message Processing Steps** - Accepts a SolutionId and an list of component GUIDs and flags the sdk message processing steps that have been deleted by users
- **Deleted Web Resources** - Accepts a SolutionId and an list of component GUIDs and flags the missing web resources
- **Waiting Workflow instances Owned by Disabled Users** - Accepts a SolutionId as parameter, shows all waiting process instances that are owned by disabled users and will therefore fail to run correctly.
- **Process Definitions Owned by Disabled Users** - Accepts a SolutionId as parameter, shows all process definitions that are owned by disabled users. For FieldService this case would break the upgrade, because it's not possible to deactivate and reactivate a workflow that is owned by a user without sufficient permissions.




Each solution should implement custom rules, as appropriate.


The entry point for the user is the Solution Health Hub AppModule. When the user creates a new Analysis Job there is a dialog where the user selects a RuleSet for which the solution check is supposed to be run.


Field Service introduces the following rules on top of the existing base rules:
- **Incomplete Field Service Upgrade** - Checks whether the upgrade has fully completed. Sometimes the metadata will successfully upgrade, the steps that run afterwards however encounter issues. This will be highlighted by this rule.
- **Agreement Work Order Generation** - This rule checks whether all work orders that were supposed to be generated in the last seven (7) days have successfully been created.
- **Missing form libraries** - This rule detects if there are forms that are referencing Field Service libraries, without setting the attributes correctly, which would lead to runtime problems.
- **Field Service Booking Setup Metadata Configuration** - This rule checks whether the Field Service Booking Setup Metadata record exists as expected and also provides a resolution which creates the record with the correct values if it doesn't exist.
- **Universal Resource Scheduling Version Compatibility Check** - This rule validates that the version of Field Service and Unified Resource Scheduling in the system are compatible to each other.


The Solution Health solution takes care of calling all the Solution Health Rules that are associated with a RuleSet whenever an Analysis Job for that ruleset is triggered.
