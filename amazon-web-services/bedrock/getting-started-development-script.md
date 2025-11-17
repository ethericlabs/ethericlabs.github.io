# [Home](https://ethericlabs.github.io) / [Amazon Web Services](../README.md) / [Bedrock](README.md) / Getting started development script

Define AWS credentials

```
AWS_ACCESS_KEY_ID="XXXXXXXXXXXXXXXX"
AWS_SECRET_ACCESS_KEY="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```

Install, configure and test the AWS CLI and Python API

```
python -m venv bedrock
source bedrock/bin/activate
sudo dnf -y install unzip
cat << EOF > awscli-exe-linux-x86_64.zip.pub
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
curl --silent --remote-name "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
curl --silent --remote-name "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip.sig"
gpg --import awscli-exe-linux-x86_64.zip.pub
gpg --verify awscli-exe-linux-x86_64.zip.sig awscli-exe-linux-x86_64.zip
unzip -q awscli-exe-linux-x86_64.zip
sudo aws/install
aws --version
mkdir ~/.aws
cat << EOF > ~/.aws/config
[default]
region = us-east-1
output = json
EOF
cat << EOF > ~/.aws/credentials
[default]
aws_access_key_id = ${AWS_ACCESS_KEY_ID}
aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}
EOF
aws bedrock list-foundation-models | grep modelId | awk '{ print $2 }' | sed 's/^"//' | sed 's/",$//'
pip install boto3
cat << EOF > list-foundation-models.py
import boto3
import json

bedrock = boto3.client(service_name='bedrock')
response = bedrock.list_foundation_models()
models = response["modelSummaries"]

print( 'Got %s models' % ( len( models ) ) )
EOF
cat << EOF > invoke.py
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
cat << EOF > converse.py
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
python list-foundation-models.py
python invoke.py
python converse.py
```
