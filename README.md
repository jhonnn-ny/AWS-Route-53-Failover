# AWS-Route-53-Failover
Amazon Route 53 Failover Routing
# **Amazon Route 53 Failover Routing**

## **Lab overview**

In this activity, you configure failover routing for a simple web application.

The activity environment starts with two Amazon Elastic Compute Cloud (Amazon EC2) instances that have already been created. Each of the instances has the full LAMP stack installed and the café website deployed and running. The EC2 instances are deployed in different Availability Zones. For example, if the web servers are running in the us-west-2 Region, then one of the web servers runs in the us-west-2a Availability Zone and the other one runs in the us-west-2b Availability Zone.

You will configure your domain such that, if the website in the primary Availability Zone becomes unavailable, Amazon Route 53 will automatically fail over application traffic to the instance in the secondary Availability Zone.

When you are finished, your environment will look like the following architecture:

The architecture diagram shows the final state of the infrastructure that you build. Route 53 records store the IP address of the EC2 instance in each Availability Zone. User requests are normally sent to the IP address corresponding to Café Instance1 in Availability Zone 1. If Café Instance1 is unavailable, requests are routed to Café Instance2 in Availability Zone 2 based on the configuration in the Route 53 records. When Café Instance1 becomes unavailable, a Route 53 health check alarm is invoked, and an email alert is sent to the email address provided.

