# Langchain

## What is Langchain?
It is an open source framework that helps in building LLM based applications. It provides modular components and end-to-end tools that helps developers build complex AI applications, such that chatbots, question-answering systems, RAGs, and more.

1. Supports all Major LLMs. (including hugging face)
2. Simplifies developing LLM based applications. (chains)
3. Integrations available for all major tools. (like aws have wrappers to avoid boiler plate code)
4. Open **Source/Free/Actively developed**. 
5. Supports all major GenAI use cases.


## Problems before langchain

1. Prompt Engineering Was Manual

    - Developers had to hardcode prompts as plain strings in their code.

    - Hard to reuse or structure prompts for different tasks.

    - No standard way to inject variables or templates — you’d just concatenate strings.
    👉 Result: brittle prompts, hard to maintain.

2. No Standard Abstraction Layer for Models

    - Each provider (OpenAI, Hugging Face, Anthropic, Cohere, etc.) had its own API style.

    - If you wanted to switch from GPT-3 to another model, you had to rewrite large parts of code.
    👉 No plug-and-play model switching.

3. Lack of Memory / Context Handling

    - LLMs are stateless. They don’t remember past interactions unless you manually feed the history every time.

    - Developers had to manually build conversation history and re-send it, leading to:

        - Higher token costs 💰

        - Complexity in managing long conversations
    👉 No native conversation memory.

4. Orchestration of Multiple Steps Was Hard

    - Many real-world tasks need multi-step reasoning:

        - Example: search → summarize → format → return answer.

    - Without LangChain, you had to hardcode pipelines of prompts + custom logic.

    - No standard framework for chains or agents to decide next actions.

5. Tool Integration Was Messy

- LLMs often need tools like:

    - Search APIs

    - Calculators

    - Database queries

- Before LangChain, developers had to manually write:

    - Prompt instructions telling the model how to call the tool.

    - Code to intercept responses and run the tool.
    👉 Super ad hoc and fragile.

6. No Vector Store Integration (for RAG)

- Retrieval-Augmented Generation (RAG) needs vector databases (Pinecone, FAISS, Weaviate).

- Before LangChain:

    - Developers had to separately manage embeddings, store them, retrieve top-k matches.

    - Then manually inject results into prompts.
👉 No unified interface to connect LLMs with knowledge bases.

7. Deployment Was Not Standardized

    - Everyone wrote custom glue code to wrap models, prompts, chains, memory, and tools.

    - Hard to test, debug, or share.

👉 Lack of ecosystem → slowed adoption.





### Components

1. Models

    * The core of LangChain — wrappers around LLMs (like OpenAI GPT, Anthropic Claude, Hugging Face models, AWS Bedrock, local LLMs).

    * Provide a standard interface for calling models regardless of the provider.

    * main types:

        * LLMs → take text input, return text output.

        * Chat Models → take structured messages (HumanMessage, AIMessage) and return chat responses.

        * Embedding Models → Convert text into numeric vectors

        * Other custom models (e.g., image, code completion, etc.)

    * LangChain provides abstractions so that you can swap between providers (OpenAI, Anthropic, HuggingFace, etc.) with minimal code changes.



    1. Large Language Models (LLMs)

        - Definition: Predict the next token in a sequence, given input text.

        - Usage: Best for single-turn tasks like summarization, Q&A, classification, text generation.

        - API in LangChain: LLM

        Examples: OpenAI text-davinci-003, Hugging Face transformers, Cohere.


    - Key Features:

        - Input: plain text → Output: plain text.

        - Stateless (do not remember history unless you pass it explicitly).

        - Good for deterministic single queries.

        Code Example:
        ```
        from langchain_openai import OpenAI

        llm = OpenAI(model="text-davinci-003", temperature=0)
        response = llm.invoke("Explain Kubernetes in simple terms.")
        print(response)
        ```

    2. Chat Models

        - Definition: Specialization of LLMs designed for conversational input/output.

        - Usage: Used in multi-turn dialogue, chatbots, agents.

        - API in LangChain: ChatModel (e.g., ChatOpenAI).

        - Examples: OpenAI gpt-3.5-turbo, Anthropic Claude, Google Gemini chat.

        Key Features:

        - Input: structured messages (HumanMessage, AIMessage, SystemMessage).

        - Maintains conversation roles.

        - Better for dialogue reasoning and contextual flows.

        ```
        from langchain_openai import ChatOpenAI
        from langchain_core.messages import HumanMessage, SystemMessage

        chat = ChatOpenAI(model="gpt-3.5-turbo")
        messages = [
            SystemMessage(content="You are a helpful assistant."),
            HumanMessage(content="Explain Kubernetes in 3 bullet points.")
        ]
        response = chat.invoke(messages)
        print(response.content)
        ```



        3. Embedding Models

        - Definition: Convert text into vector embeddings (numerical representations).

        - Usage: Search, semantic similarity, retrieval, clustering, recommendation.

        - API in LangChain: Embeddings.

        - Examples: OpenAI text-embedding-ada-002, HuggingFace sentence-transformers.

        Key Features:

        - Input: text → Output: list of floats (vector).

        - Can be stored in Vector Stores (e.g., Pinecone, Weaviate, FAISS).

        - Used in RAG (Retrieval-Augmented Generation) pipelines.

        ```
        from langchain_openai import OpenAIEmbeddings

        embed = OpenAIEmbeddings(model="text-embedding-ada-002")
        vector = embed.embed_query("Kubernetes is an orchestration system")
        print(len(vector), "dimensions")

        ```

        ## Why Abstraction Matters?

        LangChain unifies all these models into standard interfaces:

        - LLM → .invoke(text)

        - ChatModel → .invoke(messages)

        - Embeddings → .embed_query(text)

        This means you can swap providers (OpenAI → HuggingFace → Anthropic) without rewriting your logic.


    
