# ReAct-agent-from-scratch-using-only-python-

# Agent Class for ReAct Framework

This `Agent` class is designed to interact with an LLM (e.g., Groq's API) while implementing the ReAct (Reasoning + Acting) framework. It handles conversation flow, maintains message history, and sends prompts to the model.

## Code Breakdown  

### 1️⃣ **Initialization (`__init__`)**  
The `Agent` class initializes with the following parameters:  
- **`client`:** The client object used to communicate with the LLM API.  
- **`system`:** (Optional) A system-level prompt (e.g., guiding the agent to follow the ReAct framework).  
- **`messages`:** A list that stores the conversation history (system prompt, user queries, and assistant responses).  
  - If a `system` prompt is provided, it is added to the `messages` list with the role set to `"system"`.  

### 2️⃣ **Callable Interface (`__call__`)**  
- Makes the agent class callable.  
- Appends the user's message to the conversation history (if provided).  
- Calls the `execute()` method to fetch the assistant's response.  
- Adds the assistant's response to the `messages` list.  
- Returns the assistant's response.  

### 3️⃣ **Execution (`execute`)**  
- Sends the `messages` list to the LLM API using `client.chat.completions.create`.  
- Assumes the model being used is `"llama3-70b-8192"`.  
- Extracts and returns the assistant's response content from the API result.  

---

## Code Implementation  

```python
class Agent:
    def __init__(self, client: Groq, system: str = "") -> None:
        """
        Initializes the agent with a client, an optional system prompt, 
        and a message history list.
        """
        self.client = client
        self.system = system
        self.messages: list = []
        if self.system:
            self.messages.append({"role": "system", "content": system})

    def __call__(self, message=""):
        """
        Handles the callable behavior of the agent.
        Appends the user query, executes the LLM response, 
        and returns the assistant's output.
        """
        if message:
            self.messages.append({"role": "user", "content": message})
        result = self.execute()
        self.messages.append({"role": "assistant", "content": result})
        return result

    def execute(self):
        """
        Sends the current conversation history to the LLM API and 
        returns the response content.
        """
        completion = self.client.chat.completions.create(
            model="llama3-70b-8192", messages=self.messages
        )
        return completion.choices[0].message.content


# ReAct Agent Loop Function

This `loop` function drives the interaction between the agent and tools in an iterative process. It enables the agent to execute actions based on its responses and adaptively refine its behavior until a conclusion is reached.

---

## Code Breakdown

### **Function Signature**
```python
def loop(max_iterations=10, query: str = ""):


