# Module 1: Static Web Hosting with Amazon S3

In this module you'll configure Amazon Simple Storage Service (S3) to host the static resources for your web application. In subsequent modules you'll add dynamic functionality to these pages using JavaScript to call remote RESTful APIs built with AWS Lambda and Amazon API Gateway.

## Architecture Overview

The architecture for this module is very straightforward. All of your static web content including HTML, CSS, JavaScript, images and other files will be stored in Amazon S3. Your end users will then access your site using the public website URL exposed by Amazon S3. You don't need to run any web servers or use other services in order to make your site available.

![Static website architecture](../images/static-website-architecture.png)

For the purposes of this module you'll use the Amazon S3 website endpoint URL that we supply. It takes the form `http://{your-bucket-name}.s3-website-{region}.amazonaws.com` or `bucket-name.s3-website.region.amazonaws.com` depending on the region you use. For most real applications you'll want to use a custom domain to host your site. If you're interested in using a your own domain, follow the instructions for [setting up a static website using a custom domain](http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) in the Amazon S3 documentation.

## Implementation Instructions

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### Region Selection

This workshop can be deployed in any AWS region that supports the following services:

- Amazon Cognito
- AWS Lambda
- Amazon API Gateway
- Amazon S3
- Amazon DynamoDB

You can refer to the [region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services. Among the supported regions you can choose are N. Virginia, Ohio, Oregon, Ireland, London, Frankfurt, Tokyo, Seol, Mumbai, and Sydney.

Once you've chosen a region, you should deploy all of the resources for this workshop there. Make sure you select your region from the dropdown in the upper right corner of the AWS Console before getting started.

![Region selection screenshot](../images/region-selection.png)

### 1. Create an S3 Bucket

Amazon S3 can be used to host static websites without having to configure or manage any web servers. In this step you'll create a new S3 bucket that will be used to host all of the static assets (e.g. HTML, CSS, JavaScript, and image files) for your web application.

#### High-Level Instructions

Use the console or AWS CLI to create an Amazon S3 bucket. Keep in mind that your bucket's name must be globally unique across all regions and customers. We recommend using a name like `wildrydes-firstname-lastname`. If you get an error that your bucket name already exists, try adding additional numbers or characters until you find an unused name.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **Services** then select **S3** under Storage.

1. Choose **+Create Bucket**

1. Provide a globally unique name for your bucket such as `wildrydes-firstname-lastname`.

1. Select the Region you've chosen to use for this workshop from the dropdown.

1. Choose **Create** in the lower left of the dialog without selecting a bucket to copy settings from.

    ![Create bucket screenshot](../images/create-bucket.png)

</p></details>

### 2. Upload Content

Upload the website assets for this module to your S3 bucket. You can use the AWS Management Console (requires Google Chrome browser), AWS CLI, or the provided CloudFormation template to complete this step. If you already have the AWS CLI installed and configured on your local machine, we recommend using that method. Otherwise, use the console if you have the latest version of Google Chrome installed.

#### CLI step-by-step instructions (expand for details)

If you already have the CLI installed and configured, you can use it to copy the necessary web assets from `s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website` to your bucket.

Execute the following command making sure to replace `your-bucket-name` with the name you used in the previous section and `aws-region` with the region code (e.g. us-east-1) where you created your bucket.

```
aws s3 sync s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website s3://{your-bucket-name} --region {aws-region} --profile {your-profile-name}
```

If the command was successful, you should see a list of objects that were copied to your bucket.

```
aws s3 ls s3://{your-bucket-name}/ --region {aws-region} --profile {your-profile-name}

                           PRE css/
                           PRE fonts/
                           PRE images/
                           PRE js/
2018-08-15 12:31:35       2992 apply.html
2018-08-15 12:31:35       6540 faq.html
2018-08-15 12:31:35       1406 favicon.ico
2018-08-15 12:31:37       6757 index.html
2018-08-15 12:31:37       6079 investors.html
2018-08-15 12:31:38       2735 register.html
2018-08-15 12:31:38       4329 ride.html
2018-08-15 12:31:38         43 robots.txt
2018-08-15 12:31:38       2590 signin.html
2018-08-15 12:31:38       5047 unicorns.html
2018-08-15 12:31:38       2595 verify.html
```

</p></details>

### 3. Add a Bucket Policy to Allow Public Reads

You can define who can access the content in your S3 buckets using a bucket policy. Bucket policies are JSON documents that specify what principals are allowed to execute various actions against the objects in your bucket.

#### High-Level Instructions

You will need to add a bucket policy to your new Amazon S3 bucket to let anonymous users view your site. By default your bucket will only be accessible by authenticated users with access to your AWS account.

See [this example](http://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) of a policy that will grant read only access to anonymous users. This example policy allows anyone on the Internet to view your content. The easiest way to update a bucket policy is to use the console. Select the bucket, choose the permission tab and then select Bucket Policy.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the S3 console, select the name of the bucket you created in section 1.

1. Choose the **Permissions** tab, then choose **Bucket Policy**.

1. Enter the following policy document into the bucket policy editor replacing `your-bucket-name` with the name of the bucket you created in section 1:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::{your-bucket-name}/*"
            }
        ]
    }
    ```

    ![Update bucket policy screenshot](../images/update-bucket-policy.png)

1. Choose **Save** to apply the new policy.

</p></details>

### 4. Enable Website Hosting

By default objects in an S3 bucket are available via URLs with the structure `http://{regional-s3-prefix}.amazonaws.com/{your-bucket-name}/{object-key}`. In order to serve assets from the root URL (e.g. /index.html), you'll need to enable website hosting on the bucket. This will make your objects available at the AWS region-specific website endpoint of the bucket: `{your-bucket-name}.s3-website-{aws-region}.amazonaws.com`

You can also use a custom domain for your website. For example http://www.wildrydes.com is hosted on S3. Setting up a custom domain is not covered in this workshop, but you can find detailed instructions in our [documentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html).

#### High-Level Instructions

Using the console, enable static website hosting. You can do this on the Properties tab after you've selected the bucket. Set `index.html` as the index document, and leave the error document blank. See the documentation on [configuring a bucket for static website hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html) for more details.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. From the bucket detail page in the S3 console, choose the **Properties** tab.

1. Choose the **Static website hosting** card.

1. Select **Use this bucket to host a website** and enter `index.html` for the Index document. Leave the other fields blank.

1. Note the **Endpoint** URL at the top of the dialog before choosing **Save**. You will use this URL throughout the rest of the workshop to view your web application. From here on this URL will be referred to as your website's base URL.

1. Click **Save** to save your changes.

    ![Enable website hosting screenshot](../images/enable-website-hosting.png)

</p></details>


## Implementation Validation

After completing these implementation steps you should be able to access your static website by visiting the website endpoint URL for your S3 bucket.

Visit your website's base URL (this is the URL you noted in section 4) in the browser of your choice. You should see the Wild Rydes home page displayed. If you need to lookup the base URL, visit the S3 console, select your bucket and then click the **Static Web Hosting** card on the **Properties** tab.

If the page renders correctly (see below for an example screenshot), you can move on to the next module, [User Management](../2_UserManagement).

![Wild Rydes homepage screenshot](../images/wildrydes-homepage.png)
