# Terraform AWS S3-hosted Static Site

This Terraform module deploys an S3-hosted static site with HTTPS enabled.

## Resources

- S3 bucket to deploy files
- CloudFront distribution fronting the bucket to provide an SSL connection
- Route 53 hosted zone for the BYU sub-domain with records to the CloudFront distribution
- ACM certificate for the URL

## Usage
```hcl
module "s3_site" {
  source    = "github.com/byu-oit/terraform-aws-s3staticsite?ref=v7.0.3"
  site_url       = "my-site.byu.edu"
  hosted_zone_id = "zoneid"
  s3_bucket_name = "bucket-name"
  tags = {
    "tag" = "value"
  }
}
```

> **Note**: Using this module will require you to run `terraform apply` twice. The first time it will create the Route 53 hosted zone, certificate in ACM, and S3 bucket for deployment. Then it will fail because AWS can't validate the certificate. You'll get an error message similar to the image below. Using [this form](https://support.byu.edu/it?id=sc_cat_item&sys_id=2f7a54251d635d005c130b6c83f2390a) or [Teams](https://teams.microsoft.com/l/channel/19%3a7221c80487644c478ceb3f3606d38b15%40thread.tacv2/CES%2520Network%2520Center?groupId=54688770-069e-42a2-9f77-07cbb0306d01&tenantId=c6fc6e9b-51fb-48a8-b779-9ee564b40413), ask the network team to set up a record in BlueCat for your desired subdomain name, pointing to the name servers of the hosted zone created by Terraform (which can be found in the Route 53 console). After AWS has validated the certificate (which you can see in the ACM console), run `terraform apply` again and it should succeed.
> 
> ![First Terraform Error: InvalidViewerCertificate](readme/terraform-apply-1.png)

## Requirements

- AWS Provider version 4.48.0 or greater

## Inputs
| Name                   | Type        | Description                                                                       | Default        |
| ---------------------- | ----------- | --------------------------------------------------------------------------------- | -------------- |
| hosted_zone_id         | string      | Hosted Zone ID                                                                    |                |
| index_doc              | string      | The index document of the site.                                                   | index.html     |
| error_doc              | string      | The error document (e.g. 404 page) of the site.                                   | index.html     |
| origin_path            | string      | The path to the file in the S3 bucket (no trailing slash).                        | *Empty string* |
| site_url               | string      | The URL for the site.                                                             |                |
| additional_domains    | list(object) | Additional domains to route to this site, and the associated hosted zones for cert validation | [] |
| wait_for_deployment    | bool        | Define if Terraform should wait for the distribution to deploy before completing. | `true`         |
| s3_bucket_name         | string      | Name of S3 bucket for the website                                                 |                |
| tags                   | map(string) | A map of AWS Tags to attach to each resource created                              | {}             |
| cloudfront_price_class | string      | The price class for the cloudfront distribution                                   | PriceClass_100 |
| cors_rules             | list(object) | The CORS policies for S3 bucket                                                  | []             |
| forward_query_strings  | bool         | Forward query strings to the origin.                                             | `false`        |
| log_cookies            | bool         | Include cookies in the CloudFront access logs.                                   | `false`        |
| force_destroy          | bool         | Destroy site buckets even if they're not empty on a `terraform destroy` command. | `false`        |
| waf_acl_arn            | string       | The ARN of the WAF that should front the CloudFront distribution.                |                |
| cloudfront_function_arn | string      | The ARN of the CloudFront function to attach to the distribution.                | *Empty string* |
| cloudfront_function_event_type | string | The event type that should trigger the CloudFront function.                      | viewer-request |

## Outputs
| Name            | Type                                                                                                     | Description                                             |
| --------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| site_bucket     | [object](https://www.terraform.io/docs/providers/aws/r/s3_bucket.html#attributes-reference)              | The deployed S3 bucket.                                 |
| cf_distribution | [object](https://www.terraform.io/docs/providers/aws/r/cloudfront_distribution.html#attribute-reference) | The deployed CloudFront distribution.                   |
| dns_record      | [object](https://www.terraform.io/docs/providers/aws/r/route53_record.html#attributes-reference)         | The DNS A-record mapped to the CloudFront Distribution. |
