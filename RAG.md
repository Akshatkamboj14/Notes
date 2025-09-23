# RAG (Retrieval Augmented Generation)
## What were the problems before RAG?

1. **Hallucinations and Factual Inaccuracy**<br>
One of the most notorious problems was hallucination. Since LLMs are pattern-matching systems, not knowledge databases, they sometimes generate text that is plausible and grammatically correct but factually incorrect or completely fabricated. This happens because the model is trained to predict the next token based on statistical patterns, not to verify information against a source of truth. Without a mechanism to retrieve and verify facts, LLMs could confidently present false information, which was a major roadblock for applications requiring high accuracy, like customer support or legal research.

2. **Stale and Limited Knowledge**<br>
LLMs are trained on massive, static datasets that are often months or even years out of date. This creates a knowledge cutoff, meaning the model has no information about events or data that occurred after its training was completed. For example, a model trained in 2023 would have no knowledge of events from 2024 or 2025. This made them useless for tasks that require current information, such as summarising recent news or answering questions about a company's most up-to-date policies.


**These problems can be solved partially by Fine-tuning.<br>**

Fine-tuning is the process of taking a pre-trained model (a foundation model or large language model) and continuing its training on a smaller, task-specific dataset so it performs much better on that task.

**PRE-TRAINING - BTECH<br>
FINE-TUNING - Company specific training**

## **Fine-tuning can solve these problems**
1. **Hallucination**: Fine-tuning on high-quality, domain-specific question→answer pairs teaches the model the correct factual patterns and phrasing for your domain, reducing the kinds of made-up answers it produces for those tasks.

2. **Staleness**: Fine-tuning on new data updates the model’s weights so it can include new facts and behaviours.

## Problems with fine tuning
1. Costly for large models (computationally + time), especially full fine-tuning.(Expensive)

2. One should be technically Expert to carry out fine tuning.

3. Repeatedly fine-tuning against frequently changing data can be very costly.

What is RAG (short definition)

RAG (Retrieval-Augmented Generation) - it is a way to make a language model smarter by giving it extra information at the time you ask your question.

```
Query   -> 
             Prompt -> LLM -> Response
Context -> 

```

RAG = Information Retrivel + text generation 

can be divided into 4 steps,

1. Indexing
2. Retrieval 
3. Augumentation
4. Generation

* **Indexing is when we create an external knowledge base and which we eventually give to prompt. or it is the process of preparing your knowledge base so that it can be efficiently search at query time.** It consists of 4 steps.<br>

    1. **Document Ingestion** -> you load your source knowledge into memory. (now our document is in the string.)

    2. **Text Chunking** -> Break large documents into small, semantically meaning chunks.(LLM's have context limit 4K-32K tokens)

    3. **Embedding Generation** -> convert each chunk into a dense vector (embedding) that captures it's semantic meaning.

    4. **Storage in a vector store** -> Store the vectors along with the original chunk text + metadata in a vector database.

        * Local -> Faiss, Chroma
        * Cloud -> Pinecone, Weaviate




        ![alt text](<image-first.png>)



* **Retrieval is that we understand the query and based on query we derived chunks that can help to solve our query and from that where we derive our context. 0r it is the real time process of finding the most relevent peices of information from a pre-built index(created during indexing) based on the user's question**


    ![alt text](<image-yt.png>)





* **Augumentation is that where we combine our context and user's query to make a prompt.**


    ![alt text](<image copy 2.png>)
    

* **Generation is the final step where LLM uses the user's query and the retireved & augumented context to generate a response**



    ![alt text](<image-final.png>)