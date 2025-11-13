# [Amazon Web Services](../README.md) / [Bedrock](README.md) / [AgentCore](agentcore.md) / AgentCore memory


# Summary

An overview of Bedrock AgentCore memory


# References

Add memory to your Amazon Bedrock AgentCore agent

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory.html

Terminology

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-terminology.html

Memory strategies

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-strategies.html

Getting started with AgentCore memory

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-get-started.html

Complete list of Amazon Bedrock AgentCore memory operations in AgentCore Control

- https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-agentcore-control.html

Complete list of Amazon Bedrock AgentCore memory operations in AgentCore

- https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-agentcore.html


# Types of memory

Bedrock AgentCore memory dashboard

- https://us-east-1.console.aws.amazon.com/bedrock-agentcore/memory?region=us-east-1


## Short term memory

Single session recall

> Session 1 prompt 1: What's the weather like in Seattle ?
> Session 1 response 1: Pretty good
> Session 1 prompt 2: What about tomorrow ?
> Session 1 response 2: Also, pretty good

- Sessions
  - Events
      - Unencrypted event metadata


## Long term memory

Multiple session recall

> Session 1 prompt 1: Window seat, please
> Session 1 response 1: That's booked

> Session 2 prompt 1: Any seats on Friday ?
> Session 2 response 1: Sure, would you like a window seat ?

- Session
  - Events
      - Trigger
          - Built in strategies
              - Insight extraction
              - Consolidation
          - Built in overrides
          - Self-managed strategies

Long term memory provides personal continuity, whereas retrieval augmented generation provides data from curated resources


# Benefits

- Remembering previous preferences and topics to establish context and resolve ambiguity
- Personalisation
- Reduced development complexity


# Getting started

Status: Incomplete ... and needs testing
https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-get-started.html


Requires `BedrockAgentCoreFullAccess` permissions on the `bedrock-users` group that was created while getting started


## 0 Set up Python

```
[neil@bedrock ~]$ python -m venv agentcore-memory
```

```
((agentcore-memory)) [neil@bedrock ~]$ source agentcore-memory/bin/activate
```

```
((agentcore-memory)) [neil@bedrock ~]$ pip install bedrock-agentcore bedrock-agentcore-starter-toolkit
```

## 1 Create an AgentCore Memory

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > create-memory.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_manager = MemoryManager(region_name="us-east-1")

print("Creating memory resource...")

memory = memory_manager.get_or_create_memory(
    name="CustomerSupportSemantic",
    description="Customer support memory store",
    strategies=[
        SemanticStrategy(
            name="semanticLongTermMemory",
            namespaces=['/strategies/{memoryStrategyId}/actors/{actorId}'],
        )
    ]
)

print(f"Memory ID: {memory.get('id')}")

memories = memory_manager.list_memories()

print(memories)
EOF
```

```bash
((agentcore-memory)) [neil@bedrock ~]$ python create-memory.py
```

```bash
✅ MemoryManager initialized for region: us-east-1
Creating memory resource...
Created memory: CustomerSupportSemantic-oWuCeU3dJK
Created memory CustomerSupportSemantic-oWuCeU3dJK, waiting for ACTIVE status...
Waiting for memory CustomerSupportSemantic-oWuCeU3dJK to return to ACTIVE state and strategies to reach terminal states...
[15:19:39]    ⏳ Memory: CREATING, Strategies: 0/1 active (10s elapsed)
<snip>
[15:22:01]    ⏳ Memory: CREATING, Strategies: 0/1 active (152s elapsed)
[15:22:11]    ⏳ Memory: ACTIVE, Strategies: 1/1 active (162s elapsed)
Memory CustomerSupportSemantic-oWuCeU3dJK is ACTIVE and all strategies are in terminal states (took 162 seconds)
              ✅ Memory is ACTIVE (took 162s)
