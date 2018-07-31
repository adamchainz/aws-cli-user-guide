# Specifying Parameter Values for the AWS Command Line Interface<a name="cli-using-param"></a>

Many parameters are simple string or numeric values, such as the key pair name `my-key-pair` in the following example: 

```
$ aws ec2 create-key-pair --key-name my-key-pair
```

Strings without any space characters may be quoted or unquoted\. However, strings that include one or more space characters must be quoted\. Use a single quote \('\) in Linux, macOS, or Unix and Windows PowerShell, or use a double quote \("\) in the Windows command prompt, as shown in the following examples\. 

**Windows PowerShell, Linux, macOS, or Unix**

```
$ aws ec2 create-key-pair --key-name 'my key pair'
```

**Windows Command Processor**

```
> aws ec2 create-key-pair --key-name "my key pair"
```

You can also use an equals sign instead of a space\. This is typically only necessary if the value of the parameter starts with a hyphen:

```
$ aws ec2 delete-key-pair --key-name=-mykey
```


+ [Common Parameter Types](#parameter-types)
+ [Using JSON for Parameters](#cli-using-param-json)
+ [Quoting Strings](#quoting-strings)
+ [Loading Parameters from a URI](#cli-using-param-uri)

## Common Parameter Types<a name="parameter-types"></a>

This section describes some of the common parameter types and the format that the services expect them to conform to\. If you are having trouble with the formatting of a parameter for a specific command, check the manual by typing **help** after the command name, for example: 

```
$ aws ec2 describe-spot-price-history help
```

The help for each subcommand describes its function, options, output, and examples\. The options section includes the name and description of each option with the option's parameter type in parentheses\. 

 **String** – String parameters can contain alphanumeric characters, symbols, and whitespace from the ASCII character set\. Strings that contain whitespace must be surrounded by quotes\. Use of symbols and whitespace other than the standard space character is not recommended and may cause issues when using the AWS CLI\. 

Some string parameters can accept binary data from a file\. See  for an example\. 

 **Timestamp** – Timestamps are formatted per the ISO 8601 standard\. These are sometimes referred to as "DateTime" or "Date" type parameters\. 

```
$ aws ec2 describe-spot-price-history --start-time 2014-10-13T19:00:00Z
```

Acceptable formats include:

+ YYYY\-MM\-DDThh:mm:ss\.sssTZD \(UTC\), e\.g\., 2014\-10\-01T20:30:00\.000Z

+ YYYY\-MM\-DDThh:mm:ss\.sssTZD \(with offset\), e\.g\., 2014\-10\-01T12:30:00\.000\-08:00

+ YYYY\-MM\-DD, e\.g\., 2014\-10\-01

+ Unix time in seconds, e\.g\. 1412195400

 **List** – One or more strings separated by spaces\. 

```
$ aws ec2 describe-spot-price-history --instance-types m1.xlarge m1.medium
```

 **Boolean** – Binary flag that turns an option on or off\. For example, `ec2 describe-spot-price-history` has a boolean dry\-run parameter that, when specified, validates the command against the service without actually running a query\. 

```
$ aws ec2 describe-spot-price-history --dry-run
```

The output indicates whether the command was well formed or not\. This command also includes a no\-dry\-run version of the parameter that can be used to explicitly indicate that the command should be run normally, although including it is not necessary as this is the default behavior\. 

 **Integer** – An unsigned whole number\. 

```
$ aws ec2 describe-spot-price-history --max-items 5
```

 **Blob** – Binary object\. Blob parameters take a path to a local file that contains binary data\. The path should not contain any protocol identifier such as `http://` or `file://`\. 

The `--body` parameter for `aws s3api put-object` is a blob: 

```
$  aws s3api put-object --bucket my-bucket --key testimage.png --body /tmp/image.png
```

 **Map** – A sequence of key value pairs specified in JSON or shorthand syntax\. The following example reads an item from a DynamoDB table named *my\-table* with a map parameter, `--key`\. The parameter specifies the primary key named *id* with a number value of *1* in a nested JSON structure\. 

```
$ aws dynamodb get-item --table-name my-table --key '{"id": {"N":"1"}}'
{
    "Item": {
        "name": {
            "S": "John"
        },
        "id": {
            "N": "1"
        }
    }
}
```

The next section covers JSON arguments in more detail\. 

## Using JSON for Parameters<a name="cli-using-param-json"></a>

JSON is useful for specifying complex command line parameters\. For example, the following command will list all EC2 instances that have an instance type of `m1.small` or `m1.medium` that are also in the `us-west-2c` Availability Zone\. 

```
$ aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro,m1.medium" "Name=availability-zone,Values=us-west-2c"
```

The following example specifies the equivalent list of filters in a JSON array\. Square brackets are used to create an array of JSON objects separated by commas\. Each object is a comma separated list of key\-value pairs \("Name" and "Values" are both keys in this instance\)\. 

Note that value to the right of the "Values" key is itself an array\. This is required, even if the array contains only one value string\. 

```
[
  {
    "Name": "instance-type",
    "Values": ["t2.micro", "m1.medium"]
  },
  {
    "Name": "availability-zone",
    "Values": ["us-west-2c"]
  }
]
```

The outermost brackets, on the other hand, are only required if more than one filter is specified\. A single filter version of the above command, formatted in JSON, looks like this: 

```
$ aws ec2 describe-instances --filters '{"Name": "instance-type", "Values": ["t2.micro", "m1.medium"]}'
```

Some operations require data to be formatted as JSON\. For example, to pass parameters to the `--block-device-mappings` parameter in the `ec2 run-instances` command, you need to format the block device information as JSON\. 

This example shows the JSON to specify a single 20 GiB Elastic Block Store device to be mapped at `/dev/sdb` on the launching instance\. 

```
{
  "DeviceName": "/dev/sdb",
  "Ebs": {
    "VolumeSize": 20,
    "DeleteOnTermination": false,
    "VolumeType": "standard"
  }
}
```

To attach multiple devices, list the objects in an array like in the next example\. 

```
[
  {
    "DeviceName": "/dev/sdb",
    "Ebs": {
      "VolumeSize": 20,
      "DeleteOnTermination": false,
      "VolumeType": "standard"
    }
  },
  {
    "DeviceName": "/dev/sdc",
    "Ebs": {
      "VolumeSize": 10,
      "DeleteOnTermination": true,
      "VolumeType": "standard"
    }
  }
]
```

You can either enter the JSON directly on the command line \(see [Quoting Strings](#quoting-strings)\), or save it to a file that is referenced from the command line \(see \)\. 

When passing in large blocks of data, you might find it easier to save the JSON to a file and reference it from the command line\. JSON data in a file is easier to read, edit, and share with others\. This technique is described in the next section\. 

For more information about JSON, see [Wikipedia \- JSON](http://en.wikipedia.org/wiki/JSON) and [RFC4627 \- The application/json Media Type for JSON](http://tools.ietf.org/html/rfc4627)\. 

## Quoting Strings<a name="quoting-strings"></a>

The way you enter JSON\-formatted parameters on the command line differs depending upon your operating system\. Linux, macOS, or Unix and Windows PowerShell use the single quote \('\) to enclose the JSON data structure, as in the following example: 

```
$ aws ec2 run-instances --image-id ami-05355a6c --block-device-mappings '[{"DeviceName":"/dev/sdb","Ebs":{"VolumeSize":20,"DeleteOnTermination":false,"VolumeType":"standard"}}]'
```

The Windows command prompt, on the other hand, uses the double quote \("\) to enclose the JSON data structure\. In addition, a backslash \(\\\) escape character is required for each double quote \("\) within the JSON data structure itself, as in the following example: 

```
> aws ec2 run-instances --image-id ami-05355a6c --block-device-mappings "[{\"DeviceName\":\"/dev/sdb\",\"Ebs\":{\"VolumeSize\":20,\"DeleteOnTermination\":false,\"VolumeType\":\"standard\"}}]"
```

Windows PowerShell requires a single quote \('\) to enclose the JSON data structure, as well as a backslash \(\\\) to escape each double quote \("\) within the JSON structure, as in the following example: 

```
> aws ec2 run-instances --image-id ami-05355a6c --block-device-mappings '[{\"DeviceName\":\"/dev/sdb\",\"Ebs\":{\"VolumeSize\":20,\"DeleteOnTermination\":false,\"VolumeType\":\"standard\"}}]'
```

If the value of a parameter is itself a JSON document, escape the quotes on the embedded JSON document\. For example, the `attribute` parameter for `aws sqs create-queue` can take a `RedrivePolicy` key\. The value of `RedrivePolicy` is a JSON document, which must be escaped:

```
$ aws sqs create-queue --queue-name my-queue --attributes '{ "RedrivePolicy":"{\"deadLetterTargetArn\":\"arn:aws:sqs:us-west-2:0123456789012:deadletter\", \"maxReceiveCount\":\"5\"}"}'
```

## Loading Parameters from a URI<a name="cli-using-param-uri"></a>

Sometimes it's convenient to load a parameter value from a local or remote file, in these cases the below URI formats can be used to provide the path to the file\. This behaviour is disabled for some parameters on commands where the underlying API expects a URI, for example CloudFormation Template URI's\. Also the behaviour can be disabled overall by adding `cli_follow_urlparam = false` to the configuration file\.

### Loading Parameters from a local file<a name="cli-using-param-file"></a>

This can be useful to avoid the need to escape JSON strings at the command line\. Load parameters from a local file by providing the path to the file using the `file://` prefix, as in the following examples\.

**Linux, macOS, or Unix**

```
// Read from a file in the current directory
$ aws ec2 describe-instances --filters file://filter.json

// Read from a file in /tmp
$ aws ec2 describe-instances --filters file:///tmp/filter.json
```

**Windows**

```
// Read from a file in C:\temp
> aws ec2 describe-instances --filters file://C:\temp\filter.json
```

The `file://` prefix option supports Unix\-style expansions including '\~/', '\./', and '\.\./'\. On Windows, the '\~/' expression expands to your user directory, stored in the %USERPROFILE% environment variable\. For example, on Windows 7 you would typically have a user directory under `C:\Users\User Name\`\.

JSON documents that are provided as the value of a parameter key must still be escaped:

```
$ aws sqs create-queue --queue-name my-queue --attributes file://attributes.json
```

**attributes\.json**

```
{
  "RedrivePolicy":"{\"deadLetterTargetArn\":\"arn:aws:sqs:us-west-2:0123456789012:deadletter\", \"maxReceiveCount\":\"5\"}"
}
```

### Binary Files<a name="cli-using-param-file-binary"></a>

For commands that take binary data as a parameter, specify that the data is binary content by using the `fileb://` prefix\. Commands that accept binary data include: 

+  **`aws ec2 run-instances`** – `--user-data` parameter\. 

+  **`aws s3api put-object`** – `--sse-customer-key` parameter\. 

+  **`aws kms decrypt`** – `--ciphertext-blob` parameter\. 

The following example generates a binary 256 bit AES key using a Linux command line tool and then provides it to Amazon S3 to encrypt an uploaded file server\-side: 

```
$ dd if=/dev/urandom bs=1 count=32 > sse.key
32+0 records in
32+0 records out
32 bytes (32 B) copied, 0.000164441 s, 195 kB/s
$ aws s3api put-object --bucket my-bucket --key test.txt --body test.txt --sse-customer-key fileb://sse.key --sse-customer-algorithm AES256
{
    "SSECustomerKeyMD5": "iVg8oWa8sy714+FjtesrJg==",
    "SSECustomerAlgorithm": "AES256",
    "ETag": "\"a6118e84b76cf98bf04bbe14b6045c6c\""
}
```

### Remote Files<a name="cli-using-param-file-remote"></a>

The AWS CLI also supports loading parameters from a file hosted on the Internet with an `http://` or `https://` URL\. The following example references a file in an Amazon S3 bucket\. This allows you to access parameter files from any computer, but requires the file to be stored in a publically accessible location\. 

```
$ aws ec2 run-instances --image-id ami-a13d6891 --block-device-mappings http://my-bucket.s3.amazonaws.com/filename.json
```

In the preceding examples, the `filename.json` file contains the following JSON data\.

```
[
  {
    "DeviceName": "/dev/sdb",
    "Ebs": {
      "VolumeSize": 20,
      "DeleteOnTermination": false,
      "VolumeType": "standard"
    }
  }
]
```

For another example referencing a file containing more complex JSON\-formatted parameters, see \. 