2. Prompts

    * Templates for structuring queries before sending to the model.

    * Support variables, few-shot examples, and formatting rules.

    * Example:
        ```
        from langchain.prompts import PromptTemplate

        template = PromptTemplate.from_template("Translate this into {lang}: {text}")
        ```
    * Prompts ensure the right instructions are consistently sent to the LLM.


    ## Big picture — what “prompts” mean in LangChain

    A prompt is the text (or structured messages) you send to a model. LangChain treats prompts as first-class objects so you can:

    - Compose and reuse them programmatically,

    - Insert variables safely (no hand-concatenation),

    - Build few-shot / in-context examples,

    - Create multi-message prompts (system + human + assistant) for chat models,

    - Hook output parsers to enforce structured responses.

    There are three common primitives you’ll use:

    - ***PromptTemplate*** — single-string template with input variables.

    - Few-shot prompting — include example input→output pairs in the prompt.

    - ***ChatPromptTemplate*** (multi-message) — role-based templates (System/Human/AI) for chat-style models.

    ### PromptTemplate — single-text templates (what & how)

    What it is\
        A PromptTemplate is a templated string with named variables. You define placeholders and then .format(...) (or equivalent method) to produce the final prompt text.

    Why use it

    - Avoid ad-hoc string concatenation.

    - Clearly specify which variables a prompt needs (helps testing).

    - Reuse / audit prompts easily.

    Key fields

    - template (the string with placeholders like {task} / {context}).

    - input_variables (list of variable names).

    - Optionally: partial / partial_variables to pre-fill some variables.


    ```
    from langchain_core.prompts import PromptTemplate

    template = (
        "You are an expert on Kubernetes. Answer briefly.\n\n"
        "Question: {question}\n\n"
        "Answer:"
    )
    pt = PromptTemplate(template=template, input_variables=["question"])

    filled = pt.format(question="How does a Pod restart policy work?")
    # send `filled` to your LLM
    ```

    Partial / pre-filled variables
    ```
    pt_partial = PromptTemplate(template=template, input_variables=["question", "tone"])
    pt_partial = pt_partial.partial(tone="concise")
    # Now you only need to pass `question` when formatting
    ```


    Best practice

    - Keep prompt templates small and modular; combine templates rather than making one giant string.

    - Validate pt.input_variables at unit-test time.

    - Avoid leaking secrets into templates.


    ### Few-shot prompting — what it is, how to design examples

    What it is\
    Few-shot prompting means placing a small number (1–8 typically) of example input→output pairs in the prompt to show the model the desired behavior. It’s in-context learning — you teach the model by example rather than fine-tuning.

    Structure
    ```
    Instruction / system line
    Example 1:
    Input: ...
    Output: ...

    Example 2:
    Input: ...
    Output: ...

    Now perform:
    Input: <user input>
    Output:
    ```

    Simple example using PromptTemplate
    ```
    few_shot_template = """
    You are a helpful assistant that converts natural language to kubectl commands.

    Example 1:
    Input: "show pods in kube-system"
    Output: kubectl get pods -n kube-system

    Example 2:
    Input: "describe pod nginx-123"
    Output: kubectl describe pod nginx-123

    Now do:
    Input: {user_input}
    Output:
    """
    pt = PromptTemplate(template=few_shot_template, input_variables=["user_input"])
    ```

    How many examples?

    - Too few: model may not generalize well.

    - Too many: token cost grows and context window may be exceeded.

    - Typical sweet spot: 2–6 targeted examples that cover common variants.

    How to choose examples

    - Pick representative examples that cover different edge-cases.

    - Vary phrasing to teach robustness.

    - Keep examples short and focused (avoid lengthy transcripts).

    Dynamic / retrieved few-shot
    Instead of static examples, you can pick examples from a corpus at runtime — e.g., nearest neighbors by embedding similarity (Retrieval-Augmented Example Selection). This dramatically improves accuracy for domain-specific tasks.

    Pitfalls

    - Example leakage: examples with sensitive data will be sent to model every time.

    - Overfitting: if your examples are too similar to each other, the model learns a narrow mapping.


    ### ```ChatPromptTemplate``` — multi-message prompts for chat models

    Why chat templates
    Modern chat models consume role-tagged messages (system/human/assistant). ChatPromptTemplate helps you build that list programmatically with templating for each message.

    Roles

    - SystemMessage — sets behavior/instructions (high priority).

    - HumanMessage — user-provided input or instruction.

    - AIMessage — previous assistant outputs (useful in multi-turn or few-shot where you show assistant responses as examples).

    How it looks (conceptually)
    ```
    System: "You are a helpful assistant that speaks simply."
    Human:  "Translate: {text}"
    ```

    LangChain pattern
    You typically use message templates (e.g., ```SystemMessagePromptTemplate```, ```HumanMessagePromptTemplate```) and then combine them into ```ChatPromptTemplate```.

    Example
    ```
    from langchain_core.prompts import ChatPromptTemplate, SystemMessagePromptTemplate, HumanMessagePromptTemplate
    from langchain_core.messages import HumanMessage, SystemMessage

    system = SystemMessagePromptTemplate.from_template(
        "You are a Kubernetes assistant. Be concise and list only commands."
    )

    human = HumanMessagePromptTemplate.from_template(
        "Convert the following instruction into a kubectl command:\n\n{instruction}"
    )

    chat_prompt = ChatPromptTemplate.from_messages([system, human])
    messages = chat_prompt.format_messages(instruction="get pods in kube-system")
    # messages is a list of message objects ready to be passed to a ChatModel
    ```

    Few-shot in chat form
    Show example exchanges as a sequence:
    ```
    System: you are an assistant...
    Human: Convert X
    AI: kubectl get X
    Human: Convert Y
    AI: kubectl describe Y
    Human: Convert {user_instruction}
    AI:
    ```

    That is more natural for chat models and often yields better behavior than a single monolithic prompt string.

    When to use ChatPromptTemplate

    - When you call a chat model (GPT-style) that expects message-role structure.

    - When you want to separate system role (instruction) from user content.

    - When you want to include previous assistant examples as AIMessage examples for few-shot.


    ### Output formats & Output parsers — make the model reliable

    Problem
    Models are free-form. If you expect JSON or a strict schema, you need to enforce it.

    Solutions

    - Explicit instruction: tell model verbatim to respond with JSON only.

    - Output parsers: LangChain provides utils (e.g., StructuredOutputParser, RegexParser) that parse and validate model output and raise errors if they don’t match.

    Example (JSON parser pattern)
    ```
    # Pseudocode — exact API may vary by LangChain version
    from langchain_core.output_parsers import StructuredOutputParser

    parser = StructuredOutputParser.from_schema({"command": str, "notes": str})
    prompt = PromptTemplate(
        template="Return a JSON object: {{command: ..., notes: ...}}\n\nInstruction: {instr}",
        input_variables=["instr"]
    )
    final_prompt = prompt.format(instr="show pods in kube-system")

    raw = llm(final_prompt)
    parsed = parser.parse(raw)  # raises if not valid JSON / schema mismatch
    ```

    Best practice

    - Ask for JSON and show a short example of the expected schema (one-shot).

    - Combine parser + retry logic: if parse fails, you can re-prompt the model to return valid JSON (or run a second pass to fix format).




