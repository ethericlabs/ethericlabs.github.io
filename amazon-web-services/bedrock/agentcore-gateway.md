# [Home](https://ethericlabs.github.io) / [Amazon Web Services](../README.md) / [Bedrock](README.md) / [AgentCore](agentcore.md) / AgentCore gateway


# Summary

An overview of Bedrock AgentCore gateway


# References

Amazon Bedrock AgentCore Gateway: Securely connect tools and other resources to your gateway

- [https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway.html](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway.html)

Core concepts for Amazon Bedrock AgentCore gateway

- [https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway-core-concepts.html](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway-core-concepts.html)


# Overview

Convert resources (APIs, third party services etc) into model context protocol (MCP) compatible tools and make them available at an endpoint.

Resources can be expressed as
- OpenAPI
- Smithy
- Lambda

Ingress and egress are authenticated

Integration with some resources is facilitated
- Salesforce
- Slack
- Atlassian Jira
- Asana
- Zendesk

Gateways can be defined in a number of ways
- AgentCore CLI
- AWS CLI
- Python

# Demo 

Add a new permission policy to the `bedrock-users` group to provide
- `cognito-idp:*`
- `iam:*`
- `bedrock-agentcore:*`
- `lambda:*`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "bedrock-users-demo",
      "Effect": "Allow",
      "Action": [
        "bedrock-agentcore:*",
        "cognito-idp:*",
        "iam:*",
        "lambda:*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

```text
python -m venv agentcore-gateway
source agentcore-gateway/bin/activate
pip install boto3 bedrock-agentcore-starter-toolkit strands-agents
```

Create Python script

```text
cat << EOF > create_gateway.py
from bedrock_agentcore_starter_toolkit.operations.gateway.client import GatewayClient

import json
import logging
import time

region = 'us-east-1'

def createGateway():
    print( 'Creating gateway' )
    client = GatewayClient( region_name=region )
    print( '- Creating an OAuth authorizer to provide a client identifier and secret' )
    cognito_response = client.create_oauth_authorizer_with_cognito( "TestGateway" )
    print( '- Creating a gateway with MCP support, OAuth authorisation and semantic search' )
    gateway = client.create_mcp_gateway(
            name=None,
            role_arn=None,
            authorizer_config=cognito_response["authorizer_config"],
            enable_semantic_search=True
            )
    print( 'Fixing IAM permissions (30s)' )
    client.fix_iam_permissions(gateway)
    time.sleep(30)
    print( '- Creating lambda target with get_weather and get_time tools, then register it as a target in the gateway' )
    lambda_target = client.create_mcp_gateway_target(
            gateway=gateway,
            name=None,
            target_type="lambda",
            target_payload=None,
            credentials=None
            )
    print( 'Dumping configuration file for agents to adopt' )
    config = {
            "gateway_url": gateway["gatewayUrl"],
            "gateway_id": gateway["gatewayId"],
            "region": region,
            "client_info": cognito_response["client_info"]
            }
    with open( "/tmp/gateway_config.json", "w") as f:
        json.dump(config, f, indent=2)
    print( 'Wrote gateway configuration to /tmp/gateway_config.json' )
    return config

if __name__ == "__main__":
    createGateway()
EOF
```

Run the script

```text
python create_gateway.py
```

```text
Creating gateway
- Creating an OAuth authorizer to provide a client identifier and secret
2025-11-17 15:03:59,815 - bedrock_agentcore.gateway - INFO - Starting EZ Auth setup: Creating Cognito resources...
2025-11-17 15:04:01,058 - bedrock_agentcore.gateway - INFO -   ✓ Created User Pool: us-east-1_QpeJFIWvW
2025-11-17 15:04:01,725 - bedrock_agentcore.gateway - INFO -   ✓ Created domain: agentcore-87d12e1e
2025-11-17 15:04:01,725 - bedrock_agentcore.gateway - INFO -   ⏳ Waiting for domain to be available...
2025-11-17 15:04:01,788 - bedrock_agentcore.gateway - INFO -   ✓ Domain is active
2025-11-17 15:04:02,042 - bedrock_agentcore.gateway - INFO -   ✓ Created resource server: TestGateway
2025-11-17 15:04:02,337 - bedrock_agentcore.gateway - INFO -   ✓ Created client: k3mgdrnhrklpkns5cc146op0n
2025-11-17 15:04:02,337 - bedrock_agentcore.gateway - INFO -   ⏳ Waiting for DNS propagation of domain: agentcore-87d12e1e.auth.us-east-1.amazoncognito.com
2025-11-17 15:05:02,338 - bedrock_agentcore.gateway - INFO - ✓ EZ Auth setup complete!
- Creating a gateway with MCP support, OAuth authorisation and semantic search
2025-11-17 15:05:02,339 - bedrock_agentcore.gateway - INFO - Role not provided, creating an execution role to use
2025-11-17 15:05:02,469 - bedrock_agentcore.gateway - INFO - ✓ Role already exists: arn:aws:iam::496170005851:role/AgentCoreGatewayExecutionRole
2025-11-17 15:05:02,470 - bedrock_agentcore.gateway - INFO - ✓ Successfully created execution role for Gateway
2025-11-17 15:05:02,471 - bedrock_agentcore.gateway - INFO - Creating Gateway
2025-11-17 15:05:02,669 - bedrock_agentcore.gateway - INFO - ✓ Created Gateway: arn:aws:bedrock-agentcore:us-east-1:496170005851:gateway/testgatewaye4548e26-tq5tq7rngt
2025-11-17 15:05:02,669 - bedrock_agentcore.gateway - INFO -   Gateway URL: https://testgatewaye4548e26-tq5tq7rngt.gateway.bedrock-agentcore.us-east-1.amazonaws.com/mcp
2025-11-17 15:05:02,669 - bedrock_agentcore.gateway - INFO -   Waiting for Gateway to be ready...
2025-11-17 15:05:04,874 - bedrock_agentcore.gateway - INFO -
✅Gateway is ready
Fixing IAM permissions (30s)
2025-11-17 15:05:05,080 - bedrock_agentcore.gateway - INFO - ✓ Fixed IAM permissions for Gateway
- Creating lambda target with get_weather and get_time tools, then register it as a target in the gateway
2025-11-17 15:05:35,578 - bedrock_agentcore.gateway - INFO - ✓ Created Lambda function: arn:aws:lambda:us-east-1:496170005851:function:AgentCoreLambdaTestFunction
2025-11-17 15:05:35,578 - bedrock_agentcore.gateway - INFO - ✓ Attaching access policy to: arn:aws:lambda:us-east-1:496170005851:function:AgentCoreLambdaTestFunction for arn:aws:iam::496170005851:role/AgentCoreGatewayExecutionRole
2025-11-17 15:05:35,667 - bedrock_agentcore.gateway - INFO - ✓ Attached permissions for role invocation: arn:aws:lambda:us-east-1:496170005851:function:AgentCoreLambdaTestFunction
2025-11-17 15:05:35,669 - bedrock_agentcore.gateway - INFO - Creating Target
2025-11-17 15:05:35,669 - bedrock_agentcore.gateway - INFO - {'gatewayIdentifier': 'testgatewaye4548e26-tq5tq7rngt', 'name': 'TestGatewayTarget2717487a', 'targetConfiguration': {'mcp': {'lambda': {'lambdaArn': 'arn:aws:lambda:us-east-1:496170005851:function:AgentCoreLambdaTestFunction', 'toolSchema': {'inlinePayload': [{'name': 'get_weather', 'description': 'Get weather for a location', 'inputSchema': {'type': 'object', 'properties': {'location': {'type': 'string'}}, 'required': ['location']}}, {'name': 'get_time', 'description': 'Get time for a timezone', 'inputSchema': {'type': 'object', 'properties': {'timezone': {'type': 'string'}}, 'required': ['timezone']}}]}}}}, 'credentialProviderConfigurations': [{'credentialProviderType': 'GATEWAY_IAM_ROLE'}]}
2025-11-17 15:05:35,870 - bedrock_agentcore.gateway - INFO - ✓ Added target successfully (ID: CJ5BUN2E9H)
2025-11-17 15:05:35,870 - bedrock_agentcore.gateway - INFO -   Waiting for target to be ready...
2025-11-17 15:05:40,274 - bedrock_agentcore.gateway - INFO -
✅Target is ready
Dumping configuration file for agents to adopt
Wrote gateway configuration to /tmp/gateway_config.json
```

This creates a configuration file at `/tmp/gateway_config.json`

```
{
  "gateway_url": "https://testgatewaye4548e26-tq5tq7rngt.gateway.bedrock-agentcore.us-east-1.amazonaws.com/mcp",
  "gateway_id": "testgatewaye4548e26-tq5tq7rngt",
  "region": "us-east-1",
  "client_info": {
    "client_id": "k3mgdrnhrklpkns5cc146op0n",
    "client_secret": "4i8j0n8b7rbens1vglv4ah5dsoj4802a0l4p97g40fqlnid502k",
    "user_pool_id": "us-east-1_QpeJFIWvW",
    "token_endpoint": "https://agentcore-87d12e1e.auth.us-east-1.amazoncognito.com/oauth2/token",
    "scope": "TestGateway/invoke",
    "domain_prefix": "agentcore-87d12e1e"
  }
}
```

Create an agent

```text
cat << EOF > run_agent.py
from strands import Agent
from strands.models import BedrockModel
from strands.tools.mcp.mcp_client import MCPClient
from mcp.client.streamable_http import streamablehttp_client
from bedrock_agentcore_starter_toolkit.operations.gateway.client import GatewayClient
import json
import sys

def create_streamable_http_transport(mcp_url: str, access_token: str):
    return streamablehttp_client(mcp_url, headers={"Authorization": f"Bearer {access_token}"})

def get_full_tools_list(client):
    """Get all tools with pagination support"""
    more_tools = True
    tools = []
    pagination_token = None
    while more_tools:
        tmp_tools = client.list_tools_sync(pagination_token=pagination_token)
        tools.extend(tmp_tools)
        if tmp_tools.pagination_token is None:
            more_tools = False
        else:
            more_tools = True
            pagination_token = tmp_tools.pagination_token
    return tools

def run_agent():
    # Load configuration
    try:
        with open("/tmp/gateway_config.json", "r") as f:
            config = json.load(f)
    except FileNotFoundError:
        print("Error: gateway_config.json not found!")
        print("Please run 'python setup_gateway.py' first to create the Gateway.")
        sys.exit(1)

    gateway_url = config["gateway_url"]
    client_info = config["client_info"]

    # Get access token for the agent
    print("Getting access token...")
    client = GatewayClient(region_name=config["region"])
    access_token = client.get_access_token_for_cognito(client_info)
    print("Access token obtained\n")

    # Model configuration - change if needed
    model_id = "amazon.titan-text-lite-v1"

    print("Starting AgentCore Gateway Test Agent")
    print(f"Gateway URL: {gateway_url}")
    print(f"Model: {model_id}")
    print("-" * 60)

    # Setup Bedrock model
    bedrockmodel = BedrockModel(
        inference_profile_id=model_id,
        streaming=True,
    )

    # Setup MCP client
    mcp_client = MCPClient(lambda: create_streamable_http_transport(gateway_url, access_token))

    with mcp_client:
        # List available tools
        tools = get_full_tools_list(mcp_client)
        print(f"\nAvailable tools: {[tool.tool_name for tool in tools]}")
        print("-" * 60)

        # Create agent
        agent = Agent(model=bedrockmodel, tools=tools)

        # Interactive loop
        print("\nInteractive Agent Ready!")
        print("Try asking: 'What's the weather in Seattle?'")
        print("Type 'exit', 'quit', or 'bye' to end.\n")

        while True:
            user_input = input("You: ")
            if user_input.lower() in ["exit", "quit", "bye"]:
                print("Goodbye!")
                break

            print("\nThinking...\n")
            response = agent(user_input)
            print(f"\nAgent: {response.message.get('content', response)}\n")

if __name__ == "__main__":
    run_agent()
EOF
```

Run the script

```text
python run_agent.py
```

It sort of works, but strands.models default model is anthropic, for which a statement is required

```text
File "/home/neil/agentcore-gateway/lib64/python3.12/site-packages/strands/models/bedrock.py", line 768, in _stream
```
```text
botocore.errorfactory.ResourceNotFoundException: An error occurred (ResourceNotFoundException) when calling the ConverseStream operation: Model use case details have not been submitted for this account. Fill out the Anthropic use case details form before using the model. If you have already filled out the form, try again in 15 minutes.
```


```text
└ Bedrock region: us-east-1
└ Model id: us.anthropic.claude-sonnet-4-20250514-v1:0

```
