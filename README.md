
# Amazon API Gateway Custom Domain for Private API

This CDK pattern implements the workaround solution presented [here](https://serverlessland.com/repos/apigw-private-custom-domain-name).
API calls to a fully qualified name (FQDN), such as private.domain.com, are resolved by Route53 to a Network Load Balancer (NLB) and forwarded to the API Gateway.

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Architecture
![Architecture](resources/architecture.png)

## Requirements

* [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and log in. The IAM user that you use must have sufficient permissions to make necessary AWS service calls and manage AWS resources.
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [AWS Cloud Development Kit](https://docs.aws.amazon.com/cdk/latest/guide/cli.html) (AWS CDK) installed

## Sample Stacks
| Stack                                                      | Description                                             |
|------------------------------------------------------------|---------------------------------------------------------|
| [SampleStack](lib/sample-stack.js)                         | Stack with single private API using Public Hosted Zone  |
| [SamplePublicZoneStack](lib/sample-public-zone-stack.js)   | Stack with three private APIs using Public Hosted Zone  |
| [SamplePrivateZoneStack](lib/sample-private-zone-stack.js) | Stack with three private APIs using Private Hosted Zone |

## Deployment Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ```bash
    git clone https://github.com/aws-samples/apigateway-custom-domain-for-private-api-cdk
    ```
2. Change directory to the pattern directory its source code folder:
    ```bash
    cd apigateway-custom-domain-for-private-api-cdk
    ```
3. Install the dependencies:
    ```bash
    npm install -g aws-cdk
    npm install
    ```
   
4. Edit `lib/sample-stack.js` and adjust the following constants according to your environment:

   ```javascript
   const HOSTED_ZONE_ID = 'Z0000000XXXXXXXXXXXX';
   const HOSTED_ZONE_NAME = 'domain.com';
   const SERVICE_FQDN = 'api.domain.com';
   ```
5. To deploy from the command line use the following:
    ```bash
    cdk deploy --require-approval never --concurrency 10
    ```

## Testing
Testing can be done using an EC2 instance to invoke the private API:
1. Connect to the EC2 instance using SSM
    ```bash
    aws ssm start-session --target <INSTANCE_ID>
    ```
   _Note: It may take a few minutes for the instance to be available._


2. Invoke the API FQDN using cURL
    ```bash
    curl https://api.domain.com/hello
    ```

## Cleanup
 
1. From the command line, use the following in the source folder
    ```bash
    cdk destroy
    ```
2. Confirm the removal and wait for the resource deletion to complete.


## Construct Definition
### Constructor Props
Route53 Public Hosted Zone

| Name        | Type                                                                                                             | Required | Description                                          |
|-------------|------------------------------------------------------------------------------------------------------------------|----------|------------------------------------------------------|
| publicZone  | [HostedZone](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_route53.HostedZone.html)                | Yes      | A Public Hosted Zone where DNS entry will be created |
| serviceFqdn | string                                                                                                           | Yes      | FQDN for the service (e.g.: api.domain.com)          |
| certificate | [ICertificate](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_certificatemanager.ICertificate.html) | No       | Certificate protecting the service                   |
| vpc         | [IVpc](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.IVpc.html)                                | No       | A VPC in which the Construct will be deployed        |

Route53 Private Hosted Zone

| Name              | Type                                                                                                             | Required | Description                                                                                    |
|-------------------|------------------------------------------------------------------------------------------------------------------|----------|------------------------------------------------------------------------------------------------|
| serviceFqdn       | string                                                                                                           | Yes      | FQDN for the service. For example: api.domain.com                                              |
| certificate       | [ICertificate](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_certificatemanager.ICertificate.html) | Yes      | Certificate protecting the service                                                             |
| privateZone       | [HostedZone](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_route53.HostedZone.html)                | No       | A Private Hosted Zone where DNS entry will be created                                          |
| privateZoneDomain | string                                                                                                           | No*      | Hosted Zone domain (e.g.: api.domain.com). _*Property required if privateZone is not provided_ |
| vpc               | [IVpc](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.IVpc.html)                                | No       | A VPC in which the Construct will be deployed                                                  |

---
### Properties

| Name                       | Type                                                                                                                               |
|----------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| zone                       | [HostedZone](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_route53.HostedZone.html)                                  |
| certificate                | [ICertificate](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_certificatemanager.ICertificate.html)                   |
| vpc                        | [IVpc](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.IVpc.html)                                                  |_
| vpcEndpoint                | [InterfaceVpcEndpoint](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.InterfaceVpcEndpoint.html)                  |_
| nlb                        | [NetworkLoadBalancer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_elasticloadbalancingv2.NetworkLoadBalancer.html) |
| apigwDomain                | [DomainName](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.DomainName.html)                               |
| apigwDefaultResourcePolicy | [Policy](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Policy.html)                                              |

---
### Methods
| Name                            | Description                                             |
|---------------------------------|---------------------------------------------------------|
| createPrivateApi(id, props)     | Create a private API in API Gateway                     |
| configureApiMapping(api, props) | Configure an API mapping at API Gateways' custom domain |

#### createPrivateApi(id, props)
Parameters
- **id** `string`
- **props** [RestApi Construct Props](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.RestApi.html#construct-props)

Returns
- [RestApi](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.RestApi.html)
---

#### configureApiMapping(api, props)
Parameters
- **api** [RestApi](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.RestApi.html)
- **props** [BasePathMapping Construct Props](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.BasePathMapping.html#construct-props)

Returns
- [BasePathMapping](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.BasePathMapping.html)

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
