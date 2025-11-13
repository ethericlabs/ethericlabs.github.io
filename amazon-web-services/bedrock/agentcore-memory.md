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

- Session
    - Events
        - Unencrypted event metadata

```
Session 1 prompt 1: What's the weather like in Seattle ?
Session 1 response 1: Pretty good
Session 1 prompt 2: What about tomorrow ?
Session 1 response 2: Also, pretty good
```

## Long term memory

Multiple session recall

- Session
  - Events
      - Trigger
          - Built in strategies
              - Insight extraction
              - Consolidation
          - Built in overrides
          - Self-managed strategies

Long term memory provides personal continuity, whereas retrieval augmented generation provides data from curated resources

```
Session 1 prompt 1: Window seat, please
Session 1 response 1: That's booked
```

```
Session 2 prompt 1: Any seats on Friday ?
Session 2 response 1: Sure, would you like a window seat ?
```


# Benefits

- Remembering previous preferences and topics to establish context and resolve ambiguity
- Personalisation
- Reduced development complexity


# Getting started

- https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-get-started.html


This requires that the user group `bedrock-users` be given these permissions

- `AmazonBedrockFullAccess`
- `BedrockAgentCoreFullAccess`


## 1 Create and activate a Python virtual environment

```
[neil@bedrock ~]$ python -m venv agentcore-memory
```

```
((agentcore-memory)) [neil@bedrock ~]$ source agentcore-memory/bin/activate
```

```
((agentcore-memory)) [neil@bedrock ~]$ pip install bedrock-agentcore bedrock-agentcore-starter-toolkit
```


## 2 Python to interact with memory

### 2.1 Create memory

Create Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > agentcore-memory-1-create-memory.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

region = 'us-east-1'

print( 'Create a memory manager in region [ %s ]' % ( region ) )

memory_manager = MemoryManager(region_name=region)

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

print(f"Memory manager created with identifier [ {memory.get('id')} ]")
EOF
```

Run Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ python agentcore-memory-1-create-memory.py
```

```text
<snip>
Memory manager created with identifier [ CustomerSupportSemantic-03SmFW6MAU ]
```

Define an environment variable to provide the identifier to the following Python scripts 

```bash
((agentcore-memory)) [neil@bedrock ~]$ MEMORY_IDENTIFIER="XXXXXXX"
```

### 2.2 Simulate conversation

Create Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > agentcore-memory-2-simulate-conversation.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_identifier = '${MEMORY_IDENTIFIER}'

region = 'us-east-1'

print( 'Create a session manager' )

session_manager = MemorySessionManager( memory_id=memory_identifier, region_name=region )

print( '- Session manager created' )

session = session_manager.create_memory_session(
    actor_id="User1",
    session_id="OrderSupportSession1"
)

print( '- Session created' )

print( 'Simulate a conversation' )

print( '- Simulate message 1' )

author = MessageRole.ASSISTANT
message = "Hi, how can I help you today?"
print( "  - %s -- %s" % ( message, author ) )
session.add_turns( messages=[ ConversationalMessage( message, author ) ] )

print( '- Simulate message 2' )

author = MessageRole.USER
message = "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476"
print( "  - %s -- %s" % ( message, author ) )
session.add_turns( messages=[ ConversationalMessage( message, author ) ] )

print( '- Simulate message 3' )

author = MessageRole.ASSISTANT
message = "I'm sorry to hear that. Let me look up your order."
print( "  - %s -- %s" % ( message, author ) )
session.add_turns( messages=[ ConversationalMessage( message, author ) ] )

print( 'Conversation simulated' )
EOF
```

Run Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ python agentcore-memory-2-simulate-conversation.py
```

```text
Create a session manager
- Session manager created
- Session created
Simulate a conversation
- Simulate message 1
  - Hi, how can I help you today? -- MessageRole.ASSISTANT
- Simulate message 2
  - Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476 -- MessageRole.USER
- Simulate message 3
  - I'm sorry to hear that. Let me look up your order. -- MessageRole.ASSISTANT
Conversation simulated
```

### 2.3 Recall short term memory

Create Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > agentcore-memory-3-recall-short-term.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_identifier = '${MEMORY_IDENTIFIER}'

region = 'us-east-1'

print( 'Create a session manager' )

session_manager = MemorySessionManager( memory_id=memory_identifier, region_name=region )

print( '- Session manager created' )

session = session_manager.create_memory_session(
    actor_id="User1",
    session_id="OrderSupportSession1"
)

print( '- Session created' )

print( 'Retrieve exchanges from short term memory' )

print( '- Get the last 5 exchanges' )

turns = session.get_last_k_turns(k=5)

print( '- Got %i exchanges' % ( len(turns) ) )

for turn in turns:
    print(f"  - {turn}")
EOF
```

Run Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ python agentcore-memory-3-recall-short-term.py
```

```text
Create a session manager
- Session manager created
- Session created
Retrieve exchanges from short term memory
- Get the last 5 exchanges
- Got 2 exchanges
  - [{'content': {'text': "I'm sorry to hear that. Let me look up your order."}, 'role': 'ASSISTANT'}]
  - [{'content': {'text': "Hi, I am a new customer. I just made an order, but it hasn't arrived. The Order number is #35476"}, 'role': 'USER'}, {'content': {'text': 'Hi, how can I help you today?'}, 'role': 'ASSISTANT'}]
```

### 2.4 Recall long term memory

Create Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > agentcore-memory-4-recall-long-term.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_identifier = '${MEMORY_IDENTIFIER}'

region = 'us-east-1'