3. Chains

    * Compositions of Models + Prompts + Other components.

    * Let you combine multiple steps into one pipeline.

    * Example: Input → Format with prompt → Send to LLM → Parse output.

    * Common chain types:

        1. **LLMChain:** 
            * simplest (prompt + model).
            * The most basic chain.
            * Just a PromptTemplate + LLM → produces output.
            ```
            from langchain.chains import LLMChain
            ```

        2. **SequentialChain:** 
            * multiple steps in sequence.
            * Output of one chain → input of the next.
            ```
            from langchain.chains import SimpleSequentialChain, SequentialChain
            ```

        3. **RouterChain:** routes queries to different sub-chains.

        4. **Parallel Chains** 
            * A Parallel Chain lets you run multiple chains at the same time (in parallel) instead of one after another (sequential).

            * In SequentialChain, output of chain A → input of chain B → chain C, etc. (step by step).

            * In Parallel Chain, multiple sub-chains are run independently on the same input, and then you collect their outputs together.

            ```
            from langchain.chains import LLMChain, SimpleSequentialChain
            from langchain.chains import RunnableParallel
            from langchain.prompts import PromptTemplate
            from langchain_openai import OpenAI

            llm = OpenAI()

            # Chain 1 → summarize text
            summary_prompt = PromptTemplate.from_template("Summarize: {text}")
            summary_chain = LLMChain(llm=llm, prompt=summary_prompt, output_key="summary")

            # Chain 2 → extract keywords
            keywords_prompt = PromptTemplate.from_template("Extract keywords: {text}")
            keywords_chain = LLMChain(llm=llm, prompt=keywords_prompt, output_key="keywords")

            # Parallel chain
            parallel = RunnableParallel(
                summary=summary_chain,
                keywords=keywords_chain
            )

            result = parallel.invoke({"text": "LangChain helps build applications with LLMs by combining models, prompts, and tools."})

            print(result)
            ```


