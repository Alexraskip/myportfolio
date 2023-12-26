#Secure Website Hosting with Amazon CloudFront, S3 and CloudFormation

I am sure we all know the amazing powers of Amazon Simple Storage Service (S3). One of the most common use cases for S3 is simple website hosting. Website hosting on Amazon S3 is very common, and it is one of the many capabilities of this amazing service that most beginners learn about. In this course, we are going to explore a different approach.

The security of your static content that is served via your S3 bucket is very important, and Amazon S3 makes it possible for you to block public access to your bucket for your static content. There are better ways to serve the static content stored in your bucket using AWS CloudFront. CloudFront requires some initial setup, and once you have set it up, it works together with your website or application and speeds up the delivery of your content.

In this project, I am going to explore the Amazon CloudFront service with Amazon S3 for simple website hosting. To make it more fun, you will do it all using Infrastructure as Code (IaC).

In this project, I will create a CloudFormation template, which will build the resources I need for this workshop.

Open my VSCode and open a new blank text file. This is where I will write my yaml code for your CloudFormation template.
In the blank text file, copy and paste the following code (code in main.yaml file).

#Exploring the resources in the main.yaml file:
The first resource I created was an S3 bucket.

#Amazon S3
The AWS::S3::Bucket resource creates an Amazon S3 bucket in the same AWS Region where I create the AWS CloudFormation stack.
Resources:
  S3Bucket:
    DeletionPolicy: 'Delete'
    Metadata:
      Comment: 'Bucket to store some data'
    Properties:
      AccessControl: 'Private'
      BucketName: !Sub 'cf-simple-s3-origin-${AWS::StackName}-${AWS::AccountId}'
    Type: 'AWS::S3::Bucket'

S3Bucket Defines an S3 bucket resource.
DeletionPolicy specifies the deletion policy for the bucket; the 'Delete' policy means the bucket is deleted when the stack is deleted.
Metadata provides additional metadata for the resource. Here I have put a comment to explain what the bucket is for: 'Bucket to store some data’.
AccessControl This is one of the properties for my bucket, It sets the access control for the bucket to 'Private.'
BucketName uses the !Sub function to dynamically generate the bucket name based on the stack name and AWS account ID.
Type specifies the resource type, in this case, an S3 bucket.

#Amazon S3BucketPolicy
The code block defines the S3 bucket policy for my S3 bucket.
A bucket policy is a resource-based policy that I can use to grant access permissions to my Amazon S3 bucket and the objects in it.

S3BucketPolicy:
    Metadata:
      Comment: 'Bucket policy to allow cloudfront to access the data'
    Properties:
      Bucket: !Ref S3Bucket
      PolicyDocument:
        Statement:
          - Action:
              - 's3:GetObject'
            Effect: 'Allow'
            Principal:
              CanonicalUser: !GetAtt CfOriginAccessIdentity.S3CanonicalUserId
            Resource:
              - !Sub 'arn:aws:s3:::${S3Bucket}/*'
    Type: 'AWS::S3::BucketPolicy'

S3BucketPolicy defines a bucket policy to grant CloudFront permission to access S3 objects.
Metadata provides additional metadata for the resource. in this case, it’s providing a simple comment
Bucket This is a property of my bucket policy; it is referencing the S3 bucket created earlier.
PolicyDocument is another property of your policy; it defines the policy allowing S3 to GetObject for CloudFront Origin Access Identity on objects in the bucket.
Type Specifies the resource type as an S3 bucket policy.

#Cloudfront distribution
A distribution tells CloudFront where I want content to be delivered and the details about how to track and 
manage content delivery.

CfDistribution:
    Metadata:
      Comment: 'A simple CloudFront distribution with an S3 origin'
    Properties:
      DistributionConfig:
        Comment: 'A simple distribution with an S3 origin'
        DefaultCacheBehavior:
          AllowedMethods:
            - 'HEAD'
            - 'GET'
          CachedMethods:
            - 'HEAD'
            - 'GET'
          Compress: false
          DefaultTTL: 86400
          ForwardedValues:
            Cookies:
              Forward: 'none'
            Headers:
              - 'Origin'
            QueryString: false
          MaxTTL: 31536000
          MinTTL: 86400
          TargetOriginId: !Sub 's3-origin-${S3Bucket}'
          ViewerProtocolPolicy: 'redirect-to-https'
        DefaultRootObject: 'index.html'
        Enabled: true
        HttpVersion: 'http1.1'
        IPV6Enabled: false
        Origins:
          - DomainName: !GetAtt S3Bucket.DomainName
            Id: !Sub 's3-origin-${S3Bucket}'
            OriginPath: ''
            S3OriginConfig:
              OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CfOriginAccessIdentity}'
        PriceClass: 'PriceClass_All'
    Type: 'AWS::CloudFront::Distribution'


