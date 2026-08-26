# Deploying a Highly Available Static Website on AWS

## AIM
This project's aim is simple: deploy a fast, secure, cost optimized and highly available static website on AWS. 

## Service used
- Amazon Route 53
- Amazon S3
- Amazon Cloudfront
- AWS Certificate Manager

## Architecture
This architecture is built to handle a simple HTML static website that is cost-optimized. When a client sends a request (using the domain), Route 53, which is the DNS, routes the client to the website. The website delivers content from the nearest Edge Location, provided by Amazon CloudFront. It is important to deliver content from an Edge Location, which serves as a data center that caches copies of content, because getting a cached copy from these Edge Locations is relatively cheaper than fetching content directly from the S3 bucket. If the requested content isn't already cached at that Edge Location (a cache miss), CloudFront retrieves it from the S3 origin, caches it there for future requests, and then serves it to the client. AWS Certificate Manager handles issuing the SSL certificate, which confirms ownership of the domain and enables HTTPS, supporting SEO best practices for your static website.
<img width="1002" height="842" alt="HIghly Available Static Website on AWS (2)" src="https://github.com/user-attachments/assets/cb81c39d-e4a4-438d-be6c-11c718d40921" />
