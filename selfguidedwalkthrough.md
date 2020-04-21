# Amazon AppStream 2.0 Student Lab EUC Self-Paced Setup Guide
### When utilizing remote end user compute solutions, users need to have a consistent application experience regardless of their location, as well as secure access to enterprise resources through a single sign-on portal.  AppStream 2.0 provides the resources necessary to build, deploy, and stream secure user-installed applications at scale.  All compute, storage, and configurations remain inside the private account environment within AWS, enabling robust security by default.

### This walkthrough will guide you through the technical implementation steps required for deploying an AppStream 2.0 Stack and Fleet, as well as how to build your custom application image for deployment to said fleet.

## Getting Started
The core components of an AppStream 2.0 deployment consist of the fleet of compute instances that host the Windows OS and application environments, the user-created "golden" images, and finally the stack, which brings together the aforementioned components and presents them to users via a single sign-on access portal.

AppStream 2.0 stacks can be joined to Microsoft Active Directory domains and can utilize VPN and direct-connect technologies to reach on-premises resources, such as storage, intranet sites, and printers.

Regardless of Active Directory domain membership, however, AppStream 2.0 can also federate logins using SAML 2.0 in order to integrate with existing identity providers (IdPs).  While each IdP will have a different overall methodology for integrating into AppStream 2.0, this self-paced guide will provide the necessary common steps for getting this process started, as well as referencing the specific step-by-step instructions for various IdPs.

## Pre-requisites
This courtesy on-board material set includes a Microsoft Word document titled **Getting Started Prerequisites for Amazon AppStream 2**.  This document contains the checklists and processes which should be followed for determining important items pre-deployment, such as:

  * Number of concurrent client sessions
  * Instance type for streaming
  * Whether or not IdP integration will be required