4. Memory




    By default, an LLM is stateless: every prompt is treated independently.
    👉 If you ask:
    ```
    User: Hi, my name is Akshat.  
    User: What’s my name?
    ```

    The second query has no idea about the first unless you carry over the history.

    Memory in LangChain is the mechanism that:

    - Tracks previous conversation turns,

    - Decides how much history to pass into the next LLM call,

    - Manages trade-offs (context length vs relevance).

There are multiple memory classes depending on how you want to store/trim/summarize history.

* Maintains state across multiple LLM calls (like conversation history).

* Useful in chatbots → model remembers previous turns.

* Types:

    - **ConversationBufferMemory** → stores raw history. stores a script of recent messages. great for shorts chats, but can grow large quickly.



    What it is

    - The simplest memory: keeps a full transcript of the conversation as-is.

    - Every new LLM call includes the entire conversation history in the prompt.

    Pros
    ✅ Keeps complete fidelity (no loss of detail).
    ✅ Useful for short conversations, debugging.


    Cons
    ❌ Grows unbounded → risk of hitting context/token limits.
    ❌ Expensive as conversation gets longer.


    ```
    from langchain.memory import ConversationBufferMemory
    from langchain.chains import ConversationChain
    from langchain.chat_models import ChatOpenAI

    llm = ChatOpenAI()
    memory = ConversationBufferMemory()

    conversation = ConversationChain(llm=llm, memory=memory)
    print(conversation.run("Hi, I am Akshat"))
    print(conversation.run("What’s my name?"))
    ```

    👉 The second call remembers “Akshat” because the buffer included the full history.

    - **ConversationBufferWindowMemory** → Only keeps the last N interactions to avoid excessive token usgae. (like last 100)


    What it is

    Like ConversationBufferMemory, but keeps only the last k turns of history.

    Works like a sliding window.

    Pros
    ✅ Prevents context from growing too large.
    ✅ Keeps the most recent context, which is often the most relevant.

    Cons
    ❌ Loses older context permanently (unless stored elsewhere).

    Example

    ```
    from langchain.memory import ConversationBufferWindowMemory

    memory = ConversationBufferWindowMemory(k=3)  # keep last 3 exchanges
    conversation = ConversationChain(llm=llm, memory=memory)

    conversation.run("Hi, my name is Akshat")
    conversation.run("I live in India")
    conversation.run("I love Kubernetes")
    print(conversation.run("What did I just say?"))

    ```

    👉 If k=3, older history (“Hi, my name is Akshat”) is forgotten once new turns exceed the window.

    - **Summarizer-Based Memory** → Periodically summarize older chats segments to keep a condensed memory footprint.

    What it is

    Instead of storing all past messages, it uses the LLM itself to summarize the past into a shorter form.

    Example: instead of replaying 20 long messages, you pass in:
    “Summary so far: Akshat introduced himself, lives in India, loves Kubernetes.”

    Pros
    ✅ Keeps conversation compressed → efficient use of tokens.
    ✅ Captures important context without raw text bloat.
    ✅ Great for long conversations.

    Cons
    ❌ Summaries may lose fine details.
    ❌ Relies on LLM summarization quality.

    Example.
    ```
    from langchain.memory import ConversationSummaryMemory

    memory = ConversationSummaryMemory(llm=llm)
    conversation = ConversationChain(llm=llm, memory=memory)

    conversation.run("Hi, I am Akshat and I am learning LangChain.")
    conversation.run("I am also interested in Kubernetes and Docker.")
    print(conversation.run("What do you know about me?"))
    ```

    👉 Instead of passing the full text, LangChain sends a summary like:
    “Akshat is learning LangChain, interested in Kubernetes and Docker.”

    - **VectorStore-backed memory (a.k.a. Semantic Memory)**

        What it is

        - Stores conversation history as embeddings in a vector database (FAISS, Pinecone, Weaviate, etc.).

        - At query time, retrieves only the most semantically relevant chunks of past conversation.

        Pros
        ✅ Scales to very long conversations (millions of tokens).
        ✅ You don’t blow up the prompt; only fetch what’s relevant.
        ✅ Very powerful for RAG-style chatbots.

        Cons
        ❌ Needs a vector DB backend.
        ❌ Slightly more complex setup (embedding model, retriever, etc.).

        Example
        ```
        from langchain.memory import VectorStoreRetrieverMemory
        from langchain.vectorstores import FAISS
        from langchain.embeddings.openai import OpenAIEmbeddings

        # Create FAISS vectorstore
        embedding = OpenAIEmbeddings()
        vectorstore = FAISS(embedding.embed_query, embedding.dimension)

        # Wrap into memory
        memory = VectorStoreRetrieverMemory(retriever=vectorstore.as_retriever())

        conversation = ConversationChain(llm=llm, memory=memory)

        conversation.run("My favorite programming language is Python.")
        conversation.run("I work on Kubernetes.")
        print(conversation.run("What did I say about tech?"))
        ```

        👉 Instead of dumping full history, it fetches relevant past chunks like “Python” + “Kubernetes” whenever the new query mentions “tech.”