Memory ID: CustomerSupportSemantic-oWuCeU3dJK
[{'arn': 'arn:aws:bedrock-agentcore:us-east-1:496170005851:memory/CustomerSupportSemantic-oWuCeU3dJK', 'id': 'CustomerSupportSemantic-oWuCeU3dJK', 'status': 'ACTIVE', 'createdAt': datetime.datetime(2025, 11, 12, 15, 19, 28, 671000, tzinfo=tzlocal()), 'updatedAt': datetime.datetime(2025, 11, 12, 15, 19, 28, 932000, tzinfo=tzlocal()), 'memoryId': 'CustomerSupportSemantic-oWuCeU3dJK'}]
```

## 2 Write events to memory

```
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > write-events-to-memory.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_manager = MemoryManager(region_name="us-east-1")

print("Creating memory resource...")

memory = memory_manager.get_or_create_memory(
    name="CustomerSupportSemantic",
    description="Customer support memory store",
    strategies=[
        SemanticStrategy(
            name="semanticLongTermMemory",
            namespaces=['/strategies/{memoryStrategyId}/actors/{actorId}'],
        )
    ]
)

print(f"Memory ID: {memory.get('id')}")

print( 'Simulating chat' )

# Create a session to store memory events
session_manager = MemorySessionManager(
    memory_id=memory.get("id"),
    region_name="us-east-1")

session = session_manager.create_memory_session(
    actor_id="User1",
    session_id="OrderSupportSession1"
)

# Write memory events (conversation turns)
session.add_turns(
    messages=[
        ConversationalMessage(
            "Hi, how can I help you today?",
            MessageRole.ASSISTANT)],
)

session.add_turns(
    messages=[
        ConversationalMessage(
            "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476",
            MessageRole.USER)],
)

session.add_turns(
    messages=[
        ConversationalMessage(
            "I'm sorry to hear that. Let me look up your order.",
            MessageRole.ASSISTANT)],
)

print( 'Retrieving chat' )

# Get the last k turns in the session
turns = session.get_last_k_turns(k=5)

for turn in turns:
    print(f"Turn: {turn}")
EOF
```

```bash
((agentcore-memory)) [neil@bedrock ~]$ python write-events-to-memory.py
```

```bash
((agentcore-memory) ) [neil@bedrock ~]$ python write-events-to-memory.py
✅ MemoryManager initialized for region: us-east-1
Creating memory resource...
Memory already exists. Using existing memory ID: CustomerSupportSemantic-oWuCeU3dJK
🔎 Retrieving memory resource with ID: CustomerSupportSemantic-oWuCeU3dJK...
  Found memory: CustomerSupportSemantic-oWuCeU3dJK
Existing {'type': 'SEMANTIC', 'name': 'semanticLongTermMemory', 'description': None, 'namespaces': ['/strategies/{memoryStrategyId}/actors/{actorId}']}
Requested {'type': 'SEMANTIC', 'name': 'semanticLongTermMemory', 'description': None, 'namespaces': ['/strategies/{memoryStrategyId}/actors/{actorId}']}
Universal strategy validation passed for memory CustomerSupportSemantic. Strategies match: [SEMANTIC]
Memory ID: CustomerSupportSemantic-oWuCeU3dJK
Simulating chat
Retrieving chat
Turn: [{'content': {'text': "I'm sorry to hear that. Let me look up your order."}, 'role': 'ASSISTANT'}]
Turn: [{'content': {'text': "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476"}, 'role': 'USER'}, {'content': {'text': 'Hi, how can I help you today?'}, 'role': 'ASSISTANT'}]
```

## 3 Retrieve records from long term memory

```
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_manager = MemoryManager(region_name="us-east-1")

print("Creating memory resource...")

memory = memory_manager.get_or_create_memory(
    name="CustomerSupportSemantic",
    description="Customer support memory store",
    strategies=[
        SemanticStrategy(
            name="semanticLongTermMemory",
            namespaces=['/strategies/{memoryStrategyId}/actors/{actorId}'],
        )
    ]
)

print(f"Memory ID: {memory.get('id')}")

print( 'Simulating chat' )

# Create a session to store memory events
session_manager = MemorySessionManager(
    memory_id=memory.get("id"),
    region_name="us-east-1")

session = session_manager.create_memory_session(
    actor_id="User1",
    session_id="OrderSupportSession1"
)