### AWS account
You will need to have created an AWS account for yourself, or have one provided to you for the purpose of the deployment where you have been provided IAM Administrator permissions.  It is recommended to use a dedicated AWS account for the EUC standard play, if possible.  It is also recommended that you create an IAM user with administrator permissions inside the account for creating and managing resources, rather than using the root user login for any deployment or management within that account.  [For more helpful AWS Account security best practices, click here.](https://aws.amazon.com/blogs/security/getting-started-follow-security-best-practices-as-you-configure-your-aws-resources/)

### Network requirements
Your users will need outbound TCP ports open for the NICE DCV protocol:

  * TCP 443
  * TCP 8443

In order to ensure your users have a consistent experience, follow [the peak bandwidth and latency requirements outlined in the documentation here.](https://docs.aws.amazon.com/appstream2/latest/developerguide/bandwidth-recommendations-user-connections.html)

### Software installers
You will need Internet-accessible installers for your software package(s) which need to be installed for streaming to your users.  If there are no locations on the public Internet you can source these installers, you can also place the installer(s) into the S3 bucket created by this tutorial and access the bucket from the Image Builder.

### User authentication via existing identity provider (IdP)
If you plan to integrate your user logins with a SAML 2.0 identiy provider, either you or someone involved with the process must have sufficient permissions (or personnel) to administrate the existing identity provider.

The trust relationship must be set up between the existing IdP and AWS.

There must be an IdP federation metadata document generated for loading into AWS.

## AWS account and accessing the AWS management console
  * Open a new browser tab
  * Sign in to your AWS account by going to [https://aws.amazon.com](https://aws.amazon.com)
  * Click on **My Account** and then select **AWS Management Console**

## Deploy CloudFormation template
Once signed into the AWS management console:
  * Navigate to **Services** and select **CloudFormation** from the list of services
  * You will be presented with the AWS CloudFormation Create Stack console
  * Click on the **Create Stack** button

An AWS CloudFormation Stack is based on a template that is a JSON or YAML file that contains configuration information (infrastructure-as-code) about the AWS resources you want to include in the stack.

Select the following options:
  * Prerequisite – Prepare template = select **Template is ready** option
  * Specify template = select **Amazon S3 URL** option
  * Amazon S3 URL text box:
    * Copy and paste the below Amazon S3 URL:
        * <URL to S3-hosted YAML CFn template goes here>

Click on **Next** to continue.

You will now be presented with a page where you will specify details of the stack you are going to create.  AWS CloudFormation will automatically deploy resources in your account based upon the template file specified.  In order to create an implementation specific to your environment, the template contains parameters which you need to provide, akin to variables in a script.  

Default values are filled in for you, but most of them will need to be adjusted for your specific deployment needs:

  * **Stack Name** = Keep the stack name unique, particularly if you are using a shared AWS account. Good examples include identifying information about the specific deployment, such as "AppstreamYourOrganizationYourUserbase01"
  * **DesiredFleetInstances** = This is the number of concurrent streaming sessions that your fleet can support in a steady state.  This parameter may be adjusted later either manually, or automatically using [Application Automatic Scaling](https://docs.aws.amazon.com/appstream2/latest/developerguide/autoscaling.html).  For now, set it to a "best guess" estimate of your application's required user concurrency.
  * **EnvironmentName** = This is a naming prefix for all of the resources deployed by your CloudFormation template. This should be unique within the AWS account it is deployed in.
  * **FleetImageName** = This is the name of an application image which has been created in advance using an Image Builder instance. It will contain the application(s) your users will be utilizing via the streaming service.  For now, you can use a generic Windows image as a placeholder, such as what is specified in the default.  You can find a list of available base images here:
    * [https://docs.aws.amazon.com/appstream2/latest/developerguide/base-image-version-history.html](https://docs.aws.amazon.com/appstream2/latest/developerguide/base-image-version-history.html)

  * **FleetInstanceType** = This is the type of compute instance that will be used in the AppStream 2.0 deployment. You can select the type of instance from the drop-down based upon your application's recommended minimum system requirements. To reference the various instance types available:
    * [AppStream 2.0 Instance Families](https://docs.aws.amazon.com/appstream2/latest/developerguide/instance-types.html)
    * [AppStream 2.0 Instance Specifications and Pricing](https://aws.amazon.com/appstream2/pricing/)

  * **FleetTypeSetting** = Initially, it is recommended to leave this setting "ON_DEMAND."  Using "ALWAYS_ON" can decrease the response times of initial connections for users, however it maintains the fleet in an always-running state to do so.  Choosing which setting is optimal will depend upon your user activity patterns.
  * **SessionRedirectURL** = The website your users will be directed to once they log out of their AppStream 2.0 session. This can be any publicly-accessible website, or if your users are exclusively connecting from a private network it can be a local intranet site.
  * **SoftwareS3BucketName** = Set this to a globally-unique DNS name for an S3 bucket. The bucket this template creates will be used for storing software to install via the Image Builder stage.
  * **VpcCIDR** = The IPv4 CIDR /16 private IP range which will comprise the virtual private cloud (VPC) in use by this AppStream 2.0 deployment. Make sure this range does not overlap with your on-premesis network range(s), and that it does not use the range 198.19.0.0/16.
  * **PrivateSubnet1CIDR, PrivateSubnet2CIDR, PublicSubnet1CIDR, PublicSubnet2CIDR** = Use these to slice out portions of the VPC CIDR range for use by the various parts of your AppStream 2.0 deployment.  While the "public" subnets can be very small since they will only need to contain NAT Gateways, each "private" subnet should be sized with sufficient space to handle _all_ concurrent client streaming connections.  In this way, there is sufficient IP space to make the deployment fault-tolerant.
  * **TestUserEmail** = The email address which will be used to register the first user in your user pool. This account will be for testing and general purpose use.

Once you have filled in the appropriate parameters, click on **Next** in the lower-right corner to continue.

On the **Configure Stack Options** page, add any appropriate tags for the stack.  While this is not required, it can be very beneficial for tracking utilization across accounts for billing and administrative purposes.  For guidance on resource tagging within AWS, [reference this page.](https://aws.amazon.com/answers/account-management/aws-tagging-strategies/)

The remaining options for this stack can be left on their defaults at this time.

Click **Next** in the lower-right corner.

You will be presented with a summary of the Stack configuration for review.
  * Scroll to the bottom of the page to the “Capabilities” section.
  * Select _BOTH_ of the acknowledge checkboxes, if they appear. This will allow the template to create appropriate permissions within your account.
  * Click on the **Create Stack** button to execute the CloudFormation stack configuration.

You will be redirected to the AWS CloudFormation main page and you will see your Stack listed in the page.

Click on “View nested” to un-select the option.

The Stack will have the status of **CREATE IN PROGRESS.**

>**Note:** The AWS CloudFormation template will take about 15-20 minutes to complete. The AWS CloudFormation template is now provisioning resources such as the VPC, subnets, Internet Gateways, NAT gateways, routing tables, S3 buckets, and AppStream 2.0 fleet, stack, and IAM role.
Once complete, the stack will show in the list with a status of "UPDATE_COMPLETE" with a green check mark.  This indicates that your resources have all been deployed, and you can move on to the next step.

## Building your application's "golden image" via the image builder
#### Now that you have the basic components deployed, it's time to configure your application image so it can be associated with your stack, enabling users to run the application via a streaming session.  You will do this by creating and connecting to an Image Builder instance, and installing your application(s).

### Download software installer file(s) and perform the installation.

First, ensure you have either uploaded your installer files to the S3 bucket created by the CloudFormation template, or ensure you have the ability to download the installers over the Internet from the image builder once you're connected.

Open the AppStream 2.0 console at [https://console.aws.amazon.com/appstream2](https://console.aws.amazon.com/appstream2)

In the navigation pane, choose **Images** then **Image Builder.**

Click on **Launch Image Builder** and choose an appropriate base Windows image for your use case.  Then, scroll to the bottom of the screen and click **Next**.

Give your Image Builder a meaningful name that relates to your application(s). The **Name** field cannot contain spaces, so for a more descriptive name, you can fill in the **Display Name** field as well. Add any appropriate tags to your Image Builder on this screen.

Choose an instance type that meets the CPU and RAM needs of your application for installation under the **Instance Type** section.  

On the **Network Access** screen, select the VPC based on the **Environment Name**, a *private* subnet in that VPC, and the no-ingress-sg security group.  Then click **Review**.  Make sure all settings look correct, then click **Launch**.

You will be taken back to the **Image Builder** list, and should see your newly created entry in a "Pending" state.  Click the refresh button periodically to check until it's shown to be in a running state.

Once your Image Builder is running, select it from the list and click **Connect**.  This will open a new browser tab displaying options for logging into the image builder. Choose **Local User**, **Administrator**:
  * Note: If a new browser tab does not open, configure your browser to allow pop-ups from https://console.aws.amazon.com/

After a few moments, you will be connected to the image builder instance with administrator rights, and should see a Windows desktop.

From your Image Builder session, download the software installer(s) from a verified location, and run their respective installation process(es) as they would normally be done.  You could be downloading them from the public Internet, or from an S3 bucket, depending upon your specific source location.

This step will vary depending upon the application itself that you're deploying via AppStream 2.0, but you can find a series of walkthroughs for commonly-deployed apps on the platform [via this link](https://aws.amazon.com/appstream2/getting-started/) under **Application Deployment Guides**.

> **Important:** Ensure that your application does not install into a user's home directory.  It will need to be an installation accessible to all users in order for it to be available for streaming.

If an application requires the Windows operating system to restart, let it do so. Before the operating system restarts, you are disconnected from your image builder. After the restart is complete, connect to the image builder again, then finish installing the application.

## Use image assistant to create the AppStream 2.0 image from the image builder
The following steps will guide you through the process of generically deploying a new application for streaming via your AppStream 2.0 fleet.  In addition to the guide below, AWS has prescriptive app-specific guidance for deploying many commonly-distributed applications via AppStream 2.0 such as:

* [SAP GUI](https://d1.awsstatic.com/product-marketing/AppStream2.0/Amazon%20AppStream%202.0%20SAP%20GUI%20Deployment%20Guide.pdf)
* [Solidworks](https://d1.awsstatic.com/product-marketing/AppStream2.0/Amazon%20AppStream%202.0%20SOLIDWORKS%20Deployment%20Guide.pdf)
* [Siemens NX](https://d1.awsstatic.com/product-marketing/AppStream2.0/AppStream%202.0%20Siemens%20NX%20Deployment%20Guide.pdf)
* [MATLAB](https://d1.awsstatic.com/product-marketing/AppStream2.0/AppStream2-MatLab-Deployment-Guide.pdf)
* [Topaz on AWS](http://frontline.compuware.com/doc/KB/KBTAWS/readme.html)
* [ArcGIS Pro](https://d1.awsstatic.com/product-marketing/AppStream2.0/AppStream2-ESRI-ArcGISPro-Deployment-Guide-Feb.pdf)
* [AutoCAD](https://d1.awsstatic.com/end-user-computing/AppStream2-AutoCAD-Deployment-Guide-new.pdf)

### Build the application catalog
#### In this step, create an AppStream 2.0 application catalog by specifying applications (.exe), batch scripts (.bat), and application shortcuts (.lnk) for your image. For each application that you plan to stream, you can specify the name, display name, executable file to launch, and icon to display. If you choose an application shortcut, these values are prepopulated for you.
> **Important:** In order to complete this step, you must be logged into the Image Builder as an account that has local Administrator rights.

  * From the desktop of the Image Builder, double-click on the **Image Assistant** icon.  
  * In **1. Add apps**, choose **+Add app**, and navigate to the location of the application, script, or shortcut to add. Then choose **Open**.
  * In the **App Launch Settings** dialog box, keep or change the default settings for **Name, Display Name,** and **Icon Path**. Optionally, you can specify launch parameters (additional arguments passed to the application when it is launched) and a working directory for the application. When you're done, choose **Save**.
  The **Display Name** and **Icon Path** settings determine how your application name and icon appearin the application catalog. The catalog displays to users when they sign in to an AppStream 2.0streaming session.
  * Repeat steps 2 and 3 for each application in Image Assistant and confirm that the applications appear on the **Add Apps** tab. When you're done, choose **Next** to continue using Image Assistant to create your image.

> **Important:** In order to complete this next step, you must be logged into the image builder with the local **Template User** account or a domain user account that does _not_ have local administrator rights.

### Create default application and windows settings
#### In this step, you create default application and Windows settings for your AppStream 2.0 users. Doing this enables your users to get started with applications quickly during their AppStream 2.0 streaming sessions, without the need to create or configure these settings themselves.
  * From the desktop of the Image Builder, double-click on the **Image Assistant** icon.  
  * In Image Assistant, in **2. Configure Apps**, choose **Switch user**. This disconnects you from the current session and displays the login menu.
  * On the **Local User** tab, choose **Template User**. This account enables you to create your default application and Windows settings.
  * Once again, from the image builder desktop, open **Image Assistant**, which displays the applications that you added when you created the application catalog.
  * Choose the application for which you want to create default application settings.
  * After the application opens, create these settings as needed.
  * When you're done, close the application, and return to Image Assistant.
  * If you specified more than one application in Image Assistant, repeat steps 5 through 7 for each application as needed.
  * If you want default Windows settings, create them now. When you're done, return to Image Assistant.
  * Choose **Switch user** and log in with the same account that you used to create the application catalog (an account that has local administrator permissions).
  * In Image Assistant, in **2. Configure Apps**, choose **Save settings.**  The **Choose which settings to copy** list displays any user account that currently has settings saved on the image builder.
  * When you're done, choose **Next** to continue creating your image.

> **Important:** To complete this step, you must log in to the image builder with the Test User account or a domain user account that does _not_ have local administrator permissions.

### Test your applications in the catalog
#### In this step, verify that the applications you've added open correctly and perform as expected. To do so, start a new Windows session as a user who has the same permissions as your users.
  * In Image Assistant, in **3. Test**, choose **Switch user**.  
> **Note:** If your image builder is new and no users have settings on the image builder, the list does not display any users.

  * Choose Test User. This account enables you to test your applications by using the same policies and permissions as your users.
  * From the image builder desktop, open Image Assistant, which displays the applications that you specified when you created the application catalog.
  * Choose the application that you want to test, to confirm that it opens correctly and that any default application settings you created are applied.
  * After the application opens, test it as needed. When you're done, close the application and return to Image Assistant.
  * If you specified more than one application in Image Assistant, repeat steps 3 and 4 to test each application as needed.
  * When you're done, choose **Switch user**, then on the Local User tab, choose **Administrator**.
  * Choose **Next** to continue creating your image.

### Optimize applications
#### In this step, Image Assistant opens your applications one after another, identifies their launch dependencies, and performs optimizations to ensure that applications launch quickly. These are required steps that are performed on all applications in the list.
**To optimize your applications:**
  * In the Image Assistant, under **4. Optimize**, choose **Launch**
  * AppStream 2.0 automatically launches the first application in your list. When the application completely starts, provide any required input to perform the first run experience for the application. For example, a web browser may prompt you to import settings before it is completely up and running.
  * After you complete the first run experience and verify that the application performs as expected, choose **Continue.** If you added more than one application to your image, each application opens automatically. Repeat this step for each application as needed, leaving all applications running.
  * When you're done, the next tab in Image Assistant, **5. Configure Image**, automatically displays.

### Finish creating your image
#### In this step, choose an image name and finish creating your image.
**To create your image:**
  * Type a unique image name, and an optional image display name and description. The image name cannot begin with "Amazon," "AWS," or "AppStream." Note this image name down for later, as you will be using it to update your stack.
  You can also add one or more tags to the image. To do so, choose **Add Tag**, and type the key and value for the tag. To add more tags, repeat this step. For more information, see [Tagging Your Amazon AppStream 2.0 Resources.](https://docs.aws.amazon.com/appstream2/latest/developerguide/tagging-basic.html) When you're done, choose **Next.**

> **Note:** If you choose a base image that is published by AWS on or after December 7, 2017, the option **Always use the latest agent version appears**, selected by default. We recommend that you leave this option selected so that streaming instances that are launched from the image always use the latest version of the agent. If you disable this option, you cannot enable it after you finish creating the image. For information about the latest release of the AppStream 2.0 agent, see [AppStream 2.0 Agent Version History.](https://docs.aws.amazon.com/appstream2/latest/developerguide/agent-software-versions.html)

  * In **6. Review**, verify the image details. To make changes, choose **Previous** to navigate to the appropriate Image Assistant tab, make your changes, and then proceed through the steps in Image Assistant as needed.
  * After you finish reviewing the image details, choose **Disconnect and Create Image.**
  * The remote session disconnects within a few moments. When the **Lost Connectivity** message appears, close the browser tab. While the image is created, the image builder status appears as **Snapshotting**. You cannot connect to the image builder until this process finishes.
  * Return to the console and navigate to **Images, Image Registry**. Verify that your new image appears in the list.
  While your image is being created, the image status in the image registry of the console appears as **Pending** and you cannot connect to it.
  * Choose the **Refresh** icon periodically to update the status. After your image is created, the image status changes to **Available** and the image builder is automatically stopped.  To continue creating images, start the image builder and connect to it from the console, or create a new image builder.

> **Note:** After you create an image, you can't change it. To add other applications, update existing applications, or change image settings, you must start and reconnect to the image builder that you used to create the image, or, if you deleted that image builder, launch a new image builder that is based on your image. Then, make your changes and create a new image.

### Enable user home folders
#### In order for your users to have a persistent storage available to their application, you can utilize native home folders via an S3 bucket, Google Drive for G Suite, or OneDrive for Business.  To get your users online as quickly as possible with storage for them to consume, we're going to walk through setting up native Home Folders for your AppStream 2.0 Stack.

Navigate to the AppStream 2.0 console, select **Stacks** then choose the stack we have created using the CloudFormation template.  

Navigate to the **Storage** tab below the list of stacks, and turn on the **Enable Home Folders** check box.  

You will see the name of an S3 bucket that has been created on your behalf by AppStream 2.0 for use in this purpose.

When your users log in, they will now see two locations to save files within their applications.  One is temporary storage that does not persist between sessions, and the other is a home folder which will retain data saved to it between sessions.

### Clean-up
#### Finally, stop your running image builders to free up resources and avoid unintended charges to your account. We recommend stopping any unused, running image builders. For more information, see [AppStream 2.0 Pricing.](https://aws.amazon.com/appstream2/pricing/)

**To stop running an image builder**

In the navigation pane, choose **Images**, **Image Builders**, and select the running image builder instance.

Choose **Actions**, **Stop**.

### Update your stack and fleet to use the newly-created image
#### Now that you have created an application image for your users, it needs to be associated with the stack.  By default, the CloudFormation template associates your stack with a generic Windows image.  To change this in your deployment, you just need to alter the **FleetImageName** parameter of your CloudFormation stack.

Navigate to **Services**, then **CloudFormation**.

Open the CloudFormation Stack you created in the initial step of this guide, and click on **Update**.

Choose **Use current template** on the **Specify template** (Step 1) step.  We will not be changing the template itself, only one of the parameters it uses to create and configure your resources.

On Step 2, **Specify stack details** page scroll down to **FleetImageName** and replace the default setting with the name of the custom image you created earlier in this process under the **Finish Creating your Image** step.  Use a "copy/paste" method to fill it in, in order to avoid typography errors.

Click **Next**, then click **Next** again, and finally on the Review page, scroll to the bottom and click **Update Stack**.

Back on the **CloudFormation** **Stacks** page, you should see the stack in an **UPDATING** state.  Use the refresh button next to **Stacks** to refresh the list until it shows **UPDATE_COMPLETE**.  Your AppStream 2.0stack is now updated to use your new custom "golden image" for your users.

## Logging in, testing the applications, and customizing branding
#### Your applications are now ready for use by users in your user pool.  The CloudFormation template has created your first user named **Test User** for you, and you should have now received the activation email with the temporary password.

First, go to the AppStream 2.0 console and navigate to your **Fleet**, then click **Actions**, **Start**.  The fleet will start up, and after a few minutes you should see it in a **Running** state.

Open the activation email that contains your temporary password, and click the **Login Page** link to log in.

> **Note:** If you are unable to find the activation email, or if you entered an incorrect address in your CloudFormation template, you can change the email address by using the same method you used above to change the Fleet image parameter, except change the **TestUserEmail** parameter instead.

Enter your email address and temporary password on the login page, and then set a new password when prompted.

After logging in, you should see your application(s) available for launch from the page.  You will also see a link to download the native AppStream 2.0 client which can enable multi-screen capabilities and connect without the use of a browser.

Click on the Application you want to load.  Depending upon whether you have configured the fleet for ON_DEMAND, there may be a short delay for the session to spin up (around 2 minutes.)  If you're using ALWAYS_ON, the session will spin up almost immediately.  You will see a notification when the application is ready.

Once loaded into the application, your users will see two locations where they can save their files, with "Home Folder" being the persistent storage which remains between sessions.

#### In order to give your users a more consistent login experience, you can also customize the AppStream 2.0login screen's appearance.

Navigate to **Stacks** in the AppStream 2.0 console, then choose your stack and select the **Branding** tab.

Change your theme selection from **AppStream 2.0 (default)** to **Custom**.

You will see an area appear below which will allow you to upload your logo, add appropriate company URLs, change the color scheme, and add the website browser title and icon.  Once you've made your desired changes, use the **Save** button in the lower-right.

You can log in as your test user again after refreshing the browser, and you should now see your chosen branding on the AppStream 2.0 page.

## Adding production users to your stack with User Pools
#### AppStream 2.0 User Pools can accommodate up to 50 users by default. This quota can be increased by opening a support ticket, however for deployments involving 100 users or more, it is highly recommended to implement SAML 2.0 authentication to an external identity provider (IdP) for user management.

> **Note:** For setting up SAML 2.0 federation, skip the step outlined below and follow the directions [in this guide](https://docs.aws.amazon.com/appstream2/latest/developerguide/external-identity-providers-setting-up-saml.html) for the specific identity provider you are using.

Your first user was created for you by the CloudFormation template, but now you will need to add additional users to your pool so they can stream your application(s).  This is done in the AppStream 2.0 console under **User Pool**.  

Click **Create User**, enter the person's email address and name, then click the blue **Create User** button.

You can use this process to add as many users as you need to the User Pool, and all of them will receive welcome emails which will give them their temporary passwords, their link to login, and instructions.

Once the users are created, select the checkboxes next to each of the new users you have added, and click the **Actions** button.  Then choose **Assign Stack**.

Choose the stack you've created in the previous steps from the pull-down menu, and assign them using the **Assign Stack** button.  The users will now see the application(s) you've configured into the stack's catalog when they log in.

## Congratulations!  
#### You have now deployed a custom application for use by the users in your User Pool, and they can now access it by streaming from anywhere with an Internet connection.  You can also share the login link provided in the welcome email via internal channels.