![1](https://github.com/user-attachments/assets/b5e941ba-67f6-44c2-8067-09cd334deb75)


## **Objectives**

After completing this activity, you should be able to do the following:

- Configure a Route 53 health check that sends emails when the health of an HTTP endpoint becomes unhealthy.
- Configure failover routing in Route 53.

## **Duration**

This activity requires approximately **45 minutes** to complete.

## **Task 1: Confirming the café websites**

In this task, you analyze the resources that AWS CloudFormation has automatically created for you.

1. At the top of this page, choose **Details**. For **AWS**, choose **Show**. A **Credentials** panel opens.
2. Copy the values for the following parameters, and paste them into a text editor to use later.
    - CafeInstance1IPAddress
    - PrimaryWebSiteURL
    - SecondaryWebsiteURL
    - CafeInstance2IPAddress
3. Choose **X** to close the **Credentials** panel.
4. Navigate to the browser tab with the **AWS Management Console**. In the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.
5. In the left navigation pane, in the **Instances** section, choose **Instances**.
    
    Two EC2 instances have already been created for you. **CafeInstance1** is running in Cafe Public Subnet 1 (us-west-2a), and **CafeInstance2** is running in Cafe Public Subnet 2 (us-west-2b).
    
    The URLs that you copied earlier correspond to the café application running on each instance.
    
    Although both EC2 instances have the same configuration and application installed, one instance is a primary instance.
    
6. Open a new browser tab, and paste the value for **PrimaryWebSiteURL**.
    - The café application webpage should open.
    - Along with other information about the café, notice the **Server Information** that is displayed. It shows information about the EC2 instance and the Availability Zone where it is running.
7. Open another browser tab, and paste the value for **SecondaryWebsiteURL**. Confirm that the second EC2 instance has similar configurations as the first instance.
    
    These configurations confirm that the café application is running on both instances.
    
8. On one of the websites, choose **Menu**.
9. Choose any item on the menu, and choose **Submit Order**.
    
    The **Order Confirmation** page reflects the time that the order was placed in the time zone where the web server is running.
    
    You have now confirmed that two instances are running the café application. Each application is running in a different Availability Zone to provide high availability.
    

## **Task 2: Configuring a Route 53 health check**

The first step to configure failover is to create a health check for your primary website.

1. In the AWS Management Console, from the **Services** menu, enter and choose `Route 53` to open the **Route 53 Management Console**.

You can safely ignore any error messages displayed because of AWS Identity and Access Management (IAM) restrictions placed on lab accounts.

1. In the left navigation pane, choose **Health checks**.
2. Choose **Create health check**, and configure the following options. Leave the default values for all other fields.
    - **Name:** Enter `Primary-Website-Health`
    - **What to monitor:** Choose **Endpoint**.
    - **Specify endpoint by:** Choose **IP address**.
    - **IP address:** Paste in the **Public IPv4 address(and the path)** of **CafeInstance1**
    ![2](https://github.com/user-attachments/assets/5463b72c-c5ac-46a4-abfa-727c3c7445c9)

   
    
3. Expand  **Advanced configuration**, and configure the following options. Leave the default values for all other fields.
    - **Request interval:** Choose **Fast (10 seconds)**.
    - **Failure threshold:** Enter `2`
    
    These options make your health check respond faster.
    
4. Choose **Next**.
5. For **Get notified when health check fails**, configure the following options:
    - **Create alarm:** Choose **Yes**.
    - **Send notification to:** Choose **New SNS topic**.
    - **Topic name:** Enter `Primary-Website-Health`
    - **Recipient email address:** Enter an email address that you can access.
    
    ![3](https://github.com/user-attachments/assets/ddcc0cdb-2630-46d6-9f36-51ba491164bf)

    
6. Choose **Create health check**.
    
    Route 53 now checks the health of your site by periodically requesting the domain name that you provided and verifying that it returns a successful response.
    
    The health check might take up to a minute to show a *Healthy* **Status**. Choose the refresh icon to update your view of the current status.
    
7. Select **Primary-Website-Health**, and then choose the **Monitoring** tab.
    
    This tab provides a view of the status of the health check over time. It might take a few seconds before the chart becomes available. Choose the refresh icon to update your view.
    
8. Check your email. You should have received an email from AWS Notifications.
9. In the email, choose the **Confirm subscription** link to finish setting up the email alerting that you configured when you created the health check.

![4](https://github.com/user-attachments/assets/3e00c037-a88f-41c8-b4b8-73882e8181c4)

![5](https://github.com/user-attachments/assets/a925d260-c1b5-447b-a95c-053b7f3aa912)


## **Task 3: Configuring Route 53 records**

In the following tasks, you create Route 53 records for the hosted zone.

### **Task 3.1: Creating an A record for the primary website**

You now configure failover routing based on the health check that you just created.

1. In the Route 53 console, in the left navigation pane, choose **Hosted zones**.
    
    The domain name **XXXXXX_XXXXXXXXXX.vocareum.training** (where the Xs are digits unique to your AWS account) has already been created for you.
    
    All lab participants have been given a unique domain name.
    
2. Choose **XXXXXX_XXXXXXXXXX.vocareum.training** to display the two records that already exist in this hosted zone.
    
    These two records were created when the domain was registered with Route 53. The **NS**, or name server record, lists the four name servers that are the authoritative name servers for your hosted zone in the **Value/Route traffic to** column. You should not add, change, or delete name servers from this record.
    
    The **SOA**, or start of authority record, identifies the base Domain Name System (DNS) information about the domain in the **Value/Route traffic to** column. It was also created when the domain was registered with Route 53.
    
   ![6](https://github.com/user-attachments/assets/ac33b965-c8ea-445f-b75d-befb6f771c3e)

    
3. Choose **Create record**, and configure the following options:
    - **Record name:** Enter `www`
    - **Record type:** Choose **A - Routes traffic to an IPv4 address and some AWS resources**.
    - **Value:** In the text box, enter the IP address for **CafeInstance1IPAddress**.
    - **TTL (seconds):** Enter `15`
    - **Routing policy**: Choose **Failover**.
    - **Failover record type:** Choose **Primary**.
    - **Health check ID:** Choose **Primary-Website-Health**.
    - **Record ID:** Enter `FailoverPrimary`
4. Choose **Create records**.
    
    The A-type record that you created should now appear as the third record on the **Hosted zones** page.
    ![7](https://github.com/user-attachments/assets/83160ce3-ff8d-41ea-bf0e-5ee648bfcf40)


### **Task 3.2: Creating an A record for the secondary website**

Now you create another record for the stand-by/secondary web server.

1. Choose **Create record**, and configure the following options:
    - **Record name:** Enter `www`
    - **Record type:** Choose **A - Routes traffic to an IPv4 address and some AWS resources**.
    - **Value:** In the text box, enter the IP address for **CafeInstance2IPAddress**. To find this value, at the top of these instructions, choose **Details**, and then choose **Show**, or copy it from the values that you pasted into a text editor earlier in the lab.
    - **TTL (seconds):** Enter `15`
    - **Routing policy**: Choose **Failover**.
    - **Failover record type:** Choose **Secondary**.
    - **Health check ID:** Leave this field empty.
    - **Record ID:** Enter `FailoverSecondary`
2. Choose **Create records**.
    
    Another A-type record should now be listed on the **Hosted zones** page.
    
    You have now configured your web application to fail over to another Availability Zone.
    
  ![8](https://github.com/user-attachments/assets/9697633c-d16f-4035-8e39-6868e8298afa)

    

## **Task 4: Verifying the DNS resolution**

In this task, you visit the DNS records in a browser to verify that Route 53 is pointing correctly to your primary website.

1. Select the check box for either one of the A records. A **Record details** panel appears that includes the **Record name**. Copy the **Record name** value of the A record.
2. Open a new browser tab. Paste the A record name, enter `/cafe` at the end of the URL, and then load the page.
    
    The café primary website should load, as indicated by the **Server Information** section of the page, which should display the **Region/Availability Zone**.
    
    **Tip:** The URL should be [http://www.XXXXXX_XXXXXXXXXX.vocareum.training/cafe/](http://www.xxxxxx_xxxxxxxxxx.vocareum.training/cafe/), and in this URL, the Xs are unique digits.
    
  ![9](https://github.com/user-attachments/assets/6dd538a1-b7c3-40c6-83ad-ebf592e29f8b)

    

## **Task 5: Verifying the failover functionality**

In this task, you try to verify that Route 53 correctly fails over to your secondary server if your primary server fails. For the purposes of this activity, you simulate a failure by manually stopping **CafeInstance1**.

1. Return to the AWS Management Console. On the **Services** menu, enter and choose `EC2` and then choose **Instances**.
2. Select **CafeInstance1**.
3. From the **Instance state** menu, choose **Stop instance**.
4. In the **Stop instance?** window, choose **Stop**.
    
    The primary website now stops functioning. The Route 53 health check that you configured notices that the application is not responding, and the record entries that you configured cause DNS traffic to fail over to the secondary EC2 instance.
    
5. On the **Services** menu, enter and choose `Route 53`
6. In the left navigation pane, choose **Health checks**.
7. Select  **Primary-Website-Health**, and in the lower pane, choose the **Monitoring** tab.
    
    You should see failed health checks within minutes of stopping the EC2 instance.
    
8. Wait until the **Status** of **Primary-Website-Health** is *Unhealthy*. If necessary, periodically choose  refresh. It might take a few minutes for the status to update.
9. Return to the browser tab where you have the **vocareum_XXXXXX_XXXXXXXXXX.training/cafe** website open, and refresh the page.
    
    Notice that the **Region/Availability Zone** value now displays a different Availability Zone (for example, us-west-2b instead of us-west-2a). You are now seeing the website served from your **CafeInstance2** instance.
    
    If you do not get the correct results, reconfirm that the **Status** of **Primary-Website-Health** is *Unhealthy*, and then try again. It might take a few minutes for the DNS changes to propagate.
    
10. Check your email. You should have received an email from AWS Notifications titled "ALARM: Primary-Website-Health-awsroute53-..." with details about what initiated the alarm.
    
    **Note**: It might take a few minutes before the email arrives.
    
    You have now successfully confirmed that your application environment can fail over from its primary Availability Zone to its secondary Availability Zone if the server in the primary Availability Zone fails.
    
   ![10](https://github.com/user-attachments/assets/ca3af73c-fbb9-4696-be7f-660bccea4832)

    
 ![11](https://github.com/user-attachments/assets/71766318-5b68-49a4-b6e7-405a44bf4d1e)

    

## **Conclusion**

Congratulations! You now have successfully done the following:

- Configured a Route 53 health check that sends emails when the health of an HTTP endpoint becomes unhealthy
- Configured failover routing in Route 53
