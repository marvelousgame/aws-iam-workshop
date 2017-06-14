# AWS IAM Workshop

![architecture.png](./images/architecture-diagram.png)

This workshop will take you basic step by step how to create **IAM user**, launch **EC2 instance** with **IAM role**, access **S3 bucket using temporary credential**, and enabling **CloudTrail**.


## Implementation Steps

### Create IAM User
If you already have an IAM User which have admin privileges, then you can skip this step

- Go to [https://console.aws.amazon.com](https://console.aws.amazon.com)
- Enter your root login information
- From the AWS Management Console, choose **Services** then select **IAM** under Security
- From navigation pane, choose **Users**
- Click **Add user** button
- For **User name**, type a user name (ex. your name or `administrator`)
- For **Access type**, select check box next to **AWS Management Console access**
- For **Console password**, select **Custom password**
- Type the new user's password in the text box. 
- Optionally you can select / unselect **Require password reset** 
- Choose **Next:Permissions**
- Choose **Attach existing policies directly**
- In the policy list, search `AdministratorAccess`
- Select the checkbox
- Choose **Next:Review**
- Choose **Create User**
- (Optional) Click **Download.csv** to download your credential
- Click **Close**
- Get IAM User Sign-in Link
	- In IAM page, from navigation pane, choose **Dashboard**
	- Find the URL for **IAM users sign-in link**
	- In the future, you will use this URL to login to AWS Console
	- URL format will be: [https://aws-account-number.signin.aws.amazon.com/console/](https://aws-account-number.signin.aws.amazon.com/console/)
	- (Optional) You can also use alias for the sign-in link [http://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html](http://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html)

---

### Choose Region

Please select a working region before continuing with other step.

- Login to **AWS Management Console** using **IAM User Sign-In Link** (not **Root Account**) 
- From the AWS Management Console, in top right side, change to your working region.

---

### Enable CloudTrail
We recommend that you enable CloudTrail as part of a general security best practice. 

If you already enable CloudTrail in your working region, you can skip these steps.

- From the AWS Management Console, choose **Services** then select **CloudTrail** under Management Tools
- Choose **Get Started Now** or **Create Trail**
- For **Trail name**, type a name for your trail (ex. `Default`)
- For **Apply trail to all regions**, choose **Yes** to receive log files from all regions. This is the default and recommended setting. 
	- If you choose No, the trail logs files only from the region in which you create the trail. 
- For **Read/Write events**, choose **All**
- In **Storage location**, for **Create a new S3 bucket**, choose **Yes**
- For **S3 bucket**, type a bucket name. The name must be globally unique. (ex. `cloudtrail-yourname`)
- Click **Create**

---

### Create IAM Role
- Login to **AWS Management Console** using **IAM User Sign-In Link** (not **Root Account**) 
- From the AWS Management Console, choose **Services** then select **IAM** under Security
- In the navigation pane, click **Roles**
- Click **Create New Role**
- Under **Select Role Type**, verify that the **AWS Service Roles** option is selected 
- In the row for **Amazon EC2**, click **Select**
- Select checkbox **AmazonS3FullAccess** policy from the list, click **Next Step**
- For **Role Name**, type `MyApplicationRole`
- Click **Create Role**

---

### Create S3 Bucket
- From the AWS Management Console, choose **Services** then select **S3** under Storage
- Click **Create Bucket**
- For **Bucket Name**, type a unique bucket name
- For **Region**, choose `your-region`
- Click **Create**
- Click bucket you just created, verify that the bucket is empty.

---

### Create Key-Pair
You can skip below steps if you already have existing **EC2 Key Pair** in current working region. 

- From the AWS Management Console, choose **Services** then select **EC2** under Compute.
- Click **Create Key Pair**
- Specify **Key Pair Name**, click **Create**
- Save the **.pem** file to your local folder

---

### Launch EC2 Instance
- On the **Services** menu, click **EC2**
- Click **Launch Instance**
- From the **Quick Start** menu, in the row for the first **Amazon Linux AMI**, click **Select**
- On the **Choose an Instance Type**, choose **t2.micro**, click **Next: Configure Instance Details**
- On the **Configure Instance Details**
	- Network: **DefaultVPC**
	- Subnet: **No preference**
	- Auto-assign Public IP: **Enable**
	- IAM role: **WebServerRole** (which you created in previous task)
- Click **Next: Add Storage**
	- Leave the storage settings as default
- Click **Next: Tag Instance**
- Click **Add Tag**
- For **Key**, type **Name**
- For **Value**, type **MyInstance**
- Click **Next: Configure Security Group**
- For **Assign a security group**, verify that the **Create a new security group** option is selected
	- Security Group Name: **MySecurityGroup**
	- Description: **My Security Group**
- There should be an existing **SSH** rule. Leave that rule as-is
- Click **Review and Launch**
- Review the settings and then click **Launch**
- When prompted about keypair, choose your **keypair**, select the acknowledgement check box, and then click **Launch Instances**
- Click **View Instances**
- Select the instance that you just created
- On the **Description** tab in the lower pane, note the **Public IP** of the instance
- Wait for the instance to reach **Instance State: running** and **Status Checks: 2/2 checks passed**

---

### Connect to Instance (Windows Only)
- We are using Putty to connect to instance
- If you don't have one, download putty here: [https://the.earth.li/~sgtatham/putty/latest/x86/putty.exe](https://the.earth.li/~sgtatham/putty/latest/x86/putty.exe)
- Convert .pem to .ppk
	- We will use **PuTTYgen** to convert .pem to .ppk
	- if you don't have one, download **PuTTYgen** here: [https://the.earth.li/~sgtatham/putty/latest/w64/puttygen.exe](https://the.earth.li/~sgtatham/putty/latest/w64/puttygen.exe)
	- Run **PuTTYgen**
	- Under **Type of key to generate**, choose **RSA**
	- Choose **Load**. To locate your .pem file, select the option to display files of all types. 
	- Select your .pem file
	- Choose **Save private key**
	- Choose **Yes** in warning window
	- Specify the same name for the key. (.ppk automatically added as file extension)
	- .ppk file created
- Launch **Putty**
- For **Host Name**, enter the **Public IP** address from your instance which you copied earlier
- In the **Connection** list, expand **SSH**
- Click **Auth**
- For **Private key file for authentication**, browse **.ppk** file, click **Open**
- In the **Putty Security Alert** dialog box that opens, click **Yes** to add the key to **Putty's** cache
- For **login as**: type **ec2-user** and press ENTER. You are now logged in to your instance

---

### Connect to Instance (Linux / MacOS Only)
- Run the following commands in Terminal:
	- **chmod 400** `<path-and-name-of-pem>`
	- **ssh -i** `<path-and-name-of-pem>` **ec2-user@**`<public ip>`
- For `<path-and-name-of-pem>`, substitute with your **.pem** file
- For `<public ip>`, substitute with your **Public IP Instance**

---

### Use EC2 Instance to Access S3 Resource

You have launched EC2 instance assuming IAM role which have permission to S3. Now let's use EC2 instance to access S3 using AWS CLI. You also can use AWS SDK.

![iam-role](./images/roles-usingrole-ec2roleinstance.png)

- Connect to your EC2 instance using SSH
- run command: **aws-version**
	- Showing current AWS CLI version
	- By default, AWS CLI tool is included in Amazon Linux AMI
- run command: **aws s3 ls**
	- Showing current S3 bucket list
- run command: **aws s3 ls `bucket-name`**
	- replace the `bucket-name` with the S3 bucket that previously created
	- this bucket should be empty
- run command: **wget https://ivan-cheng-bucket-tokyo.s3.amazonaws.com/public/SampleImagesAndVideos.zip**
	- download videos and contents from internet
- run command: **mkdir SampleImagesAndVideos**
- run command: **unzip SampleImagesAndVideos.zip -d SampleImagesAndVideos**
- run command: **cd SampleImagesAndVideos**
- run command: **ls**
	- you can see the folder contains some images and videos
- Now we want to upload these contents to your S3 bucket using AWS CLI through EC2
- For the following command, replace `bucket-name` with your own bucket name
- run command: **aws s3 cp /home/ec2-user/SampleImagesAndVideos s3://`bucket-name`/SampleImagesAndVideos --recursive**
	- This will copy all files (including folder) to your S3 bucket
- run command: **aws s3 ls `bucket-name`/SampleImagesAndVideos/**
	- Showing you current uploaded content in your bucket
- Uploaded content is private content, so you need to generate **presign url** to allow private access
- run command: **aws s3 presign s3://`bucket-name`/SampleImagesAndVideos/Diving-1084.mp4**
	- Generate presign url for one of the content
	- By default, this url will be expired after 1 hour.
- Copy the URL, and paste it to your browser

---

### Find the EC2 Instance Access Key & Secret Key
So far, noticed that you never generate or hard code any access key / secret key to your application on EC2 to access to S3.  

Because when you launch Ec2 instance you associate it with **IAM Role**.
This is the best practice to allow AWS generate **temporary credential (access key / secret key)** and **rotate it before expires**.

The **temporary credential** is stored on EC2 instance metadata.   

Below step shows you how to view current EC2 temporary credential

- Access your EC2 instance (using SSH)
- run command: **curl http://169.254.169.254/latest/meta-data/iam/security-credentials/MyApplicationRole/**
- Temporary credential will be like this

```
{   
  "Code" : "Success",   
  "LastUpdated" : "2012-04-26T16:39:16Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAIOSFODNN7EXAMPLE",
  "SecretAccessKey" : "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token" : "token",
  "Expiration" : "2017-05-17T15:09:54Z"
}
```

---

### Viewing CloudTrail Logs

Typically, CloudTrail delivers an event within 15 minutes of the API 
call.   

CloudTrail delivers log to your S3 bucket in RAW file.

**How to view / process?**

- **[CloudTrail Processing Library](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/use-the-cloudtrail-processing-library.html)** to process the log
- **[Partner Solution](https://aws.amazon.com/cloudtrail/partners/)** (Ex: [Sumo Logic](https://www.sumologic.com/application/cloudtrail/))
- Build your own: use CloudTrail to CloudWatch Logs + Lambda + ElasticSearch with Kibana (**[aws blog](https://aws.amazon.com/blogs/aws/cloudwatch-logs-subscription-consumer-elasticsearch-kibana-dashboards/)**)
- **CloudTrail console** to review API activity for a region during the last seven days.


View API Activity through **CloudTrail Console**:

- From the AWS Management Console, choose **Services** then select **CloudTrail**
- In the navigation pane, choose **API Activity History**
- A list of events appears in the content pane with the latest event first.
- Scroll down to see more events. 
- If you want to actions from specific user
	- For **Filter**, select **User name**
	- Type **IAM User** that you previously created
	- Click **Enter** or **Refresh** icon on the right side
	- It will show you the related logs
 
---

### Cleanup

- Terminate your **EC2 instance** (along with the EBS volume)
- Empty and Delete your working **S3 bucket** (the one that hold your images and videos)
- (Optional) Delete **IAM Role**
- (Optional but not recommended) Delete **CloudTrail** and delete related **CloudTrail S3 Bucket**