5. Indexes

    * Enable efficient retrieval of relevant context/documents for RAG.

    * Doc loader, Text splitter, vector store, retrievers. -> these make the indexes.

    * Steps:

        * Split documents into chunks.

        * Generate embeddings.

        * Store embeddings in a vector database (FAISS, Pinecone, Chroma).

    * Index = "lookup table" for semantic search → allows the LLM to ground its answers in external data.

6. Agents 

    * The decision-makers in LangChain.

    * Unlike chains (fixed workflow), agents use LLMs to decide dynamically what to do next.

    * Can:

        * Pick which Tool to call (e.g., calculator, web search, DB query).

        * Chain multiple steps with reasoning ("thought → action → observation → next action").

    * Types:

        * Zero-shot agent (decides from instructions).

        * Conversational agent (remembers chat + tools).

    * Example:
        * User: "Find the latest stock price of Tesla and summarize the company news."
        * Agent → uses StockPriceAPI + NewsAPI tools, then combines results and replies.


1) Tools & toolkits (what they are & how to integrate them)

Short definition:
A tool is any external capability you let the model call: search engines, calculators, databases, APIs, shell, or custom functions. A toolkit (or tool collection) is a curated set of tools that an agent can use to solve tasks.

Why use tools

- LLMs are strong at pattern matching and generation, but weak at fresh facts, exact math, and side effects. Tools fill those gaps.