CfDistribution defines a CloudFront distribution with an S3 origin.

Metadata provides additional information about the resource.

DistributionConfig configures the CloudFront distribution.

DefaultCacheBehavior configures caching and behavior for default requests.

Specifies allowed and cached methods, compression, TTL settings, and more.

Forwards certain headers and handles cookies.

Specifies the S3 origin and redirects HTTP to HTTPS.

DefaultRootObject specifies the default root object.

Enabled indicates whether the distribution is enabled.

HttpVersion specifies the HTTP version.

IPV6Enabled indicates whether IPv6 is enabled.

Origins this configures the S3 origin for the distribution.

PriceClass specifies the price class for the distribution.

Type specifies the resource type as a CloudFront Distribution


#Cloudfrong Origin Access Identity (OAI)
An Origin Access Identity is a special CloudFront user that I can associate with Amazon S3 origins to secure all or just some of my Amazon S3 content.

CfOriginAccessIdentity:
    Metadata:
      Comment: 'Access S3 bucket content only through CloudFront'
    Properties:
      CloudFrontOriginAccessIdentityConfig:
        Comment: 'Access S3 bucket content only through CloudFront'
    Type: 'AWS::CloudFront::CloudFrontOriginAccessIdentity'


CfOriginAccessIdentity defines a CloudFront Origin Access Identity (OAI).
CloudFrontOriginAccessIdentityConfig configures the CloudFront OAI.
Comment a comment describing the purpose of the OAI.
Type specifies the resource type as a CloudFront Origin Access Identity.

#Outputs
Here, I declare the output values of your stack which you can view in my console.

Outputs:
  S3BucketName:
    Description: 'Bucket name'
    Value: !Ref S3Bucket
  CfDistributionId:
    Description: 'Id for our cloudfront distribution'
    Value: !Ref CfDistribution
  CfDistributionDomainName:
    Description: 'Domain name for our cloudfront distribution'
    Value: !GetAtt CfDistribution.DomainName

Outputs show information about the created resources.
S3BucketName outputs the name of the S3 bucket.
CfDistributionId outputs the ID of the CloudFront distribution.
CfDistributionDomainName outputs the domain name of the CloudFront distribution.

STEPS TO DEPLY TEMPLATE TO THE CLOUD
1. Go to my AWS account and navigate to the CloudFormation console.
2. From my CloudFormation console click on Create Stack.
3. From the Create Stack panel, Select Template is ready
4. Under the Specify template panel, Select Upload a template file.
5. Go back to VSCode and save the text file of the CloudFormation YAML code you created. You can save it with any name you want.
6. Once your .yaml file is saved. Click Choose file to upload the file to the CloudFormation console and click Next.
7. In the Specify stack details page, enter your Stack name then click Next.
8. In the Configure stack options page, you will leave everything as default. You can create tags for your stack. Tags are a way for you to be able to manage, identify, organize, search for, and filter resources, in a way, you can say tags are metadata assigned to your AWS resources.
9. Review your stack, scroll down, and click Submit
10. Your stack should show as CREATE_IN_PROGRESS, it may take some time for your stack to create all the resources. Wait for your stack to show as CREATE_COMPLETE
11. Navigate to the S3 console by typing S3 in the search bar. From the search results click S3 to proceed to the S3 console.
12. Navigate to CloudFront, and click CloudFront in the search bar.
13. From the search results click Cloudfront to proceed to the CloudFront console.
14. On the Distributions page, you should see your CloudFront distribution that was created by CloudFormation.
15. Open the S3 console, and then upload this html.index file to the S3 bucket that CloudFormation created. You can download the file here and save it to your local machine first.
16. Once uploaded, open the file and try to view it. Copy the object URL and paste it into a new tab in your web browser.
17. Files in the bucket are not publicly accessible directly from S3. You will not be able to view the content directly from S3 because you specified to use the Origin Access Identity feature in Cloudfront. 
18. Open the CloudFront console and open the distribution you created with Cloudformation.
19. Copy the Distribution domain name and paste it into a new tab in your web browser.
20. You configured your files to be accessed via the CloudFront distribution. You should have an output.

