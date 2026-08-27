## Overview
- Purchase a Domain Name using Amazon Rouute 53
- Upload our site to an Amazon S3 bucket
- Enable S3 Static Website Hosting
- Create a Cloudfront Distribution
- Route Traffic with Route 53
- HTTPS/SSL Encryption

## Step-by-Step Guide

An IAM identity was used for this project instead of the AWS root user, following AWS best practices for minimizing the use of highly privileged root credentials.
<img width="1920" height="1031" alt="01-iam-user-acc-create" src="https://github.com/user-attachments/assets/d1d029ab-6e26-419f-962d-671497b19159" />



### 1. Purchasing a Domain Name
<img width="1897" height="951" alt="02-register-domain-route-53" src="https://github.com/user-attachments/assets/05bacacd-54a5-4aaa-a4ec-2f6c8511fc34" />


The first step is to purchase a domain name. In this project, No domain was purchased since its cost-optimized (even completely cost-free)
You start by searching for an available domain name, provide your contact details, verify the information, and complete the purchase.
It's important to understand that buying a domain is a paid service. This is different from something like WordPress hosting, where you're also paying for computing/hosting resources.
Once we have our domain, we can proceed to host our website.

## 2. Creating an S3 Bucket and Hosting the Website

Next, we create an S3 bucket to store our website files.
For this project, a static HTML resume website was generated with Claude. The website consisted of two files:
- index.html
- resume.pdf

When creating the bucket, we need to consider Block Public Access.

For the traditional S3 static website-hosting approach, the website needs to be publicly accessible, so the relevant public-access restrictions have to be configured appropriately. However, in a modern production setup, we generally don't expose the S3 bucket directly to the public. Instead, we can put CloudFront in front of it and restrict direct bucket access.

We can also add tags to the bucket to help identify and organize resources.

Bucket versioning is optional for a simple project, but it's very useful in production because it allows us to keep previous versions of objects and recover from accidental changes or deletions.

Server-side encryption is also normally recommended rather than disabling it.

After creating the bucket, we upload our website files.

<img width="1920" height="1022" alt="03-creeate-s3-bucket" src="https://github.com/user-attachments/assets/0d5e0a1f-ed09-4ebe-9080-4cf2a69070e1" />
<img width="1920" height="1031" alt="04-upload-files-to-S3" src="https://github.com/user-attachments/assets/29c74677-35ee-4222-90e5-de77a5c94618" />


Once the files are uploaded, we configure the S3 bucket for static website hosting.
We go to the bucket's properties and enable static website hosting.
We then specify the index document, which in this case is: index.html

<img width="1920" height="1013" alt="05-enable-S3-hosting" src="https://github.com/user-attachments/assets/a9dcda61-bf2b-4bf2-92cb-0e38e72aab37" />


### If you encounter a 403 Forbidden / AccessDenied error when trying to access your website, check the permissions on your S3 bucket.
<img width="1917" height="650" alt="06-s3-host-error" src="https://github.com/user-attachments/assets/67103b60-b1ea-42ff-b606-6d171e016e0f" />


For a traditional S3 static website hosting setup, the bucket may require a bucket policy that allows users to retrieve (s3:GetObject) the website files.

If the required policy is missing, you can add a policy similar to the following, replacing resource-name with your actual bucket name:
`
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::resource-name/*"
    }
  ]
}
`
<img width="1917" height="967" alt="07-s3-host-error-policy-fix" src="https://github.com/user-attachments/assets/9d5bc294-6c2a-45d9-a8c3-1df351d8f0cf" />


### Important
This policy makes the objects in the bucket publicly readable. It is therefore intended for the traditional S3 static website hosting approach.

<img width="1917" height="440" alt="07-s3-host-live" src="https://github.com/user-attachments/assets/e46869c5-efdb-4748-8f2f-cec4ef797683" />



## 3. Creating an SSL Certificate with AWS Certificate Manager

Now we want our website to use HTTPS rather than plain HTTP.

For that, we use AWS Certificate Manager (ACM).
<img width="1901" height="907" alt="08-AWS-Certificate-Manager" src="https://github.com/user-attachments/assets/49f83762-c48a-4f12-ab4f-55ef4ddb0434" />


An SSL/TLS certificate basically proves that we control the domain and allows users to establish an encrypted HTTPS connection with our website.
We navigate to Certificate Manager and choose Request a certificate.
* We enter our domain, for example: websitename.com
* We can also add another domain such as: www.websitename.com

If we want the certificate to cover many subdomains, we could use a wildcard such as:

* *.websitename.com

That could cover:
www.websitename.com
shop.websitename.com
blog.websitename.com  and so on.

We then choose DNS validation.
AWS gives us a DNS record that we need to create in Route 53.
Because Route 53 manages our DNS, AWS can automatically create the required validation record if the domain is hosted there.
ACM then checks that DNS record.
Once AWS confirms that we control the domain, the certificate is issued.

Important: If you're using the certificate with CloudFront, the ACM certificate must be requested in the US East (N. Virginia) region us-east-1, even if your other AWS resources are in another region.

## 4. Creating a CloudFront Distribution

Now we introduce CloudFront.

CloudFront is AWS's Content Delivery Network (CDN).

The basic idea is this:

Normally, a user requests a file directly from our S3 bucket.

With CloudFront, CloudFront sits between the user and S3.

The first time a user requests something, CloudFront can retrieve it from the origin our S3 bucket and cache it at a nearby edge location.

Later, when another user requests the same content, CloudFront can serve it from that nearby edge location instead of going all the way back to the S3 bucket.

This gives users Lower latency, Faster content delivery and Better performance for geographically distributed users
It can also reduce the amount of data that needs to be repeatedly retrieved from the origin.
So you can think of CloudFront as a global delivery layer in front of our website.

### Configuring CloudFront

When creating the CloudFront distribution, we specify our origin. The origin is where CloudFront gets the website content from.
In our case, that's the S3 bucket. 
We then configure CloudFront to redirect HTTP requests to HTTPS.

So if someone enters:
* http://websitename.com

they are redirected to:
* https://websitename.com

We also associate our ACM SSL certificate with the CloudFront distribution.

Finally, we can specify the default root object as: index.html

This means that when someone simply visits the domain, CloudFront knows that index.html is the default page to serve.
After creating the distribution, CloudFront gives us a domain name similar to: d123example.cloudfront.net
That is the CloudFront distribution's own domain.

## 5. Connecting Our Custom Domain Using Route 53

At this point, we have three important pieces:
Domain → Route 53 → CloudFront → S3
Now we need to make our custom domain point to CloudFront.
We go to Route 53 and open the Hosted Zone for our domain.
We create a record for the root domain: websitename.com

For this record, we can use: Record type: A and enable: Alias
Then we choose the CloudFront distribution as the destination.

This means that when someone requests:
* websitename.com

Route 53 directs them to CloudFront.

For www.websitename.com, we can create another DNS record pointing to the same CloudFront distribution.

### DNS Propagation

After changing DNS records, the changes may not appear everywhere immediately.

This is called DNS propagation.

Different DNS resolvers around the world may temporarily have different versions of the DNS information because of caching and TTL. A tool such as WhatsMyDNS can be used to check how the DNS record is resolving from different locations around the world.
Once the DNS changes have propagated, users should be able to visit:
https://websitename.com and reach our website.