- Tools let agents retrieve real-time info, perform exact computation, or interact with systems (e.g., run commands, call APIs).

Common tool types

- Search / Web (SerpAPI, Bing) — up-to-date facts.

- Calculator — exact arithmetic, deterministic results.

- Knowledge DB / Vector DB retrieval — fetch relevant documents for RAG.

- Code execution / REPL — run Python/R/JS to compute, test snippets.

- APIs — call product APIs, cloud SDKs, services.

- System tools — shell, kubectl, terraform CLIs for infra ops.

- Formatter/Validators — convert/validate outputs (JSON schema, XML).

Tool interface (general pattern)

- Each tool typically implements:

- name (string)

- description (what it does; used by prompts)

- func(input_str) -> output_str (synchronous or async)

- optional: schema for structured I/O

Example (pseudo-Python):
```
def calculator_tool(query: str) -> str:
    # run sanitized math parser or safe eval
    result = safe_eval_math(query)
    return str(result)

tools = [
  Tool(name="calculator", func=calculator_tool, description="Performs math expressions. Input: expression."),
  Tool(name="search", func=serp_search, description="Search the web and return top results.")
]
```


Toolkits

- A toolkit bundles useful tools for a domain (e.g., “DevOps toolkit” = kubectl, helm, aws-cli).

- Use labels and descriptions carefully so the agent picks the right tool.

Best practices

- Clear descriptions: tools are selected by model based on descriptions — be explicit.

- Small, single-purpose tools are easier to use correctly than monolithic ones.

- Sanitize inputs for tools that execute code/commands.

- Rate-limit / caching for expensive tools (web, DB).

- Guardrails: restrict tools that can mutate production (or require review/approval).


2) Zero-shot-REACT agent (what it is, how it works)

Short definition:
REACT = Reasoning + Action. A REACT-style agent interleaves natural language reasoning (internal thoughts) with actions (tool calls) and uses the results to continue reasoning. Zero-shot-REACT means you don't provide many few-shot examples — you rely on a carefully crafted prompt instructing the model to follow the REACT protocol.

REACT pattern (concept)

The model produces alternating blocks like:
```
Thought: I should look this up
Action: search("how many cores in m5.large")
Observation: 2
Thought: Now compute ...
Action: calculator("2 * 3")
Observation: 6
Final Answer: ...
```

Agents parse Action: tokens and call the corresponding tool; they feed Observation: back to the model and continue.

Zero-shot variant

- Instead of showing examples, you provide a directive that tells the model the exact format to follow (Thought/Action/Observation/Final Answer).

- Good when you want to avoid long prompts or maintain flexibility.

Implementation sketch (LangChain-like)
```
from langchain import LLM, AgentExecutor, Tool

tools = [Tool("search", search_func, "Search the web"), Tool("calc", calc_func, "Calculator")]

prompt = """You are an assistant that must use tools.
Follow this exact format:
Thought: <reasoning>
Action: <tool_name>(<input>)
Observation: <result>
... repeat until you have final answer
Final Answer: <answer>
"""

agent = ZeroShotAgent(llm=LLM(...), tools=tools, prompt=prompt)
executor = AgentExecutor(agent, tools)
resp = executor.run("How many minutes in 3.5 days plus 2 hours?")
```
Pros & cons

- Pros: Lightweight, no need for many examples; flexible.

- Cons: Model may deviate from format, hallucinate tool use, or call wrong tool. Requires robust parsing and fallback.

Robustness tips

- Use structured tool calling APIs if provider supports them (OpenAI function-calling, Anthropic tool API).

- Limit the number of allowed action steps or set a max tool-calls budget to avoid loops.

- Validate outputs after each Observation (type-check / parser).

- Add short few-shot examples if zero-shot is brittle.




3) Structured-output agents

Short definition:
Agents that require structured outputs (e.g., strict JSON, known schema) from the model and typically pair the model with a parser/validator. Instead of free-form text, you get guaranteed fields you can program against.

Why structured outputs?

- You want machine-readable results (e.g., {"command": "kubectl get pods", "confidence": 0.9}).

- You want to safely map model decisions to actions (tool calls, API invocations).

- Better for automation, error handling, and auditing.

Common patterns
1) Schema + instruction

- Provide a JSON schema (or explicit field list) in the prompt.

- Ask the model to return only JSON conforming to the schema.

