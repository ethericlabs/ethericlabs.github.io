# [Home](https://ethericlabs.github.io/README.md) / [Amazon Web Services](../README.md) / [Bedrock](README.md) / [AgentCore](agentcore.md) / AgentCore gateway


# Summary

An overview of Bedrock AgentCore gateway


# References

Amazon Bedrock AgentCore Gateway: Securely connect tools and other resources to your gateway

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway.html

Core concepts for Amazon Bedrock AgentCore gateway

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway-core-concepts.html


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