print( 'Create a session manager' )

session_manager = MemorySessionManager( memory_id=memory_identifier, region_name=region )

print( '- Session manager created' )

session = session_manager.create_memory_session(
    actor_id="User1",
    session_id="OrderSupportSession1"
)

print( '- Session created' )

print( 'Retrieving records from long term memory' )

print( '- Get long term memory records' )

memory_records = session.list_long_term_memory_records(
    namespace_prefix="/"
)

print( '- Got %i records' % ( len(memory_records) ) )

for record in memory_records:
    print("--------------------------------------------------------------------")
    print(f"Memory record: {record}")
    print("--------------------------------------------------------------------")
EOF
```

Run Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ python agentcore-memory-4-recall-long-term.py
```

```text
Create a session manager
- Session manager created
- Session created
Retrieving records from long term memory
- Get long term memory records
- Got 3 records
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-9cbb5762-f197-4fdd-831a-903978ba0f26', 'content': {'text': 'The user is a new customer.'}, 'memoryStrategyId': 'semanticLongTermMemory-mGTO1bBV06', 'namespaces': ['/strategies/semanticLongTermMemory-mGTO1bBV06/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 13, 14, 42, 40, 992000, tzinfo=tzlocal())}
--------------------------------------------------------------------
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-72b80398-75f7-4aff-adaf-663c272ad12f', 'content': {'text': "The user's order has not arrived."}, 'memoryStrategyId': 'semanticLongTermMemory-mGTO1bBV06', 'namespaces': ['/strategies/semanticLongTermMemory-mGTO1bBV06/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 13, 14, 42, 40, 992000, tzinfo=tzlocal())}
--------------------------------------------------------------------
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-7b96a5b1-a23b-4f6a-9194-ed57ada63a59', 'content': {'text': 'The user made an order with order number #35476.'}, 'memoryStrategyId': 'semanticLongTermMemory-mGTO1bBV06', 'namespaces': ['/strategies/semanticLongTermMemory-mGTO1bBV06/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 13, 14, 42, 40, 992000, tzinfo=tzlocal())}
--------------------------------------------------------------------
```

### 2.5 Search long term memory

Create Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > agentcore-memory-5-search-long-term.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_identifier = '${MEMORY_IDENTIFIER}'

region = 'us-east-1'

print( 'Create a session manager' )

session_manager = MemorySessionManager( memory_id=memory_identifier, region_name=region )

print( '- Session manager created' )

session = session_manager.create_memory_session(
    actor_id="User1",
    session_id="OrderSupportSession1"
)

print( '- Session created' )

print( 'Perform a semantic search of long term memory' )

# Perform a semantic search
memory_records = session.search_long_term_memories(
    query="can you summarize the support issue",
    namespace_prefix="/",
    top_k=3
)

print( '- Got %i records' % ( len(memory_records) ) )

for record in memory_records:
    print("--------------------------------------------------------------------")
    print(f"Memory record: {record}")
    print("--------------------------------------------------------------------")
EOF
```

Run Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ python agentcore-memory-5-search-long-term.py
```

```text
Create a session manager
- Session manager created
- Session created
Perform a semantic search of long term memory
- Got 3 records
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-9cbb5762-f197-4fdd-831a-903978ba0f26', 'content': {'text': 'The user is a new customer.'}, 'memoryStrategyId': 'semanticLongTermMemory-mGTO1bBV06', 'namespaces': ['/strategies/semanticLongTermMemory-mGTO1bBV06/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 13, 14, 42, 40, 992000, tzinfo=tzlocal()), 'score': 0.36923698}
--------------------------------------------------------------------
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-72b80398-75f7-4aff-adaf-663c272ad12f', 'content': {'text': "The user's order has not arrived."}, 'memoryStrategyId': 'semanticLongTermMemory-mGTO1bBV06', 'namespaces': ['/strategies/semanticLongTermMemory-mGTO1bBV06/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 13, 14, 42, 40, 992000, tzinfo=tzlocal()), 'score': 0.36800358}
--------------------------------------------------------------------
--------------------------------------------------------------------
Memory record: {'memoryRecordId': 'mem-7b96a5b1-a23b-4f6a-9194-ed57ada63a59', 'content': {'text': 'The user made an order with order number #35476.'}, 'memoryStrategyId': 'semanticLongTermMemory-mGTO1bBV06', 'namespaces': ['/strategies/semanticLongTermMemory-mGTO1bBV06/actors/User1'], 'createdAt': datetime.datetime(2025, 11, 13, 14, 42, 40, 992000, tzinfo=tzlocal()), 'score': 0.36460194}
--------------------------------------------------------------------
```

### 2.6 Delete memory

Create Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ cat << EOF > agentcore-memory-6-delete-memory.py
from bedrock_agentcore_starter_toolkit.operations.memory.manager import MemoryManager
from bedrock_agentcore.memory.session import MemorySessionManager
from bedrock_agentcore.memory.constants import ConversationalMessage, MessageRole
from bedrock_agentcore_starter_toolkit.operations.memory.models.strategies import SemanticStrategy
import time

memory_identifier = '${MEMORY_IDENTIFIER}'

region = 'us-east-1'

print( 'Create a memory manager' )

memory_manager = MemoryManager( region_name=region )

memory_manager.delete_memory( memory_id=memory_identifier )

EOF
```

Run Python code

```bash
((agentcore-memory)) [neil@bedrock ~]$ python agentcore-memory-6-delete-memory.py
```

```text
<snip>
Deleted memory: CustomerSupportSemantic-03SmFW6MAU
```