Prompt snippet:
```
Return only JSON with keys:
{"action": "string", "args": {"type":"object"}, "explain":"string"}
```
2) Output parsers

- Use parsers to enforce structure: JSON parsers, regex, or LangChain OutputParser.

- If parsing fails, re-prompt or run a fallback extraction.

3) Function / RPC style (preferred when available)

- Use function-calling capability where the model chooses a named function and returns arguments — the model only returns structured arguments, the system calls the function.

- This avoids brittle string parsing and is safer.

Example (JSON-schema + parser)
```
schema = {
  "type": "object",
  "properties": {
    "tool": {"type": "string"},
    "input": {"type": "string"},
  },
  "required": ["tool","input"]
}

prompt = "Given the question, return JSON strictly matching schema: {...}\nQuestion: {q}"

raw = llm(prompt.format(q=user_q))
data = json.loads(raw)  # wrap in try/except, validate with jsonschema
```
Best practices

- Keep schema minimal and precise.

- Provide a one-line example of valid JSON.

- Use temperature=0 for deterministic outputs.

- Implement a retry-on-parse-fail loop that re-prompts with "Your previous response was not valid JSON; please only return valid JSON matching schema: ..."



4) Multi-step reasoning with tools (planner-runner & decomposition)

Short definition:
Solve complex tasks by decomposing them into smaller steps (a planner), each step possibly calling tools; then execute the steps (a runner). This can be iterative or tree-structured.

Two main styles
A) REACT / interleaved execution (reactive)

- The model reasons, decides next action, calls tool, gets result, continues — dynamic, greedy.

- Pros: flexible, can adapt based on observations.

- Cons: can loop, may be inefficient (repeated tool calls).

B) Plan-and-execute (two-phase)

- Planner: model generates a plan — a list of steps (structured).

- Executor: executes the plan step-by-step (can be another model or a non-LLM controller), calling tools as needed.

- Pros: more predictable, easier to validate and parallelize.

- Cons: planning might be suboptimal; needs robust error handling for failed steps.

Implementation sketch (plan-and-execute)
```
User: "Perform full health check of service X and report remediation steps"

Planner (LLM):
[
  {"step": 1, "action": "run_command", "tool": "kubectl", "input": "get pods -n svcX -o wide"},
  {"step": 2, "action": "parse", "tool": "parser", "input": "look for CrashLoopBackOff"},
  {"step": 3, "action": "run_command", "tool": "describe", "input": "kubectl describe pod ..."}
]

Executor:
- For step in plan:
    call tool
    save observation
    if failure: record and either retry or stop
Return final report compiled from observations
```
Handling errors & retries

- Idempotency: prefer read-only tools or ensure write steps are safe to re-run.

- Retries with backoff for transient failures.

- Fallback plan: if step fails, planner revision (ask model to re-plan using observation).

- Human-in-the-loop: optionally pause for approval before destructive actions.

Parallelization

- Independent steps can be executed in parallel (e.g., check DB, check service status). Keep tool concurrency limits in mind.

Example: multi-step diagnosis

- Planner: determine checks (pods, logs, events, node status).

- Executor: run kubectl get pods, kubectl logs, kubectl get events.

- Aggregator: summarize results and propose remediation.

Safety & governance

- Access control: separate tool permissions by role (read-only vs mutating).

- Audit logs: record every tool call, inputs, outputs, and decisions.

- Rate limits & quotas: protect external services.

- Capability gating: require human approval for privileged operations.



5) Putting it together: full-stack agent pattern

- A practical, production-ready agent often includes:

- Prompt engineering: Clear instructions, format (REACT or plan), and examples.

- Tool registry: well-documented tools with schemas and permission rules.

- Structured outputs: require JSON or function calls.

- Planner + executor: either interleaved REACT or two-phase plan-and-execute.

- Memory / context: feed relevant past info (via vector retrieval or summary).

- Monitoring & audit: log tool calls, latencies, errors, cost.

- Safety: sandbox tools, input sanitization, human checkpoints.



