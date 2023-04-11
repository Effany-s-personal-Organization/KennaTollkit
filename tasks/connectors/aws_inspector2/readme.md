# AWS Inspector V2 Connector Task

This task brings in asset and vulnerability data from AWS Inspector V2.

## Running the task

See the main toolkit README for instructions on running tasks. For this task, if you leave off the Kenna API Key and Kenna Connector ID, the task will create a json file in the default or specified output directory. You can review the file before attempting to upload to the Kenna API.

### Recommended Steps:

1. Run with AWS keys only. You can provide AWS credentials and configuration through [shared ini files, environment variables](https://docs.aws.amazon.com/sdkref/latest/guide/creds-config-files.html),

```
docker run -v ~/.aws:/root/.aws --env AWS_REGION=us-east-1 --env AWS_PROFILE=example_profile --rm -it toolkit:latest task=aws_inspector2
```

...or by directly providing them to the task as shown below.

```
docker run --rm -it toolkit:latest task=aws_inspector2 aws_access_key_id=AKIAIOSFODNN7EXAMPLE aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY aws_regions=us-east-1,us-east-2
```

2. Review output for expected data.
3. Create a "Kenna Data Importer" Connector in Kenna and give it a meaningful name, like "AWS Inspector V2 KDI."

If you're doing any scanning of ECR container images, you'll need to have Kenna Support set a custom locator order for your KDI connector similar to: `image_locator, ec2_locator, mac_address_locator, netbios_locator, external_ip_address_locator, hostname_locator, url_locator, file_locator, fqdn_locator, ip_address_locator, external_id_locator, database_locator, application_locator`
4. Manually upload the JSON output from Step 1 to the Kenna Data Connector.
5. Review resulting data and diagnose any failure.
6. Click on the name of the KDI Connector to get the connector ID.
7. Run the task with AWS credentials and your Kenna API key & connector ID.

```
docker run -v ~/.aws:/root/.aws --env AWS_PROFILE=example_profile --rm -it toolkit:latest task=aws_inspector2 aws_regions=us-east-1,us-east-2 kenna_api_key=$KENNA_API_KEY kenna_connector_id=12345
```

### Limitations:

1. The AWS SDK provides the ability to move away from using AWS static keys, as it is a security risk. Using the ability to assume arn roles will give the task access using rolling AWS keys, but they are not yet implemented in the task.
Use :

```
puts "Using role: " + role_arn
role_credentials = Aws::AssumeRoleCredentials.new(
  client: Aws::STS::Client.new(region: region),
  role_arn: role_arn,
  role_session_name: "kenna-session"
)
puts region
inspector = Aws::Inspector2::Client.new(region: region)
```

2. The task currently only handles package vulnerabilities, not code vulnerabilities (in AWS Lambda) or network reachability findings.
