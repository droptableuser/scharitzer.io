---
title: "Static Website Hosting 2022"
date: 2022-02-23
tags: ["Python","AWS"]
---
All code for this blog post can be found at:

[Static Website S3 Repository](https://github.com/droptableuser/static_website_s3/)
<!--more-->
So we all know the 200 something tutorials about static website hosting with S3, including the official one from AWS.

We all at some point or another we end up with a website hosted on S3 with all bells and whistles, but something looks off.


Well somewhere we side stepped and the only fix we knew to do was to activate public access to our bucket. Our website is up, but it doesn't look right, we know it. We would need to fix it, but where to start? What was the issue that made us turn the public access switch on? Most probably it was an issue with the CDN, but who knows?

So we leave it be. 

This is where the usual story of a misconfiguration ends for the time being. For our static website with about 4 visitors (Hi mom!) this doesn't mean much. 

For our production app or data pipeline this could spell doom. No solid means of tracking changes to your infrastructure and no way to spot a misconfiguration. This is the worst case for growth, rapid prototyping and security. 

So how do we remediate this issues? Infrastructure as code. With the cloud development kit there is native tooling provided by AWS directly. 

First we create a S3 Bucket 🪣 to host our static website:
```python
bucket = s3.Bucket(self,domain,
     access_control=s3.BucketAccessControl.PRIVATE,
     removal_policy=cdk.RemovalPolicy.DESTROY,
     website_redirect=redirect)
```
This creates a private bucket which will removed upon destruction of the stack. The `website_redirect` can be set to `None` for all use cases that are not redirecting websites to another URL. My code covers this use case as well.
 
We can also directly deploy the code into the S3 Bucket 
```python
sources = [s3deploy.Source.asset("website/public")]            
s3deploy.BucketDeployment(self,domain+"BucketDeployment",
    destination_bucket=bucket,
    sources=sources,
    retain_on_delete=False)
```
For this to work we only need to either track the website code within the same repository or have our deployment pipeline take care of it.

Next we need to create an Origin Access Identity. This object is needed for the CloudFront Distribution to grant read only access our website in the S3 bucket.
```python
origin_access_identity = cloudfront.OriginAccessIdentity(self,domain+'OriginAccessIdentity')
bucket.grant_read(origin_access_identity)
```
Managing access rights via CDK method grants is a best practice.

The next two steps can depend where we actually host your DNS records. I use Route53 from AWS, but to be honest, this is the most expensive part of this whole stack for the amount of traffic I see. 

First we retrieve the Zone from Route53, we need it to automatically validate it for the HTTPS certificate for our custom domain CloudFront distribution. 
```python
hosted_zone = route53.HostedZone.from_lookup(
     self, domain+"HostedZone", domain_name=domain)

cert = certificatemanager.DnsValidatedCertificate(
     self, domain+"Certificate",
     domain_name=domain,
     subject_alternative_names=domain_names,
     hosted_zone=hosted_zone,
     region="us-east-1")
```
This method `DnsValidatedCertificate()`is used, because the certificate for CloudFront distributions needs to be created in the "us-east-1" region. We can also pass several subject alternative names (SAN) for the certificate to cover. In this code examples it is only "www[.]scharitzer.io" and "scharitzer.io" per default. 

````python
error_response = [cf.ErrorResponse(
    http_status=404,
    response_page_path="/404.html")]


distribution = cloudfront.Distribution(self,domain+'websitedistribution',       
    default_behavior=cloudfront.BehaviorOptions(
      origin=origins.S3Origin(
          bucket,
          origin_access_identity=origin_access_identity),
      viewer_protocol_policy=cf.ViewerProtocolPolicy.REDIRECT_TO_HTTPS
     ),
    domain_names=domain_names,
    price_class=cloudfront.PriceClass.PRICE_CLASS_100,
    error_responses=error_response,
    certificate=cert, 
    default_root_object="index.html")
```
First off, we create the 404 error page object, that is already provided by our website. This is needed for the CloudFront Distribution. And then we finally create the CloudFront distribution. The `default_behavior` defines what our distribution will be serving. In our case it will be serving from an S3 origin with our previously defined access identity. We further also specify the `domain_names`to be served from this distribution. The `price_class` argument takes an enum value of either `PRICE_CLASS_100` (representing the edge locations: US, Canada, Europe & Isreal) or `PRICE_CLASS_200` (including the 4 previously mentioned locations +South Africa, Kenya, Middle East, Japan, Singapore, South Korea, Taiwan, Hong Kong, & Philippines). We chose the cheaper option, because we do not need the improved latency for the other regions in this use case. Also the `error_responses`, `certificate` and `default_root_object`are set. And finally, since it is 2022, and sane defaults are still not provided we have to set `viewer_protocol_policy` to the enum value for redirecting all visitors to HTTPS.

The last puzzle piece is to use the CloudFront distribution object to update the DNS records for the domain (A and AAAA)
```python
route53.ARecord(self,
    record+"Aalias",
    record_name=record,
    zone=hosted_zone,
    target=route53.RecordTarget.from_alias(targets.CloudFrontTarget(distribution)))
route53.AaaaRecord(self, 
    record+"AAAAAalias",
    record_name=record,
    zone=hosted_zone,
    target=route53.RecordTarget.from_alias(targets.CloudFrontTarget(distribution)))
```

If the domain would not be hosted in Route53 and the website we would be hosting should be located on the apex domain (e.g.: "scharitzer[.]io") the hoster would need to support DNS flattening for this to work. (CNAME at apex domain is usually not supported)

On the next post I will be looking at the automation I use, to deploy all my static websites. 