6) Concrete example (pseudo-code using LangChain-style APIs)
```
# tools
tools = [
  Tool("search", search_func, "Search the web (query) -> results_text"),
  Tool("calc", calc_func, "Compute math expressions"),
  Tool("kubectl", kubectl_func, "Run kubectl commands (read-only by default)")
]

# agent prompt (REACT-style)
prompt = """You are an assistant. Use the following format:
Thought: ...
Action: tool_name(args)
Observation: ...
Final Answer: ...
Tools: search, calc, kubectl
"""

agent = ZeroShotAgent(llm=llm, tools=tools, prompt=prompt)
executor = AgentExecutor(agent=agent, tools=tools, max_steps=8)

answer = executor.run("How many pods are not Ready in namespace prod-api and what might cause it?")

```
The AgentExecutor:

- Parses Action: lines and calls corresponding tools.

- Injects Observation: back into the prompt for the next step.

- Stops when model outputs Final Answer: or when max_steps reached.


















In short,

* Models = brains.

* Prompts = instructions.

* Chains = fixed workflows.
  
* Memory = conversation history.

* Indexes = external knowledge.

* Agents = reasoning + dynamic decision-making.



## Temperature

* Temperature is a parameter that controls how random or deterministic the model’s outputs are.

* It’s used during token sampling — when the model decides which word comes next.



### How it works

* At each step, the model predicts probabilities for possible next tokens.

* Low temperature (close to 0):

    * Model picks the highest-probability token almost always.

    * Output is deterministic, focused, and repetitive.

* High temperature (e.g., 1.0 or above):

    * Model samples more randomly from the probability distribution.

    * Output is creative, diverse, but may be less accurate.


### For Example, What happens at different T values

1. Temperature = 0.1 (very low)

    * The exponent **1/T = 1/0.1 =10.**

    * This makes the gap between tokens much bigger.

    * “cat” (0.6) gets boosted way more than others.

    * Result → model almost always picks cat.

2. Temperature = 1.0 (neutral)

    * Exponent **1/T=1**.

    * Probabilities stay the same (0.6, 0.3, 0.1).

    * Result → mostly cat, sometimes dog, rarely elephant.

3. Temperature = 2.0 (high)

    * Exponent **1/T=0.5.**

    * This flattens the distribution (differences shrink).

    * “dog” and “elephant” become more likely relative to “cat”.

    * Result → more variety, sometimes even elephant shows up.


So in this example:

    * T = 0.1 → "cat" nearly always.

    * T = 1.0 → "cat" usually, "dog" sometimes, "elephant" rarely.

    * T = 2.0 → "cat" still common, but "dog" and "elephant" appear much more often.


## Runnable

In LangChain, a Runnable is a standard interface that all components (LLMs, chains, retrievers, tools, prompts, etc.) implement.

Think of it as:
👉 “If it can be executed (run), it’s a Runnable.”

This makes it easy to compose and orchestrate different pieces without worrying about their internal details.



### Key Runnable Methods

1. ```.invoke(input)``` → run once, return output.

    ```
    result = chain.invoke({"question": "What is LangChain?"})
    ```

2. ```.batch([inputs])``` → run on a list of inputs (parallel batch).
    ```
    results = chain.batch([{"question": "Q1"}, {"question": "Q2"}])
    ```
3. ```.stream(input)``` → stream partial results (like tokens from an LLM).

    ```
    for chunk in chain.stream({"question": "Explain AI"}):
    print(chunk, end="")
    ```


### Why Runnables?

Before, LangChain had many different abstractions (Chains, Tools, Retrievers, etc.), and composing them was messy.

Now → everything is just a Runnable.

* Prompts → Runnable

* LLMs → Runnable

* Chains → Runnable

* Tools → Runnable

This allows you to pipe them together in a unified way.




### Runnable Composition

1. RunnableSequence → like SequentialChain
```
from langchain.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain.schema.runnable import RunnableSequence

prompt = ChatPromptTemplate.from_template("Translate to French: {text}")
llm = ChatOpenAI()

chain = RunnableSequence(first=prompt, last=llm)
result = chain.invoke({"text": "Hello, world!"})
```


2. RunnableParallel → like ParallelChain
```
from langchain.schema.runnable import RunnableParallel

parallel = RunnableParallel(
    summary=summary_chain,
    keywords=keywords_chain
)
result = parallel.invoke({"text": "LangChain is an orchestration framework for LLMs."})
```

3. RunnableLambda → wrap Python functions
```
from langchain.schema.runnable import RunnableLambda

def uppercase(x: str) -> str:
    return x.upper()

r = RunnableLambda(uppercase)
print(r.invoke("hello"))  # HELLO
```