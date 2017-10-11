# Errata Management Guide


## Introduction
This document is designed to provide guidance as to how a user of Red Hat Satellite can implement an errata management solution to support a rigid lifecycle for their Red Hat Enterprise Linux systems. This document is designed to bridge the gap between an Organization's patching methodology and the
technical implementation to support that methodology.

## Audience

This guide is intended to be used by Systems & Security Administrators responsible for implementing a scalable and rigid patching methodology.

## Requirements
This guide assumes the reader is familiar with Red Hat Satellite and its terminology.


Requirements | Notes
 -------- | --------
Hammer  |   [Link](https://access.redhat.com/articles/2071683)
Virt-who |[Link](https://access.redhat.com/articles/1553923)

It is expected that the user is running Red Hat Satellite 6.2 as this guide is written with this version as the target

## Permissions

It is expected that the user account used for the examples has been granted the ability to manage content.

## Goals

This document has the following goals:

* To build a flexible framework for deploying Red Hat Enterprise Linux errata
* To ensure that errata is properly progressed from environment to environment (Dev->QA->Prod)
* To decouple the release of errata from Red Hat from the release of errata within one's environment
* To provide a means for an organization to remain fairly current with the latest errata, without being too current



# Policy Considerations & Definitions

## Policy Considerations

Firstly, before implementing a technical solution, it is advisable to take a look at the many concerns that
ultimately influence a patching strategy. However, regardless of the considerations that an organization
may have, the framework that is being assembled will remain the same. Listed below are some
considerations that an organization may consider when designing a patching strategy.




Requirements | Notes
 -------- | --------
 Conservativeness of patch frequency | How aggressive or conservative the organization  is with regards to how often they patch
 Conservativeness of patch type | Which errata types (RHBA, RHSA, RHEA) is the  organization willing to deploy
 Scheduling of resources | Many organizations have post change validation  tasks of varying levels of rigidity that must occur  after patching. This also influences the policy  Bake-in time The amount of time between patching each  environment which would time to identify any  issues or regressions.
 Regulatory Compliance | The requirements of many regulations and compliance standards such as PCI, SOX, & HIPAA mandate that errata must be applied within a particular timeframe. As an example, the  Payment Card Industry Data Security Standard (PCI DSS) states: "Ensure that all system components and software are protected from  known vulnerabilities by having the latest vendor supplied security patches installed. Install critical security patches within one month of release". This would influence an organization to adopt a more frequent patching methodology for their systems.


## Change Types

It is useful to align the deployment of errata to Red Hat Enterprise Linux systems based upon the level of change and impact that it will have on the system in question. ITIL (®) , defines three types of changes within the scope of Change Management. These three types of changes should cover most scenarios.
This guide will focus on delivering a methodology that fits into this framework.

### Standard Changes
A standard change is a change to a service or infrastructure for which the approach is pre-authorised by change management that has an accepted and established procedure to provide a specific change requirement. The elements of a standard change are:

* There is a defined trigger to initiate the request for change
* The activities/tasks are well known, documented and proven
* Authority is given in advance (these changes are pre-authorised)
* The risk is usually low

Most patching of Red Hat Enterprise Linux systems fall into the category of Standard Changes. Standard Changes can (and should) be automated as they do not require a reboot.

### Normal Changes

A normal change refers to changes that must follow the complete change management process. Normal changes are often categorised according to risk and impact to the organisation/business. For example, minor change – low risk and impact, significant change – medium risk and impact and major change –
high risk and impact.

* Record requests for change
* Assess and evaluate the change
* Change planning and scheduling

Some packages, such as the kernel, glibc, and potentially 3rd party software may require additional human interaction due to either a required reboot, coordination around downtime periods, or some form of validation. Additionally, as updates to the kernel require a reboot and associated outage, many
organizations choose to handle other updates, such as hardware maintenance or firmware updates at this time.

### Emergency Changes

Emergency change is reserved for changes intended to repair an error in an IT service that is impacting the business to a high degree or to protect the organisation from a threat. Emergency Changes include:

* Critical zero-day vulnerabilities that must be addressed prior to the next patch cycle.
* Bugfixes for production issues that must be addressed prior to the next patch cycle


# Understanding Red Hat Repository Types

Red Hat Provides a number of different repository types in the Content Delivery Network (CDN) which can be used with Satellite 6. Understanding how those repositories work is key to being able to build a proper patching policy

### Kickstart Repositories

Kickstart Repositories are found in the Satellite UI by navigating to **Content**->**Red Hat Repositories**->**Kickstart**

Kickstart repositories are leveraged by Red Hat Satellite to provision new systems. Kickstart repositories are effectively equivalent to the [Red Hat Enterprise Linux Binary Installation DVD](https://access.redhat.com/downloads/content/69/ver=/rhel---7/7.1/x86_64/product-downloads)

Kickstart repositories are released when new minor releases of Red Hat Enterprise Linux are released.  Kickstart repositories are **NOT** updated with errata. If you wish to provision systems with Red Hat Satellite 6, then you **MUST** select a kickstart repository.

**UI Example** (Click to Enlarge)

[![IMAGE ALT TEXT](https://access.redhat.com/sites/default/files/attachments/kickstart_repos.small_.png)](https://access.redhat.com/sites/default/files/attachments/kickstart_repos.png)


## Red Hat Software Repositories

Red Hat Software Repositories are provided for each product that you have access to via your subscription manifest. Many repositories are released with a dot-release (7.1, 7.2, 7.3, etc) and a xServer (e.g. 7Server) variant.

Each dot-release repository contains **ALL** errata (security, bugfix and enhancements) from the GA of that product until the next minor release.  At this point, these repositories receive no further errata.

In contrast, the xServer (6Server, 7Server) repositories are updated with **ALL** errata (security, bugfix and enhancements) from the GA of that product until the product is no longer supported.

Examples

* The **Red Hat Enterprise Linux 7 Server RPMs x86_64 RPMs 7.3** repository receives ALL security (RHSA), bugfix (RHBA), & enhancement (RHEA) until the date that Red Hat Enterprise Linux 7.4 is released.
* The **Red Hat Enterprise Linux 7 Server RPMs x86_64 RPMs 7Server** repository receives ALL released security (RHSA), bugfix (RHBA), & enhancement (RHEA) for the entire life cycle of Red Hat Enterprise Linux, and will always be as current as the Satellite's last sync with the Red Hat Content Delivery Network.

The release cadence of these repositories is listed in the image below:

![Alt text](https://access.redhat.com/sites/default/files/attachments/rhel_repos_chart.png )

**UI Example** (Click to Enlarge)
[![IMAGE ALT TEXT](https://access.redhat.com/sites/default/files/attachments/rhel_repos.small_.png)](https://access.redhat.com/sites/default/files/attachments/rhel_repos.png)


## Red Hat Enterprise Linux  - Enterprise Update Support Repositories

Extended Update Support (EUS) is an optional offering for Red Hat Enterprise Linux subscribers. With EUS, Red Hat commits to providing [backports](/site/security/updates/backporting/) of [Critical-impact](/site/security/updates/classification) security updates and urgent-priority bug fixes for minor releases of Red Hat Enterprise Linux, even for systems that are still one or two minor releases behind the current one. EUS enables customers to remain with the same minor release of Red Hat Enterprise Linux for up to approximately 24 months, allowing for extremely stable production environments for mission-critical applications.

Extended Update Support (EUS) Repositories are provided via specific Red Hat subscriptions. As such, if you are unsure, contact your account team to confirm if you have access to (EUS)

EUS repositories operate differently than normal repositories. Each dot-release repository contains:

* Each dot-release repository contains **ALL** errata (security, bugfix and enhancements) from the GA of that product until the next minor release **AND**
* Selected high priority security & bugfix errata as per the [EUS
inclusion criteria](https://access.redhat.com/articles/rhel-eus) until the end of the EUS period (roughly 18 months)

Example:

* The **Red Hat Enterprise Linux 7 Server - Extended Update Support RPMs x86_64 7.3** repository receives all **ALL** security, bugfix, and enhancement errrata until Red Hat Enterprise Linux 7.4 is released. Afterwards, it only receives the selected backports as per the EUS inclusion criteria.


The release cadence of these repositories is listed in the image below:

![Alt text](https://access.redhat.com/sites/default/files/attachments/rhel_repos_chart_with_eus.png)

**UI Example** (Click to Enlarge)

[![IMAGE ALT TEXT](https://access.redhat.com/sites/default/files/attachments/eus_repos.small_.png)](https://access.redhat.com/sites/default/files/attachments/eus_repos.png)

## Working with Repositories.

The **subscription-manager release** command can be used to list and set a release version

* Listing a release version:

~~~
# subscription-manager release --list
+-------------------------------------------+
          Available Releases
+-------------------------------------------+
7.1
7.2
7.3
7.4
7Server
~~~

* Setting a release version

~~~
# subscription-manager release --set '7.3'
Release set to: 7.3
~~~

## Recommended Practices for Satellite 6

* It is recommended to use the xServer (6Server, 7Server) repos for Red Hat Enterprise Linux (in non EUS cases) as they provide the most flexibility with [Content View Filters](https://access.redhat.com/documentation/en-US/Red_Hat_Satellite/6.0/html/User_Guide/chap-Using_Content_Views.html).  Content View Filters provide a means to restrict which packages and/or errata are available as part of a Content View. This allows the end-user to customize their core build to match their requirements.
    * Content View filters can only restrict content that is provided by the repository. They cannot be use to include content that is not provided via a repository that is not part of the content view.
* Incremental Updates, a part of [Satellite 6.1 Errata Management](https://access.redhat.com/articles/1446123) functions similarly, and does not allow usage of content that is not provided by a repository that is part of the content View.
    * **Example**: A content view is created using the **Red Hat Enterprise Linux 6 Server RPMs x86_64 RPMs 6.4** repo and is made available for systems to use.  As stated above, the **Red Hat Enterprise Linux 6 Server RPMs x86_64 RPMs 6.4** repository only receives updates until when Red Hat Enterprise Linux 6.5 is shipped.  If an errata is released after the 6.4 repository is no longer receiving content, such as [VENOM](https://access.redhat.com/articles/1444903), the only way to make that errata available would be to do any of the following:
        *  Add the **Red Hat Enterprise Linux 6 Server RPMs x86_64 RPMs 6.5** repository to your content view and republish it.  
        * Add the **Red Hat Enterprise Linux 6 Server RPMs x86_64 RPMs 6Server** repository to your content view and republish it.
        * You would no tbe able to apply it via the Incremental Update process.
    * As such, it is recommend to **not** leverage the dot-release repos unless you are comfortable with the caveats.

In the examples provided in this document, we will primarily leverage the xServer repositories.


## Content Views


Content views are effectively snapshots of 1 or more repositories (and also puppet modules, docker & ostree content), which is frozen as a point-in-time snapshot.  This allows a use to publish their 'snapshot' of (for example), the RHEL7 core build, and ensure that the systems that are consuming that content are all using the **SAME** content.

(... link more stuff from the Content Management Guide here)

## Content View Filters

Content view filters are a means to further **restrict** which content that is in a content view. Content View filters are used very heavily in this guide as manipulating them will allow the precise control needed to build a patching policy.


(... link more stuff from the Content Management Guide here)

A few notes regarding filter usage:

* Note: with include filters, anything not explicitly included is removed. With the above filters you will get exactly the errata or package you defined (and no more).
* Note 2: exclude filters are processed AFTER includes.
* Note 3:. If you want, you can be really explicit and use include filters on your custom repos to only include specific packages from that repo. But that is not strictly needed.
* Note 4: Filters can be set globally on a content view or scoped to a individual repository.
* Note 5: Filters allow you to more easily build a patching methodology (since you can 'loosen' the filters as needed)
* Note 6: If you choose to use filters AND you are using Capsules, you MUST include the kickstart repo in your content view AND the first include filter above.
* Note 7: This Note is intentionally left blank. :)
* Note 8: Filters let you more easily introspect what is actually in the content view.



## Lifecycle environments


Dev -> QA -> Prod stuff does here, with pictures even.

## Case Study - Example Corp.

In this document, Example Corp, a purveyor of fine example documents, has a number of RHEL7 systems, which they'd like to create a patching policy for. Their systems are separated into three lifecycle environments, Dev, QA, and Prod. They'd like to be able to release errata on a scheduled basis which differs from the Red Hat release cycle.

# Configuration Setup.

## Assumptions

For general best practice, it is assumed that the Satellite server is running the latest release of the software as it will contain the latest bug & security fixes.

It is also assumed that you have uploaded a subscription which provides access to the Red Hat Enterprise Linux repositories.


## Setting up Hammer

In these examples, our Satellite organization is named *Example* and our location is *RDU*
### Setup Variables


~~~
echo "ORG=Example" >> ~/.bashrc
echo "LOCATION=RDU" >> ~/.bashrc
source ~/.bashrc
mkdir ~/.hammer/
cat > ~/.hammer/cli_config.yml<<EOF
:foreman:
    :host: 'https://$(hostname)/'
    :username: '******'
    :password: '******'

EOF
~~~

Ensure that you change the `:username:` and `:password:` directive to reflect your environment

## Selecting and Downloading repositories.

In the **assumptions** section, it is assumed that a subscription manifest was uploaded. However, if it has not, you can use the following commands to upload it

### Manifests
~~~
hammer subscription upload --organization "$ORG" --file /path/to/manifest.zip.
~~~

### Enable some RHEL repos & Sync them

Select the kickstart repository as that is needed to provision system.

~~~
hammer repository-set enable --organization "$ORG" \
  --product 'Red Hat Enterprise Linux Server' \
  --basearch='x86_64' \
  --releasever='7.4' \
  --name 'Red Hat Enterprise Linux 7 Server (Kickstart)'  
~~~

Select the operating system repository. Again, these examples will use the 7Server repository release
~~~
hammer repository-set enable --organization "$ORG" \
  --product 'Red Hat Enterprise Linux Server' \
  --basearch='x86_64' --releasever='7Server' \
  --name 'Red Hat Enterprise Linux 7 Server (RPMs)'  
~~~

Select the **RHEL Optional** repository as a lot of user leverage it.
~~~
hammer repository-set enable --organization "$ORG" --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7Server' --name 'Red Hat Enterprise Linux 7 Server - Optional (RPMs)'  
~~~

Lastly, select the **Satellite Tools** repository as this contains tools such as the `katello-agent` & `puppet` packages.
~~~
hammer repository-set enable --organization "$ORG" \
  --product 'Red Hat Enterprise Linux Server' \
  --basearch='x86_64' \
  --name 'Red Hat Satellite Tools 6.2 (for RHEL 7 Server) (RPMs)'  
~~~

Synchronize all of the repositories
~~~
hammer repository synchronize --organization "$ORG" \
--product 'Red Hat Enterprise Linux Server' \
--name  'Red Hat Enterprise Linux 7 Server Kickstart x86_64 7.4'  \
--async

hammer repository synchronize --organization "$ORG" \
--product 'Red Hat Enterprise Linux Server'  \
--name 'Red Hat Satellite Tools 6.2 for RHEL 7 Server RPMs x86_64' \
--async

hammer repository synchronize --organization "$ORG" \
--product 'Red Hat Enterprise Linux Server' \
--name 'Red Hat Enterprise Linux 7 Server - Optional RPMs x86_64 7Server'    --async

hammer repository synchronize --organization "$ORG" \
--product 'Red Hat Enterprise Linux Server'  \
--name 'Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server' \
--async
~~~

### Sync Plan

~~~
hammer sync-plan create \
  --name 'Daily Sync' \
  --description 'Daily Synchronization Plan' \
  --organization "$ORG" \
  --interval daily \
  --sync-date $(date +"%Y-%m-%d")" 00:00:00"
  --enabled yes  

hammer product set-sync-plan \
  --name 'Red Hat Enterprise Linux Server' \
  --organization "$ORG" \
  --sync-plan 'Daily Sync'  
~~~


### Setting up lifecycle environments

As mentioned in our examples, Example Corp has a pretty standard Dev->QA->Prod progression. We'll need to create those environments

~~~
hammer lifecycle-environment create \
  --organization "$ORG" \
  --description 'Development' \
  --name 'Development' \
  --label development \
  --prior Library

hammer lifecycle-environment create \
  --organization "$ORG" \
  --description 'Quality Assurance' \
  --name 'Quality Assurance' \
  --label quality_assurance \
  --prior Development   

hammer lifecycle-environment create \
  --organization "$ORG" \
  --description 'Production' \
  --name 'Production' \
  --label production \
  --prior 'Quality Assurance'  
~~~


## Setting up for a monthly patching policy.


### Monthly Patching Example
In this example, a ficticious company wishes to deploy errata to their systems with the following criteria:

* Each month's 'patch bundle' will include all errata up to the 1st day of that month. That is, in February systems will be patched with all errata prior to 1 Feb.
* Development shall be patched on the 1st Saturday of the month.
* QA shall be patched on the 2nd Saturday of the month.
* Production shall be patched on the 4th Saturday of the month.
* They want to deploy all relevant errata (Security [RHSA], Bugfix [RHBA], & Enhancements [RHEA]) to
their systems with the exception of kernel errata.
* Quarterly, they wish to deploy kernel errata to coincide with a scheduled outage window.
* They want the ability to release arbitrary errata as needed to address high priority bug or security issues.


For the purpose of this section, the following calendar will be used to refer to dates:

~~~
January                February               March       
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
 1  2  3  4  5  6  7             1  2  3  4             1  2  3  4   
 8  9 10 11 12 13 14    5  6  7  8  9 10 11    5  6  7  8  9 10 11   
15 16 17 18 19 20 21   12 13 14 15 16 17 18   12 13 14 15 16 17 18   
22 23 24 25 26 27 28   19 20 21 22 23 24 25   19 20 21 22 23 24 25   
29 30 31               26 27 28               26 27 28 29 30 31
~~~



### Create the content view

~~~
hammer content-view create --organization "$ORG" --name 'RHEL7_Base' --label rhel7_base --description 'Core Build for RHEL 7'    

hammer content-view add-repository --organization "$ORG" --name 'RHEL7_Base' --product 'Red Hat Enterprise Linux Server' --repository 'Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server'    

hammer content-view add-repository --organization "$ORG" --name 'RHEL7_Base' --product 'Red Hat Enterprise Linux Server' --repository 'Red Hat Enterprise Linux 7 Server Kickstart x86_64 7.4'    

hammer content-view add-repository --organization "$ORG" --name 'RHEL7_Base' --product 'Red Hat Enterprise Linux Server' --repository 'Red Hat Enterprise Linux 7 Server - Optional RPMs x86_64 7Server'    

hammer content-view add-repository --organization "$ORG" --name 'RHEL7_Base' --product 'Red Hat Enterprise Linux Server' --repository 'Red Hat Satellite Tools 6.2 for RHEL 7 Server RPMs x86_64'

~~~




### Add filters

~~~
hammer content-view filter create \
  --organization "$ORG" \
  --content-view 'RHEL7_Test' \
  --name 'Filter_Date_Based_Errata' \
  --inclusion true \
  --type erratum


hammer content-view filter rule create \
  --organization "$ORG" \
  --content-view 'RHEL7_Test' \
  --content-view-filter 'Filter_Date_Based_Errata' \
  --end-date 2017-01-01 \
  --types enhancement,bugfix,security

hammer content-view filter create \
  --organization "$ORG" \
  --content-view 'RHEL7_Test' \
  --type=rpm \
  --name='Original Packages' \
  --inclusion=true \
  --original-packages=true

hammer content-view filter create \
  --organization "$ORG" \
  --content-view 'RHEL7_Test' \
  --type=rpm \
  --name='No Kernel' \
  --inclusion=false

hammer content-view filter rule create \
  --organization "$ORG" \
  --content-view 'RHEL7_Test' \
  --content-view-filter 'No Kernel' \
  --name 'kernel*'
~~~


### Publish your content views


At a date on or near the 1st Saturday of the month (7 Jan in our example), the content view will need to be published.
~~~
hammer content-view publish \
  --organization "$ORG" \
  --name RHEL7_Test \
  --description 'Initial Publishing'
~~~

and promote it to development.

~~~
hammer content-view version promote \
  --organization "$ORG" \
  --content-view RHEL7_Test \
  --to-lifecycle-environment Development
~~~

At this point, version 1.0 of the content view exists in Development (and also in Library)

 - | Jan | Feb | March
-------- | -------- | ------- | --------
Development |  **1.0**
QA |
Production |

### Registering systems to Development.

Now that version 1.0 of the **RHEL7_Base** content view exists in Development, we can begin to register systems. This can be done either via username/password
or via an activation key. This example will use an activation key as those are intended for automation reasons.

Firstly, create an activation key
~~~
hammer activation-key create --organization "$ORG" --description 'Basic RHEL7 Key for Registering to Dev' --content-view 'RHEL7_Base' --unlimited-hosts  --name ak-Reg_To_Dev --lifecycle-environment 'Development'
~~~

Get a list of which subscriptions are available
~~~
hammer --output json subscription list --organization "$ORG"
{
  "ID": 151,
  "UUID": "4028fc825d75492f015d75569c9f035b",
  "Name": "Red Hat Enterprise Linux Server, Premium (Physical or Virtual Nodes)",
  "Contract": [REDACTED],
  "Account": [REDACTED],
  "Support": "Premium",
  "Quantity": 7,
  "Consumed": 0,
  "End Date": "2017-10-06T03:59:59.000+0000",
  "Attached": 0
},
~~~

Attach a subscription to the key
~~~
hammer activation-key add-subscription --organization "$ORG" --name ak-Reg_To_Dev --subscription-id 151
~~~

Your subscription ID will differ based on which subscriptions you've imported into your Satellite.

**ON A CLIENT**, register it to the Satellite.
~~~
yum -y install http://satellite.example.com/pub/katello-ca-consumer-latest.noarch.rpm
subscription-manager register --org Example --activationkey ak-Reg_To_Dev
~~~

### Patching systems in Development

Now that the system is in development, you can update it using any supported methodology:

 - Remote Execution
 - katello-agent
 - yum.

## Quality Assurance

At a date on or near the second Saturday of the month (14 Jan in our Example), we'll need to promote version 1.0 of the **RHEL7_Base** content view to the **Quality Assurance** environment.  This can be done using the following:

~~~
hammer content-view version promote \
  --organization "$ORG" \
  --content-view RHEL7_Test \
  --to-lifecycle-environment 'Quality Assurance'
~~~

At this point, version 1.0 of the content view exists in QA (and also in Development and the Library)

 - | Jan | Feb | March
-------- | -------- | ------- | --------
Development |  **1.0**
QA | **1.0**
Production |


### Registering systems to QA.

Now that version 1.0 of the **RHEL7_Base** content view exists in QA, we can begin to register systems.  Again, we are using an activation key which is preferred to username/password

Firstly, create an activation key
~~~
hammer activation-key create --organization "$ORG" --description 'Basic RHEL7 Key for Registering to QA' --content-view 'RHEL7_Base' --unlimited-hosts  --name ak-Reg_To_QA --lifecycle-environment 'Quality Assurance'
~~~

Get a list of which subscriptions are available
~~~
hammer --output json subscription list --organization "$ORG"
{
  "ID": 151,
  "UUID": "4028fc825d75492f015d75569c9f035b",
  "Name": "Red Hat Enterprise Linux Server, Premium (Physical or Virtual Nodes)",
  "Contract": [REDACTED],
  "Account": [REDACTED],
  "Support": "Premium",
  "Quantity": 7,
  "Consumed": 0,
  "End Date": "2017-10-06T03:59:59.000+0000",
  "Attached": 0
},
~~~

Attach a subscription to the key
~~~
hammer activation-key add-subscription --organization "$ORG" --name ak-Reg_To_QA --subscription-id 151
~~~

Your subscription ID will differ based on which subscriptions you've imported into your Satellite.

**ON A CLIENT**, register it to the Satellite.
~~~
yum -y install http://satellite.example.com/pub/katello-ca-consumer-latest.noarch.rpm
subscription-manager register --org Example --activationkey ak-Reg_To_QA
~~~

### Patching systems in QA

Now that the system is in QA, you can update it using any supported methodology:

 - Remote Execution
 - katello-agent
 - yum.



## Production

At a date on or near the second 4th of the month (28 Jan in our Example), we'll need to promote version 1.0 of the **RHEL7_Base** content view to the **Production** environment.  This can be done using the following:

~~~
hammer content-view version promote \
  --organization "$ORG" \
  --content-view RHEL7_Test \
  --to-lifecycle-environment 'Production'
~~~

At this point, version 1.0 of the content view exists in Production (and also in QA, Development and the Library)

 - | Jan | Feb | March
-------- | -------- | ------- | --------
Development |  **1.0**
QA | **1.0**
Production | **1.0**


### Registering systems to Production.

Now that version 1.0 of the **RHEL7_Base** content view exists in Production, we can begin to register systems.  Again, we are using an activation key which is preferred to username/password

Firstly, create an activation key
~~~
hammer activation-key create --organization "$ORG" --description 'Basic RHEL7 Key for Registering to Production' --content-view 'RHEL7_Base' --unlimited-hosts  --name ak-Reg_To_Production --lifecycle-environment 'Production'
~~~

Get a list of which subscriptions are available
~~~
hammer --output json subscription list --organization "$ORG"
{
  "ID": 151,
  "UUID": "4028fc825d75492f015d75569c9f035b",
  "Name": "Red Hat Enterprise Linux Server, Premium (Physical or Virtual Nodes)",
  "Contract": [REDACTED],
  "Account": [REDACTED],
  "Support": "Premium",
  "Quantity": 7,
  "Consumed": 0,
  "End Date": "2017-10-06T03:59:59.000+0000",
  "Attached": 0
},
~~~

Attach a subscription to the key
~~~
hammer activation-key add-subscription --organization "$ORG" --name ak-Reg_To_Production --subscription-id 151
~~~

Your subscription ID will differ based on which subscriptions you've imported into your Satellite.

**ON A CLIENT**, register it to the Satellite.
~~~
yum -y install http://satellite.example.com/pub/katello-ca-consumer-latest.noarch.rpm
subscription-manager register --org Example --activationkey ak-Reg_To_Production
~~~

### Patching systems in Production

Now that the system is in Production, you can update it using any supported methodology:

 - Remote Execution
 - katello-agent
 - yum.


**Success** - You've completed one patching cycle. Let's summarize what was completed.

- We've downloaded repositories to create a core build.
- We defined a content view (a point in time snapshot) that reflects our core build.
- We used content view filters to further limit which content is available, limiting it to a date of our choosing.
- We defined our lifecycle environments (Dev, QA & Prod) as a linear promotion path.
- We published version 1.0 of the content & promoted it to the Development environment.
- We created an activation key so that systems can register to development.
- We repeated the promotion of version 1.0 of the **RHEL7_Base** content view to the QA & Production environment and likewise created activation keys so that clients may register.


## Next Steps

Now that we've defined a basic workflow and policy to handle our *standard* changes, there are two other areas left unaddressed:

- How to handle the next iteration of the patch cycle.
- Handling the *normal* and *emergency* changes


### The next iteration of our monthly patching cadence

In our next iteration (in February based on our calendar below), we'll need to update the **RHEL7_Base** content view to reflect content up to 1 Feb and deploy it the lifecycle environments as per the policy. (Dev on the first Saturday. QA on the second. Production on the fourth)

~~~
January                February               March       
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
 1  2  3  4  5  6  7             1  2  3  4             1  2  3  4   
 8  9 10 11 12 13 14    5  6  7  8  9 10 11    5  6  7  8  9 10 11   
15 16 17 18 19 20 21   12 13 14 15 16 17 18   12 13 14 15 16 17 18   
22 23 24 25 26 27 28   19 20 21 22 23 24 25   19 20 21 22 23 24 25   
29 30 31               26 27 28               26 27 28 29 30 31
~~~


Let's go back to our **RHEL7_Base** content view and update it to reflect our new date (01-Feb-2017)

Firstly, we have to get the filter rule ID, as that is needed to update our content view.
~~~
hammer --output json content-view filter rule  list \
  --organization "$ORG"  \
  --content-view-filter Filter_Date_Based_Errata \
  --content-view RHEL7_Test
[
  {
    "Rule ID": 3,
    "Filter ID": 3,
    "Name": null,
    "Version": null,
    "Minimum Version": null,
    "Maximum Version": null,
    "Errata ID": null,
    "Start Date": null,
    "End Date": "2017-01-01"
  }
]
~~~

And now let's update our content with our new date (01 Feb)

~~~
hammer content-view filter rule update   \
  --organization "$ORG"   \
  --content-view 'RHEL7_Test'   \
  --content-view-filter 'Filter_Date_Based_Errata'   \
  --end-date 2017-02-01   \
  --types enhancement,bugfix,security \
  --id 3
~~~

(... FILE A BZ on the above. I should be able to create a content view filter rule by name and then use it later without having to query for its ID. The above is *suboptimal* )


**OPTIONAL** Check your work
~~~
hammer --output json content-view filter rule  list \
  --organization "$ORG"  \
  --content-view-filter Filter_Date_Based_Errata \
  --content-view RHEL7_Test
[
  {
    "Rule ID": 3,
    "Filter ID": 3,
    "Name": null,
    "Version": null,
    "Minimum Version": null,
    "Maximum Version": null,
    "Errata ID": null,
    "Start Date": null,
    "End Date": "2017-02-01"
  }
]
~~~


Next let's republish our content view.

~~~
hammer content-view publish  \
  --organization "$ORG" \
  --name RHEL7_Test \
  --description 'Update for 01Feb2017 patching'
~~~


Once that content view is finished publishing, we can promote it to Development

~~~
hammer content-view version promote  \
  --organization "$ORG" \
  --content-view RHEL7_Test   \
  --to-lifecycle-environment Development \
  --version 2.0
~~~

**NOTE**: as there are multiple versions of content views available, you now how have to specify a version.   

Our content view / lifecycle map now looks like.

- | Jan | Feb | March
-------- | -------- | ------- | --------
Development |  **1.0** | **2.0**
QA | **1.0**
Production | **1.0**



## Patching systems in Development.

Similar to the previous month, you can use any supported means to deploy errata to Development systems. Also, no new activation keys are required. Activation keys are only used for registration, and do not need to be regenerated each iteration of the patch cycle.


### QA & Production patching for Feb

Using the process above, on the second Saturday of Feb (11 Feb) version **2.0** needs to be promoted to QA. And subsequently, on the 4th Saturday (25 Feb), version **2.0** needs to be promoted to Production.


## Handling Normal & Emergency Changes.

The process above allows us to handling our *standard* changes (preapproved, low risk changes). However, we still need to handle these parts of the patch policy

* Quarterly, they wish to deploy kernel errata to coincide with a scheduled outage window
* They want the ability to release arbitrary errata as needed to address high priority bug or security issues.

Incremental Updates, a part of [Satellite 6.1 Errata Management](https://access.redhat.com/articles/1446123) provides this capability.

In our example, we have version 2.0 in development and in QE. Let's say on 22 Feb, prior to the next scheduled patching cutoff (01 Mar) an errata (such as RHSA-2017:0294) was released, which the organization has deemed as high priority enough to deploy **prior** to the next scheduled window.

You _could_ modify the **RHEL7_Base** content view's date filter to a new date which includes the errata in question. However, you not only get that errata, but **ALL** errata released between 01 Feb & 22 Feb, which is usually not desired.  This begs the question:


~~~
How do I release a singular errata to an existing content view in a minimally invasive fashion.
~~~

Firstly, let's get a list of our content view versions

~~~
hammer --output json content-view version list --organization "$ORG"
[
  {
    "ID": 11,
    "Name": "RHEL7_Test 2.0",
    "Version": "2.0",
    "Lifecycle Environments": [
      "Library",
      "Development",
      "Quality Assurance"
    ]
  },
  {
    "ID": 10,
    "Name": "RHEL7_Test 1.0",
    "Version": "1.0",
    "Lifecycle Environments": [
      "Production"
    ]
  },
  {
    "ID": 1,
    "Name": "Default Organization View 1.0",
    "Version": "1.0",
    "Lifecycle Environments": [
      "Library"
    ]
  }
]


~~~

Knowing the ID of our Development version of the content view, we can roll out the errata to the Development & QA content views
~~~
hammer content-view version incremental-update \
  --organization "$ORG" \
  --content-view-version-id 11 \
  --errata-ids "RHSA-2017:0294" \
  --lifecycle-environments Library,Development,"Quality Assurance" \
  --resolve-dependencies yes
Content View: RHEL7_Test version 2.1
Added Content:
  Errata:
        RHSA-2017:0294
        RHSA-2017:0294

~~~

(... Again, this is *suboptimal* . I should be able to update a CV incrementally if I provide its name, version & environment. file BZ)

Afterwards we can see that the content view has been updated with a minor revision (now it is version **2.1** and replaces the old version **2.0** in the environments).


- | Jan | Feb | March
-------- | -------- | ------- | --------
Development |  **1.0** | **2.1**
QA | **1.0** | **2.1**
Production | **1.0**

From here, we can install the errata using any supported mechanism.


### Extending this guide to include a Quarterly patching example.

This guide can be easily adapted to a quarterly, semi-annual or custom schedule.  The only changes required are to decide on which day is the cutoff for errata and when is errata released.

As an example, if Example Corp wanted to switch to a quarterly model, their new policy could be (as an Example)

* Development shall be patched on the 1st day of the first month of the quarter.
* QA shall be patched on the 1st day of the second month of the quarter.
* Production shall be patched on the 1st day of the third month of the quarter.
* They want to deploy all only security errata (RHSA) to their systems, with the exception of kernel errata.
* Quarterly, they wish to deploy kernel errata to coincide with a scheduled outage window.
* They want the ability to release arbitrary errata as needed to address high priority bug or security issues.

Now let's look at the changes required:

- Instead of creating & updating the *Filter_Date_Based_Errata* content view filter monthly, it will be updated on the first day of the quarter (01 Jan, 01 Apr, 01 Jul & 01 Oct)
- Instead of deploying errata on the 1st, 2nd & 4th Saturday for Development, Quality Assurance & Production respectively, errata is deployed on the 1st day of the month.
