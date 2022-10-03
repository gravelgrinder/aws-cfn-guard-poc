# aws-cfn-guard-poc
POC showing how to use cfn-guard to implement preventive compliance on your CloudFormation Templates

## AWS Organizations Tag Policies
1. I pre-configured a similar Tagging Policy within AWS Organizations to validate for proper Environment tagging values aginst the following services, S3, EC2 Instances and EC2 Volumes. Review the tagging policie configurations.

1. Lets change the tag value to "DEVELOPMENT" in the `02-CREATE-S3.yaml` CloudFormation template.  Save  your changes.

1. Now lets attempt to create a S3 Bucket from CloudFormation Stack with an invalid tag.
```
aws cloudformation create-stack --stack-name s3-example-invalid-tag --template-body file://02-CREATE-S3.yaml
```
1. Verify the Stack was Rolled Back because of the tag policy not allowing that specific tag value for the "Environment" tag key.


## CloudFormation Guard Demo
1. If you have an existing CloudFormation Template, you can run `cfn-guard rulegen` to help generate a baseline ruleset to work from.  You can then tweak the rules to your more specific needs.
```
cfn-guard rulegen --output output.guard --template 02-CREATE-S3.yaml
```
1. Test your template against the ruleset you just created. Then verify the output of the script is `0` (Successful).
```
cfn-guard validate --data 02-CREATE-S3.yaml --rules output.guard
echo $?
```
1. Now change the Environment tag value in the 02-CREATE-S3.yaml script to the value of "TEST".  Run the validate step again and see the output.  You should see a similar failure as shows below.
```
lwdvin@a483e7078b3f aws-cfn-guard-poc % cfn-guard validate --data 02-CREATE-S3.yaml --rules output.guard
Other
02-CREATE-S3.yaml Status = FAIL
FAILED rules
output.guard/aws_s3_bucket    FAIL
---
Evaluating data 02-CREATE-S3.yaml against rules output.guard
Number of non-compliant resources 1
Resource = s3Bucket {
  Type      = AWS::S3::Bucket
  Rule = aws_s3_bucket {
    ALL {
      Check =  %aws_s3_bucket_resources[*].Properties.Tags EQUALS  [{"Key":"Environment","Value":"DEV"}] {
        ComparisonError {
          Error           = Check was not compliant as property value [Path=/Resources/s3Bucket/Properties/Tags[L:9,C:8] Value=[{"Key":"Environment","Value":"TEST"}]] not equal to value [Path=[L:0,C:0] Value=[{"Key":"Environment","Value":"DEV"}]].
          PropertyPath    = /Resources/s3Bucket/Properties/Tags[L:9,C:8]
          Operator        = EQUAL
          Value           = [{"Key":"Environment","Value":"TEST"}]
          ComparedWith    = [{"Key":"Environment","Value":"DEV"}]
          Code:
                7.      AccessControl: Private
                8.      Tags: 
                9.        - Key: "Environment"
               10.          Value: "TEST"

        }
      }
    }
  }
}
```
1. We want our ruleset to be more specific on what values are allowed so now lets validate the CloudFormation tempate against a more refined ruleset that includes all the environment values (DEV, TEST, QA and PROD).  The CloudFormation template should now be valid.
```
cfn-guard validate --data 02-CREATE-S3.yaml --rules rules/example-policy.guard
```




## Getting Started
### Rulegen
If you have existing templates that you need rules generated for, leverage the `cfn-guard rulegen` utility to auto generate rules.  Here is a sample command.  Use the `--output` parameter to define the ruleset file that will be generated, the `--template` parameter is the input CloudFormation template.
```
cfn-guard rulegen --output output.guard --template 01-s3-fail.yaml
```

## Unit Testing
The following command will unit test the `example-policy.guard` policy against the units tests defined in the `example-policy-test.yaml` file.
```
cfn-guard test -r rules/example-policy.guard -t rules/example-policy-test.yaml
```

## Cleanup
1. Run the following script to cleanup resources.
```
rm output.guard
```

1. Delete `s3-example-invalid-tag` CloudFormation Stack
```
aws cloudformation delete-stack --stack-name s3-example-invalid-tag
```

## Helpful Resources
AWS Organizations - Tagging Policies & SCPs
* [Implement AWS resource tagging strategy using AWS Tag Policies and Service Control Policies (SCPs)](https://aws.amazon.com/blogs/mt/implement-aws-resource-tagging-strategy-using-aws-tag-policies-and-service-control-policies-scps/)

AWS CloudFormation Guard
* [AWS CloudFormation Guard User Guide](https://docs.aws.amazon.com/cfn-guard/latest/ug/what-is-guard.html)
* [AWS CloudFormation Workshop](https://catalog.workshops.aws/cfn101/en-US)


## Questions & Comments
If you have any questions or comments on the demo please reach out to me [Devin Lewis - AWS Solutions Architect](mailto:lwdvin@amazon.com?subject=AWS%2FCloudFormation%20Guard%20Demo%20%28aws-cfn-guard-poc%29)

Of if you would like to provide personal feedback to me please click [Here](https://feedback.aws.amazon.com/?ea=lwdvin&fn=Devin&ln=Lewis)