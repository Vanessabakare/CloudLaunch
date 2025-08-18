# CloudLaunch: AWS Deployment Project

This repository contains a project that showcases my understanding of AWS core services by securely hosting a static website using **Amazon S3** and designing a **custom VPC network layout** with restricted IAM access.

---

## Task 1: Static Website Hosting Using S3 and IAM

I opened the AWS Management Console and navigated to the S3 service to begin setting up the storage infrastructure. I created three buckets with different access configurations:

- **`cloudlaunch-site-bucket2025`**  
  For this bucket, I enabled static website hosting and uploaded a simple `index.html` and `style.css` files. I made this bucket publicly readable to serve as the frontend for a basic company website.

- **`cloudlaunch-private-bucket2025`**  
  For the second bucket, I configured this bucket to be private and gave a specific IAM user permission to upload (`PutObject`) and download (`GetObject`) objects. This was used for internal document storage.

- **`cloudlaunch-visible-only-bucket2025`**  
  I set this bucket to allow the IAM user to list the bucket name but not access its contents. This is useful for visibility without data access.

Next, I went to the IAM section and created a user named `cloud-watch-user`. I wrote a custom JSON policy that:

- Grants `ListBucket` access to all three buckets
- Grants `GetObject` and `PutObject` on `cloudlaunch-private-bucket2025`
- Grants `GetObject` on `cloudlaunch-site-bucket2025`
- Denies `DeleteObject` anywhere
- Prevents access to the contents of `cloudlaunch-visible-only-bucket2025`

I attached this policy to the IAM user and tested access. I also optionally created a CloudFront distribution in front of the public S3 bucket to enable HTTPS and caching.

---

### S3 Static Website URL

[http://cloudlaunch-site-bucket2025.s3-website-us-east-1.amazonaws.com](http://cloudlaunch-site-bucket2025.s3-website-us-east-1.amazonaws.com)

### CloudFront URL (Optional)

[https://d2uwi104y8x2zw.cloudfront.net](https://d2uwi104y8x2zw.cloudfront.net)

---

## Task 2: VPC Network Design

I went to the VPC dashboard and created a new VPC called `cloudlaunch-vpc2025` with a CIDR block of `10.0.0.0/16`.

Then I created the following subnets:

- `cloudlaunch-public-subnet1` (CIDR: 10.0.1.0/24)  
- `cloudlaunch-application-subnet` (CIDR: 10.0.2.0/24)  
- `cloudlaunch-db-subnet` (CIDR: 10.0.3.0/28)

I created an Internet Gateway named `cloudlaunch-igw` and attached it to the VPC.

I configured route tables:

- `cloudlaunch-public-rt1`: Associated it with the public subnet and added a route to send internet-bound traffic (0.0.0.0/0) to the IGW.
- `cloudlaunch-app-rt1` and `cloudlaunch-db-rt1`: These private route tables were associated with their respective subnets and had no route to the internet.

I then created two security groups:

- **`cloudlaunch-app-sg1`**: Allows HTTP traffic (port 80) from within the VPC (`10.0.0.0/16`)
- **`cloudlaunch-db-sg1`**: Allows MySQL traffic (port 3306) only from the app subnet (`10.0.2.0/24`)

Finally, I created another IAM policy that gives the `cloud-watch-user` read-only access to VPC components such as VPCs, subnets, route tables, and security groups.

---

## IAM Policy (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ListAccess",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::cloudlaunch-site-bucket",
        "arn:aws:s3:::cloudlaunch-private-bucket",
        "arn:aws:s3:::cloudlaunch-visible-only-bucket"
      ]
    },
    {
      "Sid": "AllowSiteBucketReadOnly",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::cloudlaunch-site-bucket/*"]
    },
    {
      "Sid": "AllowPrivateBucketReadWrite",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": ["arn:aws:s3:::cloudlaunch-private-bucket/*"]
    },
    {
      "Sid": "AllowReadOnlyVpcAccess",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVpcs",
        "ec2:DescribeSubnets",
        "ec2:DescribeRouteTables",
        "ec2:DescribeSecurityGroups"
      ],
      "Resource": "*"
    }
  ]
}
```

## IAM User Access

| Detail              | Value                                                                  |
|---------------------|------------------------------------------------------------------------|
| Console URL         | [https://xxxxxxxxxxxx.signin.aws.amazon.com/console]([https://console.aws.amazon.com](https://xxxxxxxxxxxx.signin.aws.amazon.com/console))       |
| Account ID          | xxxxxxxxxxxx                                   |
| Account Alias       |                                          |
| IAM Username        | cloud-watch-user                                                      |
| Temporary Password  | Provided privately.  |
