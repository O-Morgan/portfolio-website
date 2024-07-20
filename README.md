# Portfolio Website
This is a personal portfolio where I can showcase my work along with any projects that I build.

## Semantic Versioning
This project will utilize semantic versioning (e.g., 0.0.0), where:
- The first digit represents a Major Change and resets the latter two digits to zero when implemented.
- The second digit represents a Minor Change.
- The third digit represents a bug fix/patch.

For more information, see the [Semantic Versioning Documentation](https://semver.org/).

Refer to the [Markdown Basic Syntax](https://www.markdownguide.org/basic-syntax/#:~:text=To%20bold%20text%2C%20add%20two,without%20spaces%20around%20the%20letters.&text=I%20just%20love%20**bold,bold%20text.) for writing documentation.

# AWS Static Website Hosting

## Creating an S3 Bucket

1. **Sign in to AWS Management Console:**
   - Go to AWS Management Console and sign in with your credentials.

2. **Navigate to S3:**
   - Once logged in, go to the Services menu and select S3 under the Storage category.

3. **Create a Bucket:**
   - Click on the `Create bucket` button.

4. **Configure Bucket:**
   - **Bucket name:** Enter a unique name for your bucket. Bucket names must be globally unique across all of AWS.
   - **Region:** Choose the AWS region where you want your bucket to be located. Select the region closest to your target audience for better latency.
   - Click `Next`.

5. **Set Permissions:**
   - Leave the permissions as default for now (block all public access). We'll configure public access settings later for website hosting.

6. **Review and Create:**
   - Review your bucket configurations and click `Create bucket`.

7. **Enable Static Website Hosting:**
   - After the bucket is created, select the bucket from the list.
   - Go to the `Properties` tab.
   - Click on `Static website hosting`.
   - Select `Use this bucket to host a website`.
   - Enter the name of your index document (e.g., `index.html`).
   - Optionally, enter the name of an error document (e.g., `error.html`) for handling 4xx errors.
   - Click `Save`.

8. **Set Bucket Policy for Public Access:**
   - Go to the `Permissions` tab.
   - Click on `Bucket Policy`.
   - Enter a bucket policy that allows public read access to your bucket. Replace `your-bucket-name` with your actual bucket name.

Example of bucket policy:
```json
{
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity XXXXXXXXXXXXXXXX"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```
## Checking Availability of Domain Name via Route 53

1. **Go to Route 53 Dashboard:**
   - In the AWS Management Console, go to Services and select Route 53 under the Networking & Content Delivery category.

2. **Check Domain Availability:**
   - In the Route 53 dashboard, under Domain Registration, click `Check domain availability`.
   - Enter the domain name you want to check (e.g., `example.com`) and click `Check`.
   - Route 53 will show if the domain name is available for registration.

## Purchasing and Configuring a Domain via Route 53

1. **Purchase Domain:**
   - If the domain is available, follow the prompts to purchase it. Complete the registration process.

2. **Configure Domain:**
   - Once purchased, go to the Route 53 dashboard.
   - Select `Hosted zones` and choose your domain.
   - Click `Create Record Set` to create a new record.
   - For the name, enter `www` or the desired subdomain.
   - For the type, select `A - IPv4 address`.
   - For the alias, select `Yes` and choose the CloudFront distribution you created earlier.
   - Click `Create`.

## Creating a CloudFront Distribution

1. **Create a CloudFront distribution:**
   - Click `Create a CloudFront distribution`.

2. **Origin Domain:**
   - Your bucket should auto-populate for you to select `arn:aws:s3:::your-bucket-name **add the wildcard at the end of bucket name, forwardslash plus star** Ignore the error warning to use a website endpoint.

3. **Name of Origin:**
   - This should also auto-populate with the bucket endpoint name.

4. **Origin Access:**
   - Select `Legacy Access`.
   - Select `Update bucket policy`.
   - Select `Create a new OAI` (This will create a user for the bucket, and the bucket will not be accessible from anywhere else).

5. **View:**
   - Redirect HTTP to HTTPS.

6. **Allowed HTTP Methods:**
   - Select `GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE`.

7. **Alternate Domain Name (CNAME):**
   - Optional. Here you can put `www.example.com`.

8. **Custom SSL Certificate:**
   - Optional. Request a certificate (The certificate must be raised in the US East, N. Virginia Region, `us-east-1`).
   - Request a public certificate.
   - Fully qualified domain name: `example.com`.
   - DNS validation method: Request.

9. **DNS Validation:**
   - Go back into the certificate after a refresh; you should see that a CNAME and CNAME value have been created. These are needed for validation in Route 53 and other DNS providers. For other providers, you will need to copy and paste these to their site for validation to be successful. Note that it can take a short while to validate with other providers. With AWS Route 53, this is done for you at the click of a button.

10. **Default Root Object:**
    - Optional. Enter `index.html`. If you don't add this, you will have to do it every time.

11. **Firewall:**
    - On or off. I have left this off for cost purposes and this is an isolated project for myself.

12. **Standard Logging:**
    - On or off. You can turn this on and it will help with fault finding, but I have left it off.

13. **IPv6:**
    - On or off. Due to IPv4 addresses now incurring a charge, I have turned on IPv6.

14. **Create Distribution:**
    - Click `Create Distribution`.

## Route 53
- The CName records created in certificate manager now need configuring. 
1. Go to Route53, edit the record to point to CloudFront.
   - example.com
   - www.example.com

### Configure Code Pipeline 
1. Go to CodePipeline. 
2. Make sure you are in the correct region.
3. Create Pipeline.
4. Name the pipeline.
5. Artifact store:
   - Select default location.
   - Select default AWS managed key and hit next.
6. Source Provider:
   - Github V1.
   - Connect to Github.
   - Authorise.
   - Confirm.
7. Repository:
   - Select repo link.
   - Branch = Main.
   - Change detection options, select Github webhook (recommended).
8. Build provider - This can be supplied (it can be a buildspec.yaml file) I didn't use this.
   - Deploy Provider = S3.
   - Region.
   - Bucket - find your bucket.
   - Tick extract file before deploy.
   - No additional configuration - Next

`Create Pipeline`

### Invalidating the Cache 
1. Try changing the code in Github using the correct methods i.e. creating an issue, creating a branch and then checking in the change correctly. You will notice the website won't display the changes immediately, this because the content is cached by CloudFront.
2. To get around this you need to invalidate the cache.
3. Go to CloudFront.
   - Invalidations.
   - Create.
   - `/*` wildcard mean all files.
   - Refresh and the website and it should now display the changes.

`All Done`

**Note for self:** For images and style CSS to work I had to use the relative path, this worked in Github when deployed via a local server for testing and it continued to work when deployed via AWS. :smiley:




    