# Write memory events (conversation turns)
session.add_turns(
    messages=[
        ConversationalMessage(
            "Hi, how can I help you today?",
            MessageRole.ASSISTANT)],
)

session.add_turns(
    messages=[
        ConversationalMessage(
            "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476",
            MessageRole.USER)],
)

session.add_turns(
    messages=[
        ConversationalMessage(
            "I'm sorry to hear that. Let me look up your order.",
            MessageRole.ASSISTANT)],
)

print( 'Retrieving chat' )

# Get the last k turns in the session
turns = session.get_last_k_turns(k=5)

for turn in turns:
    print(f"Turn: {turn}")

print( 'Retrieving records from long term memory' )

# List all memory records
memory_records = session.list_long_term_memory_records(
    namespace_prefix="/"
)

for record in memory_records:
    print(f"Memory record: {record}")
    print("--------------------------------------------------------------------")

print( 'Perform a semantic search of long term memory' )

# Perform a semantic search
memory_records = session.search_long_term_memories(
    query="can you summarize the support issue",
    namespace_prefix="/",
    top_k=3
)

```

```bash
(agentcore-memory) ) [neil@bedrock ~]$ python retrieve-records-from-long-term-memory.py
✅ MemoryManager initialized for region: us-east-1
Creating memory resource...
Memory already exists. Using existing memory ID: CustomerSupportSemantic-oWuCeU3dJK
🔎 Retrieving memory resource with ID: CustomerSupportSemantic-oWuCeU3dJK...
  Found memory: CustomerSupportSemantic-oWuCeU3dJK
Existing {'type': 'SEMANTIC', 'name': 'semanticLongTermMemory', 'description': None, 'namespaces': ['/strategies/{memoryStrategyId}/actors/{actorId}']}
Requested {'type': 'SEMANTIC', 'name': 'semanticLongTermMemory', 'description': None, 'namespaces': ['/strategies/{memoryStrategyId}/actors/{actorId}']}
Universal strategy validation passed for memory CustomerSupportSemantic. Strategies match: [SEMANTIC]
Memory ID: CustomerSupportSemantic-oWuCeU3dJK
Simulating chat
Retrieving chat
Turn: [{'content': {'text': "I'm sorry to hear that. Let me look up your order."}, 'role': 'ASSISTANT'}]
Turn: [{'content': {'text': "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476"}, 'role': 'USER'}, {'content': {'text': 'Hi, how can I help you today?'}, 'role': 'ASSISTANT'}, {'content': {'text': "I'm sorry to hear that. Let me look up your order."}, 'role': 'ASSISTANT'}]
Turn: [{'content': {'text': "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476"}, 'role': 'USER'}, {'content': {'text': 'Hi, how can I help you today?'}, 'role': 'ASSISTANT'}]
Retrieving records from long term memory
Memory record: {'memoryRecordId': 'mem-b13428f6-03dc-4d27-b624-12e4f338533a', 'content': {'text': 'The user made an order with order number #35476.'}, 'memoryStrategyId': 'semanticLongTermMemory-UOxqslER9W', 'namespaces': ['/strategies/semanticLongTermMemory-UOxqslER9W/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 12, 15, 44, 50, 267000, tzinfo=tzlocal())}
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-ccf75b23-0a0c-4452-9756-2528cf8010fe', 'content': {'text': 'The user is a new customer.'}, 'memoryStrategyId': 'semanticLongTermMemory-UOxqslER9W', 'namespaces': ['/strategies/semanticLongTermMemory-UOxqslER9W/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 12, 15, 44, 50, 267000, tzinfo=tzlocal())}
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-b6485257-c03f-476e-aed2-1b33eb072e15', 'content': {'text': "The user's order hasn't arrived yet."}, 'memoryStrategyId': 'semanticLongTermMemory-UOxqslER9W', 'namespaces': ['/strategies/semanticLongTermMemory-UOxqslER9W/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 12, 15, 44, 50, 267000, tzinfo=tzlocal())}
--------------------------------------------------------------------
Perform a semantic search of long term memory
```

## 4 Tear down

```
memory_manager.delete_memory(memory_id=memory.get("id"))
```
