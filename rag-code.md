
# To run this project, please follow these steps in your terminal:<br>

## To install packages
```
sudo apt install python3-pip
sudo apt install python3.12-venv
```


1. Create a new virtual environment<br>
```
python3 -m venv venv

 ```
<br>
2. Activate the virtual environment:<br>

```
source venv/bin/activate

```
<br>

3. Install the required libraries:<br>
```
pip install langchain langchain-aws faiss-cpu langchain_community
```
<br>

4. Set your AWS credentials:<br>

- Linux/macOS:
```
export AWS_ACCESS_KEY_ID="<your_access_key>"
export AWS_SECRET_ACCESS_KEY="<your_secret_key>"
export AWS_REGION_NAME="us-east-1"
```
- Windows:
```
set AWS_ACCESS_KEY_ID="<your_access_key>"
set AWS_SECRET_ACCESS_KEY="<your_secret_key>"
set AWS_REGION_NAME="us-east-1"
```
5. Run the script:<br>
```
python rag_bedrock.py
```

<br>


## rag_bedrock.py
```
import os
import boto3
from langchain_aws import ChatBedrock, BedrockEmbeddings
from langchain_community.vectorstores import FAISS
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnablePassthrough
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.document import Document

# --- Configuration ---
# You must have your AWS credentials configured, for example, via environment variables:
# export AWS_ACCESS_KEY_ID="your_access_key"
# export AWS_SECRET_ACCESS_KEY="your_secret_key"
# export AWS_REGION_NAME="us-east-1" 
# Ensure you have access to the Bedrock Titan models in your AWS account.

# Define model IDs based on your Bedrock access
TITAN_TEXT_MODEL_ID = "amazon.titan-text-express-v1"
TITAN_EMBEDDING_MODEL_ID = "amazon.titan-embed-text-v1"
# The AWS region needs to be explicitly defined for the clients
AWS_REGION = os.getenv("AWS_REGION_NAME", "us-east-1")


# --- In-Memory Knowledge Base (Corpus) ---
# This is our source knowledge, as you described in your Document Ingestion step.
# In a real application, this would be loaded from external files, a website, etc.
documents = [
    {
        "title": "Introduction to LangChain",
        "content": "LangChain is an open-source orchestration framework designed to simplify the development of applications powered by large language models (LLMs). It provides a modular set of tools to connect LLMs with external data sources and computation. It is available as libraries in Python and JavaScript."
    },
    {
        "title": "LangChain Components",
        "content": "The core components of LangChain include Models (interfaces for LLMs), Prompt Templates (for formatting input), Chains (to link multiple components), Retrieval (for adding external knowledge), Memory (for conversational context), and Agents (for multi-step reasoning)."
    },
    {
        "title": "The Problem Before RAG",
        "content": "Before Retrieval-Augmented Generation (RAG), LLMs faced problems like hallucination, where they would generate factually incorrect information. They also had a knowledge cutoff, as their information was limited to their training data and could not access real-time or private documents. Another issue was the lack of source attribution, making it impossible to verify the claims made by the model."
    },
    {
        "title": "How RAG Works",
        "content": "RAG solves the problems of LLMs by grounding their responses in an external knowledge base. The process involves retrieving relevant document chunks from a corpus and combining them with the user's query. This augmented prompt is then sent to the LLM, ensuring the generated response is based on verifiable, up-to-date information and reduces the chance of hallucination."
    }
]

# --- RAG Setup (Following your 4 steps) ---
def setup_rag_chain():
    """
    Sets up the RAG chain using LangChain, Bedrock, and an in-memory vector store,
    explicitly demonstrating the Indexing, Retrieval, Augmentation, and Generation steps.
    """
    
    # Check if the region environment variable is set.
    if not os.getenv("AWS_REGION_NAME"):
        print("Error: The AWS_REGION_NAME environment variable is not set.")
        return None, None
        
    try:
        print("1. Indexing: Preparing the knowledge base...")
        # Step 1: Indexing
        # a) Document Ingestion -> Our `documents` list is our loaded source knowledge.
        # We convert it to LangChain's Document format.
        docs_for_faiss = [Document(page_content=doc['content'], metadata={'title': doc['title']}) for doc in documents]

        # b) Text Chunking -> Since our docs are small, we skip explicit chunking, but FAISS
        # works on these document chunks. In a real scenario, we'd use a TextSplitter.
        
        # c) Embedding Generation -> BedrockEmbeddings converts text chunks into vectors.
        # The region_name parameter is explicitly passed to ensure proper authentication.
        embeddings = BedrockEmbeddings(
            model_id=TITAN_EMBEDDING_MODEL_ID, 
            region_name=AWS_REGION
        )

        # d) Storage in a vector store -> We use FAISS, an in-memory vector store.
        vectorstore = FAISS.from_documents(docs_for_faiss, embeddings)
        
        retriever = vectorstore.as_retriever(search_kwargs={"k": 2})

        print("   Indexing complete. Knowledge base is ready.")

        # Step 2 & 3: Retrieval & Augmentation
        # These steps are defined in the chain itself. The retriever finds relevant docs,
        # and the prompt template augments the user's query with those docs.
        template = """
        Use the following context to answer the question at the end. If the information is not in the context, just say that you don't know, don't try to make up an answer.

        Context:
        {context}

        Question: {question}
        """
        prompt = ChatPromptTemplate.from_template(template)

        # Step 4: Generation
        # The LLM generates the final response.
        # The region_name parameter is explicitly passed to ensure proper authentication.
        llm = ChatBedrock(
            model_id=TITAN_TEXT_MODEL_ID, 
            region_name=AWS_REGION
        )
        
        # Build the RAG chain using LangChain Expression Language (LCEL)
        rag_chain = (
            {"context": retriever | format_docs, "question": RunnablePassthrough()}
            | prompt
            | llm
            | StrOutputParser()
        )

        return rag_chain, retriever

    except Exception as e:
        print(f"Error setting up the RAG chain: {e}")
        print("Please ensure your AWS credentials are configured and you have Bedrock access.")
        return None, None

def format_docs(docs):
    """
    Formats the retrieved documents into a single string for the context.
    """
    return "\n\n".join(doc.page_content for doc in docs)

# --- Main execution block ---
if __name__ == "__main__":
    rag_chain, retriever = setup_rag_chain()
    
    if rag_chain:
        query = "What is LangChain and what are its components?"
        print(f"\nUser Query: '{query}'")

        # Step 2: Retrieval
        print("2. Retrieval: Finding relevant chunks based on the query...")
        retrieved_docs = retriever.invoke(query)
        print("   Retrieved Documents (Sources):")
        for i, doc in enumerate(retrieved_docs):
            print(f"   {i+1}. **Title:** {doc.metadata['title']}")
            print(f"      **Content:** {doc.page_content[:150]}...")

        # Step 3: Augmentation & Step 4: Generation
        print("3. Augmentation & 4. Generation: Augmenting the prompt and generating the response...")
        response = rag_chain.invoke(query)
        
        print("--- Final Generated Answer ---")
        print(response)
```




