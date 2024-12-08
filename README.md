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

```
# ReAct Agent Loop Function

This `loop` function drives the interaction between the agent and tools in an iterative process. It enables the agent to execute actions based on its responses and adaptively refine its behavior until a conclusion is reached.

---



## `loop` Function Explanation

### Purpose
The `loop` function executes a process in a loop, iterating up to a specified number of times (`max_iterations`) to generate results using an agent. It parses the result of each iteration, checks if any specific actions (like calculating or retrieving data) need to be performed, and updates the prompt accordingly.

### Parameters
- `max_iterations` (int): The maximum number of iterations the loop will execute. The default is 10.
- `query` (str): A string that represents the initial query passed to the agent. Default is an empty string.

### Function Workflow
1. **Agent Initialization**: 
   An agent is created using the provided `client` and `system_prompt`, though their definitions are assumed to be declared elsewhere.

2. **Tools**:
   The function defines a list of tools (`["calculate", "get_planet_mass"]`) that can be called based on the result from the agent.

3. **Iteration Loop**:
   The loop runs for up to `max_iterations`:
   - Each iteration, the agent is called with the current `next_prompt`, and the result is printed.
   - The result is checked for certain keywords (`PAUSE` and `Action`). If these are found:
     - An action is extracted using regular expressions to determine the chosen tool and its argument.
     - If the chosen tool is available in the predefined `tools` list, it evaluates the result of calling that tool (e.g., calculating or getting planet mass).
     - If the tool is not in the list, the prompt is updated to reflect the absence of the tool.
   - If an "Answer" is found in the result, the loop ends.

4. **Regular Expression Usage**:
   The `re.findall` function is used to extract the action from the result if it contains specific information related to an action.

5. **Final Output**:
   The function prints the result and updates the `next_prompt` based on whether the tool was found or not.

### Code:

```python
import re

def loop(max_iterations=10, query: str = ""):
    agent = Agent(client=client, system=system_prompt)
    tools = ["calculate", "get_planet_mass"]
    next_prompt = query
    i = 0

    while i < max_iterations:
        i += 1
        result = agent(next_prompt)
        print(result)

        if "PAUSE" in result and "Action" in result:
            action = re.findall(r"Action: ([a-z_]+): (.+)", result, re.IGNORECASE)
            chosen_tool = action[0][0]
            arg = action[0][1]

            if chosen_tool in tools:
                result_tool = eval(f"{chosen_tool}('{arg}')")
                next_prompt = f"Observation: {result_tool}"
            else:
                next_prompt = "Observation: Tool not found"

            print(next_prompt)
            continue

        if "Answer" in result:
            break

loop(query="What is the mass of Earth plus the mass of Saturn and all of that times 2?")



