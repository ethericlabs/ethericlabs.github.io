# [Amazon Web Services](../README.md) / [Bedrock](README.md) / Getting started


# Summary

How to
- create an AWS user in a Bedrock group with access keys and a short-term Bedrock API key
- set up a disposable development environment on a cheap Google Cloud Platform Compute Engine instance
- send requests to the Bedrock API with cURL, AWS CLI and Python

You'll need
- a way to securely generate and store sensitive data
- a way to create a disposable development virtual machine
- an AWS user with the appropriate permissions

I chose these ingredients, you should substitute according to your tastes
- [Enpass](https://enpass.io)
- Google Cloud Platform [Compute Engine](https://cloud.google.com/products/compute)
- I explain how to use the AWS `root` user to create a user group and user with access to Bedrock


# References

Amazon Bedrock Workshop
- (https://catalog.us-east-1.prod.workshops.aws/amazon-bedrock/en-US)

Amazon Bedrock user guide
- https://docs.aws.amazon.com/bedrock/latest/userguide

Documents and user guides
- https://docs.aws.amazon.com/bedrock


# Caveat

AWS documents, products, features and processes change.


# Content

1 Amazon Web Services

- 1.1 Create an AWS user group
- 1.2 Create an AWS user
- 1.3 Create an access key for the user `bedrock`

2 Development machine

- 2.1 Create an OpenSSH key pair
- 2.2 Create a development machine
- 2.3 Connect to the development machine
- 2.4 Use cURL to list foundation models

3 Bedrock commands in the AWS CLI

- 3.1 Install the AWS CLI
- 3.2 Authenticate as an IAM user with AWS CLI
- 3.3 Use AWS CLI Bedrock command to list foundation models

4 Bedrock command in Python

- 4.1 Install `boto3`
- 4.2 Use a Python script to list foundation models
- 4.3 Use a Python script to interact with InvokeModel
- 4.4 Use a Python script to interact with Converse

5 Tear down

- 5.1 Amazon Web Services
- 5.2 Google Cloud Platform
- 5.3 Laptop


# 1 Amazon Web Services

Assuming you only have a root user account, set up a new user group and user with appropriate permissions


## 1.1 Create an AWS user group

Log in as the AWS `root` user in one of the regions recommended by the AWS workshop

- US-East-1 (North Virginia)
- US-West-2 (Oregon)

IAM > Groups

- https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/groups

Click 'Create group'

- User group name: `bedrock-users`
- Attach permissions: `AmazonBedrockLimitedAccess`

Click 'Create user group'

Log out


## 1.2 Create an AWS user

Generate and record a strong password

```
W(@yX;REDACTED;?x28xe4
```

Log in as the root user

- https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/users

Click 'Create user'

- User name: `bedrock`
- Provide user access to the AWS Management Console: `Yes`
- Console password: Custom password `W(@yX;REDACTED;?x28xe4`
- Users must create a new password at next login: `No`

Click 'Next'

- Permissions options: `Add user to group`
- User groups: `bedrock-users`

Click 'Next'

Click 'Create user'

Record the authentication URL: https://000000000000.signin.aws.amazon.com/console

Click 'Return to users list'

A warning about not viewing the password is shown

Click 'Continue'

Log out


## 1.3 Create an access key for the user `bedrock`

Log in as the root user

IAM > Users > `bedrock`

- https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/users/details/bedrock?section=permissions

Click 'Create access key'

- Use case: `Command line interface (CLI)`

Confirmation: `Yes`

Click 'Next'

- Description tag value: `bedrock-aws-cli`

Click `Create access key`

Record the access key and secret access key values

- Access key: `AAAAAAAAAAAAAAAA`
- Secret access key: `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`
- Default region name: `us-east-1`

Click 'Done'

A warning is shown: Continue without viewing or downloading

Click 'Continue'

Log out


## 1.4 Create a Bedrock API key for the user `bedrock`

Log in as the `bedrock` user

- Authentication URL: https://000000000000.signin.aws.amazon.com/console
- IAM username: `bedrock`
- Password: `W(@yX;REDACTED;?x28xe4`

Amazon Bedrock > API keys

- https://us-east-1.console.aws.amazon.com/bedrock/home?region=us-east-1#/api-keys?tab=short-term

Make sure you're logged in with the `bedrock` account, as API keys inherit the permissions of the current user

Click 'Generate short-term API keys'

The key value is much, much longer than the text area it's displayed in :

```text
bedrock-api-key-YmVkcm9jay5hbWF6b25hd3MuY29tLz9BY3Rpb249Q2FsbFdpdGhCZWFyZXJUb2tlbiZYLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFTSUFYSEJRSk9WTldOT1JRR0FCJTJGMjAyNTExMTElMkZ1cy1lYXN0LTElMkZiZWRyb2NrJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTExMTFUMTQ1NTEyWiZYLUFtei1FeHBpcmVzPTQzMjAwJlgtQW16LVNlY3VyaXR5LVRva2VuPUlRb0piM0pwWjJsdVgyVmpFRmNhQ1hWekxXVmhjM1F0TVNKSE1FVUNJRk9NUFcycUFQOUpFNTdQUFFaekZrclFKR24xamV6UXViUEdzVURIU0Jzc0FpRUFoMmZrNnRTMGRlcXhmS3d4MjlwbGM0cVpSOG43MHYlMkJNSktxWnhRUk5mYThxdmdNSUlCQURHZ3cwT1RZeE56QXdNRFU0TlRFaUROeG5ydUM4cmxPNGQwSjZLQ3FiQTc2YW02cDN4SThjSkdReGJrT2VsNjBjMlIySHclMkZiNWZXNmFZYm0zU3Z6NzBIUnp2NlZXcGhnQlV1Q3JlNmglMkJrWVpDRGtuNjdlQkJhJTJCQ3VSVVUyR1N0JTJCOXJ4dTV4endXWG5YVmZtZzZnYUEyenNSUmw0WlRXeDJCRkdnY0J5R0pmTWNzT0xacUglMkIycSUyQkYzdDAlMkJRTVpBNXU3NVY4R0x6dHJza0U4Sko3Qnc3JTJCbHd6VFE1VFZmeURoWEdkcSUyRnY0NjljUkVyd2Q3VUloWm5LN0RRVksxUmNsTW1uR1k2N2U4VyUyQndFMnRGViUyQjN0VFBFbFZ2MHdnZDVzdUNoNkdLcXFqV2hVOHJCdHp3RkVvb2ZlSFN5Y1dNMndZNkUwJTJCa0NlSzBaNFQ1QTkwQzFxYlEyZkdlYkkxdXd2RXlYZzdTdXdzeTFLd1VEUGRCb0lpM2NYVjY5dms2UXVuV2lqdkFjbWpKZkdMem1nZk13cWsyUG1LUGFIQmk3ekRMWm1mR3QlMkJ5TFVNRmFQbHdXaDc3VUhTT2RNYWxSV1JqODhtRzdiYWRyakxTbmc5c1R1aHZLOHpMNlZzZDVSNktlM2o1YTBXUXNFNkVtT3JhRFlFdFpnMlVia1k3TnBheUJnR1dYUktVcHdWSWRCcW0lMkZKJTJCSllCZiUyQnVVWVUlMkZxN1E5QXFBSTN1NEJma3RlJTJGaTlFcVlnZCUyQjR0QkYzVUFkUUVwdTE0WWhxNVM4MVhqRGttODNJQmpyZUF2dURUbFVBZDJ4VmRURVhBZ0Zyd29rTWlneGVnWXYyTnVjS0ROYVR3UWY1UlMlMkYzU0F0TTJCYUR5UHAlMkZqUTFoZXdBTTJId2s1Rjk1dHpRUCUyRjQzM2t3VzI3UFZRU2g3NGFCJTJCNmxzd244SWhzMkpCeVpYcko0R3V1d2JIRXJGbHZLaFZ4WFpvJTJCUUt3MWN6VElwbzFwMHRCSUlEaEc3OHpGVEdSVzBVWkVhUEtRQzI4UWZlaW1EJTJCd2ElMkJOZkclMkZadU1yRU02Y1JXNnhUMHkwc0dDUTdETjg0bXBocDk4bHhGVDVGV2tLNlFFYnNDakJMQ1g0MTRHaTFUMFNpb3ZTaG42amVxN0hEa1VDN0pJaHBJMzc5VyUyQnpNQUElMkZ1ZnNIMVp5cE9ObWsxOVRaJTJCSG04JTJGUDYydFoxV0MzTTlFcFVJMlJsVkJHWW9NV1hXaG5iWXB0cXFvQXRKNFgxc0NKN05CSEJ6NUpnNXRmZnp4WnozaFN2b2h1N05CZndZQlN1b2FHVDRsVmhaQ2NZU0Z5bFhHeDNxNCUyQklkczMzTGRQTkFOTGtUdCUyRm4lMkJTR2FqOGtWJTJGTUZ3bUdKb0hoR21lZTRlbHAwYkxTM2pVN1B3VWFIQXRaS2x6WjhHJlgtQW16LVNpZ25hdHVyZT1kYWIzODRlNTA1YjE4OGY1MGYyZGVkYmQ3OWU5MzU5MGM2Njg2NGJmZDFkYzkzZjY4ZDIwYTljYzJlZWEwZTJjJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZWZXJzaW9uPTE=
```
Click 'Close'

Log out


# 2 Development machine

To avoid installing and configuring on your actual laptop, start up a Google Cloud Platform Compute engine instance.


## 2.1 Create an OpenSSH key pair

```bash
[you@laptop ~]$ cd credentials/cloud.google.com
```

```bash
[you@laptop cloud.google.com]$ ssh-keygen \
  -t rsa \
  -b 4096 \
  -f bedrock \
  -C $( whoami ) \
  -N ''
```
```
Generating public/private rsa key pair.
Your identification has been saved in bedrock
Your public key has been saved in bedrock.pub
The key fingerprint is:
SHA256:8Hv/0/POJZzJe3W5E6WICpi2OBe+WnGOB0bfJ+JjSRc neil
The key's randomart image is:
+---[RSA 4096]----+
|                 |
|                 |
|    . . E        |
|   . . + .      .|
|    +o= S .. . .o|
|   .=O.+ +. .o.=o|
|   +ooB....   *.*|
|  o.+o ... .  .B+|
|  .+..      ..o+B|
+----[SHA256]-----+
```

```bash 
[you@laptop cloud.google.com]$ cat bedrock.pub	
```
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDMnu+hlBB7ow29NfgnBRnTwdrC8XBwhKDIJPT55goCIVR6DzRM6V/zRquAnbm5bzge/7dIbnkHlTNPNFhRLGKOdgUrYUGwTLIiy5IXytjaHJrOYa2SuqrtirtmWWRcWT3CnY4H9gMQ2ZdtavK/td0CsVZZ7mbm02sBoDM245fT/l3410AxBAyYV5ANsdkb3p9EluSXaenneHk0b15H4+D7nYYYhgVqyp+uUaeuc+7vnbUqdBJh9/gsGmTRU1KA1SnhNBJco1uC9bbrOhGgsjTtbaGf78Fv6m32Mtobn3QRNjIbkD9Hbi0S10OJh1Wupq6ElzoDcdvxcBEYYulSKwftXWuGtRLEEGnAIa2ys+7/J6cTKAtXzVpyF3XX8TWKXKv1xgVC7gTcxvinZq+8Gs3kwDbRRgU9TWlw+fGRYMXzuUFmuHzFWTSO7eLV5BSdj42CKgrVTL1xNZiEjY9/BCrLaqocK5JsAGcPQz+IdomtVq5Buh4xam6OZR81QvHQ3TQpgWLLe54VFnALAkL1D3VZ+VbMS4S5khEQs7q7sqJgBlCjLdxok3f1AohbcxTq60W44iCqCAuTLJ6qHlt6IT1GQBTQwq8kW5EVXWXdnJV2/sltYVpD3fHBwaRQhb2PlZpyFIUJTuEss5zHrO2VU1o1p8+36PTuruVxG6FZDiCPnQ== neil
```

## 2.2 Create a development machine

Create an instance in your default project at https://console.cloud.google.com/compute/instancesAdd

Google Cloud Platform > Compute Engine > Create an instance


### Machine configuration

- Name: `bedrock`
- Region: `us-east4 (Northern Virginia)`
- Zone: `us-east4-a`
- Type: `General purpose E2 e2-standard-4 (4 vCPU, 2 core, 16Gb memory)`

For reference, this costs about USD 0.15 per hour


### OS and storage

- Operating system: `CentOS`
- Version: `CentOS Stream 10`
- Boot disk type: `Balanced persistent disk`
- Size (Gb): `20`


### Data protection

- Backups: `No backups`


### Networking

No changes needed (1 default interface)


### Observability

No changes needed


### Security

Manage access > Add manually generated SSH keys

- Add item: Paste your public key


### Advanced

No changes needed


Finally, click 'Create' and copy the external IP address when it's available


## 2.3 Connect to the development machine

Use the SSH key pair to authenticate your SSH session at the machine's external IP address

```
[you@laptop cloud.google.com]$ BEDROCK_DEV=11.22.33.44
```

```
[you@laptop cloud.google.com]$ BEDROCK_KEY=bedrock
```

```
[you@laptop cloud.google.com]$ ssh -i ${BEDROCK_KEY} ${BEDROCK_DEV}
```

```
The authenticity of host '11.22.33.44 (11.22.33.44)' can't be established.
ED25519 key fingerprint is SHA256:9XTlpuHG5tSGwwjMn4ApnkN832CDeVlmm+OG7TMvk7M.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '11.22.33.44' (ED25519) to the list of known hosts.
[neil@bedrock ~]$
```


## 2.4 Use cURL to list foundation models

After making the API key available as an environment variable, make a request to the API and format the JSON response

```bash
((bedrock))[neil@bedrock ~]$ BEDROCK_API_KEY="bedrock-api-key-YmVkcm9jay5hbWF6b25hd3MuY29tLz9BY3Rpb249Q2FsbFdpdGhCZWFyZXJUb2tlbiZYLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFTSUFYSEJRSk9WTldOT1JRR0FCJTJGMjAyNTExMTElMkZ1cy1lYXN0LTElMkZiZWRyb2NrJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTExMTFUMTQ1NTEyWiZYLUFtei1FeHBpcmVzPTQzMjAwJlgtQW16LVNlY3VyaXR5LVRva2VuPUlRb0piM0pwWjJsdVgyVmpFRmNhQ1hWekxXVmhjM1F0TVNKSE1FVUNJRk9NUFcycUFQOUpFNTdQUFFaekZrclFKR24xamV6UXViUEdzVURIU0Jzc0FpRUFoMmZrNnRTMGRlcXhmS3d4MjlwbGM0cVpSOG43MHYlMkJNSktxWnhRUk5mYThxdmdNSUlCQURHZ3cwT1RZeE56QXdNRFU0TlRFaUROeG5ydUM4cmxPNGQwSjZLQ3FiQTc2YW02cDN4SThjSkdReGJrT2VsNjBjMlIySHclMkZiNWZXNmFZYm0zU3Z6NzBIUnp2NlZXcGhnQlV1Q3JlNmglMkJrWVpDRGtuNjdlQkJhJTJCQ3VSVVUyR1N0JTJCOXJ4dTV4endXWG5YVmZtZzZnYUEyenNSUmw0WlRXeDJCRkdnY0J5R0pmTWNzT0xacUglMkIycSUyQkYzdDAlMkJRTVpBNXU3NVY4R0x6dHJza0U4Sko3Qnc3JTJCbHd6VFE1VFZmeURoWEdkcSUyRnY0NjljUkVyd2Q3VUloWm5LN0RRVksxUmNsTW1uR1k2N2U4VyUyQndFMnRGViUyQjN0VFBFbFZ2MHdnZDVzdUNoNkdLcXFqV2hVOHJCdHp3RkVvb2ZlSFN5Y1dNMndZNkUwJTJCa0NlSzBaNFQ1QTkwQzFxYlEyZkdlYkkxdXd2RXlYZzdTdXdzeTFLd1VEUGRCb0lpM2NYVjY5dms2UXVuV2lqdkFjbWpKZkdMem1nZk13cWsyUG1LUGFIQmk3ekRMWm1mR3QlMkJ5TFVNRmFQbHdXaDc3VUhTT2RNYWxSV1JqODhtRzdiYWRyakxTbmc5c1R1aHZLOHpMNlZzZDVSNktlM2o1YTBXUXNFNkVtT3JhRFlFdFpnMlVia1k3TnBheUJnR1dYUktVcHdWSWRCcW0lMkZKJTJCSllCZiUyQnVVWVUlMkZxN1E5QXFBSTN1NEJma3RlJTJGaTlFcVlnZCUyQjR0QkYzVUFkUUVwdTE0WWhxNVM4MVhqRGttODNJQmpyZUF2dURUbFVBZDJ4VmRURVhBZ0Zyd29rTWlneGVnWXYyTnVjS0ROYVR3UWY1UlMlMkYzU0F0TTJCYUR5UHAlMkZqUTFoZXdBTTJId2s1Rjk1dHpRUCUyRjQzM2t3VzI3UFZRU2g3NGFCJTJCNmxzd244SWhzMkpCeVpYcko0R3V1d2JIRXJGbHZLaFZ4WFpvJTJCUUt3MWN6VElwbzFwMHRCSUlEaEc3OHpGVEdSVzBVWkVhUEtRQzI4UWZlaW1EJTJCd2ElMkJOZkclMkZadU1yRU02Y1JXNnhUMHkwc0dDUTdETjg0bXBocDk4bHhGVDVGV2tLNlFFYnNDakJMQ1g0MTRHaTFUMFNpb3ZTaG42amVxN0hEa1VDN0pJaHBJMzc5VyUyQnpNQUElMkZ1ZnNIMVp5cE9ObWsxOVRaJTJCSG04JTJGUDYydFoxV0MzTTlFcFVJMlJsVkJHWW9NV1hXaG5iWXB0cXFvQXRKNFgxc0NKN05CSEJ6NUpnNXRmZnp4WnozaFN2b2h1N05CZndZQlN1b2FHVDRsVmhaQ2NZU0Z5bFhHeDNxNCUyQklkczMzTGRQTkFOTGtUdCUyRm4lMkJTR2FqOGtWJTJGTUZ3bUdKb0hoR21lZTRlbHAwYkxTM2pVN1B3VWFIQXRaS2x6WjhHJlgtQW16LVNpZ25hdHVyZT1kYWIzODRlNTA1YjE4OGY1MGYyZGVkYmQ3OWU5MzU5MGM2Njg2NGJmZDFkYzkzZjY4ZDIwYTljYzJlZWEwZTJjJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZWZXJzaW9uPTE="
```

```
((bedrock))[neil@bedrock ~]$ curl -X POST "https://bedrock-runtime.us-east-1.amazonaws.com/model/us.anthropic.claude-3-5-haiku-20241022-v1:0/converse" \
  --silent \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${BEDROCK_API_KEY}" \
  -d '{ "messages": [ { "role": "user", "content": [ { "text": "Hello" } ] } ] }' \
  | python -mjson.tool 
```

Sometimes it works ... but be ready to receive a message about model use case details. Either way, we'll be using foundation models from other providers for the moment.

```json
{
    "metrics": {
        "latencyMs": 1072
    },
    "output": {
        "message": {
            "content": [
                {
                    "text": "Hi there! How are you doing today? Is there anything I can help you with?"
                }
            ],
            "role": "assistant"
        }
    },
    "stopReason": "end_turn",
    "usage": {
        "cacheReadInputTokenCount": 0,
        "cacheReadInputTokens": 0,
        "cacheWriteInputTokenCount": 0,
        "cacheWriteInputTokens": 0,
        "inputTokens": 8,
        "outputTokens": 21,
        "serverToolUsage": {},
        "totalTokens": 29
    }
}
```

The message about model use case details looks like this

```json
{
    "message": "Model use case details have not been submitted for this account. Fill out the Anthropic use case details form before using the model. If you have already filled out the form, try again in 15 minutes."
}
```


# 3 Bedrock commands in the AWS CLI

## 3.1 Install the AWS CLI

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Confirm that Python meets the requirements

```
[neil@bedrock ~]$ python --version
```

```
Python 3.12.11
```
To use Pip, create and start a Python virtual environment

```
[neil@bedrock ~]$ python -m venv bedrock
[neil@bedrock ~]$ source bedrock/bin/activate
((bedrock))[neil@bedrock ~]$ 
```

```bash
((bedrock))[neil@bedrock ~]$ sudo dnf -y install unzip
```

Create a public key file to verify the downloaded application

```bash
((bedrock))[neil@bedrock ~]$ cat << EOF > awscli-exe-linux-x86_64.zip.pub
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQINBF2Cr7UBEADJZHcgusOJl7ENSyumXh85z0TRV0xJorM2B/JL0kHOyigQluUG
ZMLhENaG0bYatdrKP+3H91lvK050pXwnO/R7fB/FSTouki4ciIx5OuLlnJZIxSzx
PqGl0mkxImLNbGWoi6Lto0LYxqHN2iQtzlwTVmq9733zd3XfcXrZ3+LblHAgEt5G
TfNxEKJ8soPLyWmwDH6HWCnjZ/aIQRBTIQ05uVeEoYxSh6wOai7ss/KveoSNBbYz
gbdzoqI2Y8cgH2nbfgp3DSasaLZEdCSsIsK1u05CinE7k2qZ7KgKAUIcT/cR/grk
C6VwsnDU0OUCideXcQ8WeHutqvgZH1JgKDbznoIzeQHJD238GEu+eKhRHcz8/jeG
94zkcgJOz3KbZGYMiTh277Fvj9zzvZsbMBCedV1BTg3TqgvdX4bdkhf5cH+7NtWO
lrFj6UwAsGukBTAOxC0l/dnSmZhJ7Z1KmEWilro/gOrjtOxqRQutlIqG22TaqoPG
fYVN+en3Zwbt97kcgZDwqbuykNt64oZWc4XKCa3mprEGC3IbJTBFqglXmZ7l9ywG
EEUJYOlb2XrSuPWml39beWdKM8kzr1OjnlOm6+lpTRCBfo0wa9F8YZRhHPAkwKkX
XDeOGpWRj4ohOx0d2GWkyV5xyN14p2tQOCdOODmz80yUTgRpPVQUtOEhXQARAQAB
tCFBV1MgQ0xJIFRlYW0gPGF3cy1jbGlAYW1hem9uLmNvbT6JAlQEEwEIAD4CGwMF
CwkIBwIGFQoJCAsCBBYCAwECHgECF4AWIQT7Xbd/1cEYuAURraimMQrMRnJHXAUC
aGveYQUJDMpiLAAKCRCmMQrMRnJHXKBYD/9Ab0qQdGiO5hObchG8xh8Rpb4Mjyf6
0JrVo6m8GNjNj6BHkSc8fuTQJ/FaEhaQxj3pjZ3GXPrXjIIVChmICLlFuRXYzrXc
Pw0lniybypsZEVai5kO0tCNBCCFuMN9RsmmRG8mf7lC4FSTbUDmxG/QlYK+0IV/l
uJkzxWa+rySkdpm0JdqumjegNRgObdXHAQDWlubWQHWyZyIQ2B4U7AxqSpcdJp6I
S4Zds4wVLd1WE5pquYQ8vS2cNlDm4QNg8wTj58e3lKN47hXHMIb6CHxRnb947oJa
pg189LLPR5koh+EorNkA1wu5mAJtJvy5YMsppy2y/kIjp3lyY6AmPT1posgGk70Z
CmToEZ5rbd7ARExtlh76A0cabMDFlEHDIK8RNUOSRr7L64+KxOUegKBfQHb9dADY
qqiKqpCbKgvtWlds909Ms74JBgr2KwZCSY1HaOxnIr4CY43QRqAq5YHOay/mU+6w
hhmdF18vpyK0vfkvvGresWtSXbag7Hkt3XjaEw76BzxQH21EBDqU8WJVjHgU6ru+
DJTs+SxgJbaT3hb/vyjlw0lK+hFfhWKRwgOXH8vqducF95NRSUxtS4fpqxWVaw3Q
V2OWSjbne99A5EPEySzryFTKbMGwaTlAwMCwYevt4YT6eb7NmFhTx0Fis4TalUs+
j+c7Kg92pDx2uQ==
=OBAt
-----END PGP PUBLIC KEY BLOCK-----
EOF
```

Download the installer

```bash
((bedrock))[neil@bedrock ~]$ curl --silent --remote-name "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
```

Download the installer signature file

```bash
((bedrock))[neil@bedrock ~]$ curl --silent --remote-name "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip.sig"
```

Import the public key

```bash
((bedrock))[neil@bedrock ~]$ gpg --import awscli-exe-linux-x86_64.zip.pub
```

```text
gpg: directory '/home/neil/.gnupg' created
gpg: /home/neil/.gnupg/trustdb.gpg: trustdb created
gpg: key A6310ACC4672475C: public key "AWS CLI Team <aws-cli@amazon.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Verify the installer signature

```bash
((bedrock))[neil@bedrock ~]$ gpg --verify awscli-exe-linux-x86_64.zip.sig awscli-exe-linux-x86_64.zip
```

```
gpg: Signature made Mon 10 Nov 2025 07:20:50 PM UTC
gpg:                using RSA key FB5DB77FD5C118B80511ADA8A6310ACC4672475C
gpg: Good signature from "AWS CLI Team <aws-cli@amazon.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: FB5D B77F D5C1 18B8 0511  ADA8 A631 0ACC 4672 475C
```

Decompress the installer

```bash
((bedrock))[neil@bedrock ~]$ unzip -q awscli-exe-linux-x86_64.zip
```

Run the installer

```bash
((bedrock))[neil@bedrock ~]$ sudo aws/install
```

Check the version of the AWS CLI

```bash
((bedrock))[neil@bedrock ~]$ aws --version
aws-cli/2.31.33 Python/3.13.9 Linux/6.12.0-140.el10.x86_64 exe/x86_64.centos.10
```

## 3.2 Authenticate as an IAM user with AWS CLI

- https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-user.html

Configure the default profile of the AWS CLI

```bash
((bedrock))[neil@bedrock ~]$ aws configure
```

```
AWS Access Key ID [None]: AAAAAAAAAAAAAAAA
AWS Secret Access Key [None]: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Default region name [None]: us-east-1
Default output format [None]: json
```

If you prefer to configure a named profile of the AWS CLI

```bash
((bedrock))[neil@bedrock ~]$ aws configure --profile ${PROFILE_NAME}
```

If you downloaded the access keys as a CSV file

```bash
((bedrock))[neil@bedrock ~]$ aws configure import --csv file://credentials.csv
```

All of these create two configuration files,  `~/.aws/config` and `~/.aws/credentials`

```bash
((bedrock))[neil@bedrock ~]$ cat ~/.aws/config
[default]
region = us-east-1
output = json
```

```
((bedrock))[neil@bedrock ~]$ cat ~/.aws/credentials
[default]
aws_access_key_id = AAAAAAAAAAAAAAAA
aws_secret_access_key = AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

## 3.3 Use AWS CLI Bedrock command to list foundation models

After creating the `bedrock` user and configuring the AWS CLI with the access keys for the `bedrock` user

```bash
((bedrock))[neil@bedrock ~]$ aws bedrock list-foundation-models
```

```text
{
    "modelSummaries": [
        {
            "modelArn": "arn:aws:bedrock:us-east-1::foundation-model/stability.stable-fast-upscale-v1:0",
            "modelId": "stability.stable-fast-upscale-v1:0",
            "modelName": "Stable Image Fast Upscale",
            "providerName": "Stability AI",
<snip>
```

# 4 Bedrock command in Python

If you need to create and activate a Python virtual environment, see 2 Software installation


## 4.1 Install `boto3`

- https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html

Install the latest version (recommended)

```bash
((bedrock))[neil@bedrock ~]$ pip install boto3
```

Install a specific version 1.0.0

```bash
((bedrock))[neil@bedrock ~]$ pip install boto3==1.0.0
```

Install a version no older than 1.15.0

```bash
((bedrock))[neil@bedrock ~]$ pip install boto3>=1.15.0
```

Install a version no newer than 1.15.3

```bash
((bedrock))[neil@bedrock ~]$ pip install boto3<=1.15.3
```

## 4.2 Use a Python script to list foundation models

- https://docs.aws.amazon.com/bedrock/latest/userguide/models-get-info.html

Create the script

```bash
((bedrock))[neil@bedrock ~]$ cat << EOF > list-foundation-models.py
import boto3
import json

bedrock = boto3.client(service_name='bedrock')
response = bedrock.list_foundation_models()
models = response["modelSummaries"]

print( 'Got %s models' % ( len( models ) ) )

for model in models:
    print( "# %s\n" % model["modelName"] )
    print( json.dumps(model, indent=2 ) )
    print( )
EOF
```

Run the script

```bash
((bedrock))[neil@bedrock ~]$ python list-foundation-models.py
```

```
Got 105 models
Stable Image Fast Upscale
{
  "modelArn": "arn:aws:bedrock:us-east-1::foundation-model/stability.stable-fast-upscale-v1:0",
  "modelId": "stability.stable-fast-upscale-v1:0",
  "modelName": "Stable Image Fast Upscale",
  "providerName": "Stability AI",
  "inputModalities": [
    "TEXT",
    "IMAGE"
  ],
  "outputModalities": [
    "IMAGE"
  ],
  "responseStreamingSupported": false,
  "customizationsSupported": [],
  "inferenceTypesSupported": [
    "INFERENCE_PROFILE"
  ],
  "modelLifecycle": {
    "status": "ACTIVE"
  }
}
----
<snip>
```

## 4.3 Use a Python script to interact with InvokeModel

Create the script

```bash
((bedrock))[neil@bedrock ~]$ cat << EOF > invoke.py
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

model = "amazon.titan-text-express-v1"
prompt = "Describe the purpose of a Hello World program in one line."
request = {
  "inputText": prompt,
  "textGenerationConfig": {
    "maxTokenCount": 512,
    "temperature": 0.5,
    "topP": 0.9
  }
}
jsonrequest = json.dumps( request )

response = bedrock.invoke_model(modelId=model, body=jsonrequest)

body = json.loads(response["body"].read())
text = body["results"][0]["outputText"]

print( text )
EOF
```

Run the script

```bash
((bedrock))[neil@bedrock ~]$ python invoke.py
```

```
The purpose of a Hello World program is to demonstrate the basic functionality of a programming language or environment by printing the phrase "Hello, World!" to the console.
```

## 4.4 Use a Python script to interact with Converse

Converse is recommended because it is the basis for multi-turn conversations

```bash
((bedrock))[neil@bedrock ~]$ cat << EOF > converse.py
import boto3

bedrock = boto3.client('bedrock-runtime')

model = "amazon.titan-text-express-v1"
message = "Describe the purpose of a Hello World program in one line."

conversation = [
  {
    "role": "user",
    "content": [{"text": message }]
  }
]

response = bedrock.converse(
  modelId=model, 
  messages=conversation, 
  inferenceConfig={ "maxTokens": 512, "temperature": 0.5, "topP": 0.9 }
  )

text = response["output"]["message"]["content"][0]["text"]

print( text )
EOF
```

Run the script

```bash
((bedrock))[neil@bedrock ~]$ python converse.py
```

```
A Hello World program is a basic computer program that outputs the phrase "Hello, World!" to the console. It is often the first program that new programmers write to learn the basics of a programming language.
```

# 5 Tear down

## 5.1 Amazon Web Services

Log in with the root account

- Deactivate the short-term access key for IAM user `bedrock`
- Delete the IAM user `bedrock`
- Delete the IAM group `bedrock-users`

Log out

## 5.2 Google Cloud Platform

- Delete the Compute Engine instance `bedrock`

## 5.3 Laptop

- Delete the SSH public key file `credentials/cloud.google.com/bedrock`
- Delete the SSH private key file `credentials/cloud.google.com/bedrock.pub`

