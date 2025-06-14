## Development of a PDF-Based Question-Answering Chatbot Using LangChain

### AIM:
To design and implement a question-answering chatbot capable of processing and extracting information from a provided PDF document using LangChain, and to evaluate its effectiveness by testing its responses to diverse queries derived from the document's content.

### PROBLEM STATEMENT:
The objective is to create a chatbot that can intelligently respond to queries based on information extracted from a PDF document. By using LangChain, the chatbot will be able to process the content of the PDF and use a language model to provide relevant answers to user queries. The effectiveness of the chatbot will be evaluated by testing it with various questions related to the document.

### DESIGN STEPS:

#### STEP 1: Initialization
* Input: PDF document path.
* Output: Document loader, embeddings, vector database, prompt, and chain.

#### STEP 2: Load PDF content
1. Import and initialize the PyPDFLoader with the provided PDF file path.
2. Extract the content of the PDF into pages.
   

#### STEP 3: Embed the document chunks
1. Import and initialize OpenAIEmbeddings to generate embeddings.
2. Initialize the Chroma vector database with:
     * Persistent storage directory (persist_directory).
     * The embedding function.
   
#### STEP 4: Define the language model
1. Import and initialize ChatOpenAI with:
     * Model name (gpt-4).
     * Temperature (0) for deterministic responses.

#### STEP 5: Create a Retrieval Prompt
1. Define a concise, user-friendly prompt template to:
     * Use context from the document.
     * Limit answers to three sentences.
     * Encourage polite responses with "Thanks for asking!" at the end.

#### STEP 6: Building the Retreival chain
1. Initialize the RetrievalQA chain by:
     * Specifying the language model (llm).
     * Linking the retriever (vectordb.as_retriever()).
     * Applying the prompt template.
     * Enabling source document return for transparency.

#### STEP 7: Run the query
1. Take a query (question) as input.
2. Pass the query to the qa_chain for processing.
3. Retrieve the result and its associated source documents.

#### STEP 8: Output the Result
1. Print the query (question).
2. Print the chatbot’s answer (result["result"]).

### PROGRAM:

```
from langchain.document_loaders import PyPDFLoader
from langchain.vectorstores import Chroma
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.chat_models import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import RetrievalQA
import os

# Make sure your OPENAI_API_KEY is set in your environment
# export OPENAI_API_KEY="your_api_key_here" on Linux/Mac or set in Windows env variables

# Step 1: Load PDF pages
loader = PyPDFLoader("tech.pdf")
pages = loader.load()

# Step 2: Initialize embeddings and vector store
persist_directory = 'docs/chroma/'
embedding = OpenAIEmbeddings()

vectordb = Chroma(
    collection_name="tech_docs",
    persist_directory=persist_directory,
    embedding_function=embedding
)

# Step 3: Add documents to vector store if empty and persist
if len(vectordb.get()['ids']) == 0:
    vectordb.add_documents(pages)
    vectordb.persist()

# Step 4: Initialize language model
llm = ChatOpenAI(model_name='gpt-4', temperature=0)

# Step 5: Define prompt template
template = """Use the following pieces of context to answer the question at the end. If you don't know the answer, just say that you don't know, don't try to make up an answer. Use three sentences maximum. Keep the answer as concise as possible. Always say "thanks for asking!" at the end of the answer. 
{context}
Question: {question}
Helpful Answer:"""
QA_CHAIN_PROMPT = PromptTemplate(input_variables=["context", "question"], template=template)

# Step 6: Build retrieval QA chain
qa_chain = RetrievalQA.from_chain_type(
    llm,
    retriever=vectordb.as_retriever(),
    return_source_documents=True,
    chain_type_kwargs={"prompt": QA_CHAIN_PROMPT}
)

# Step 7: Ask your question
question = "Is probability a class topic?"
result = qa_chain({"query": question})

# Step 8: Print result
print("Question:", question)
print("Answer:", result["result"])

```

### OUTPUT:
![image](https://github.com/user-attachments/assets/12a8541f-ddf0-4700-8044-f50c573d441c)



### RESULT:
* Prompt: A structured prompt template was designed to pass the document content and user query to the language model.
* Model: OpenAI's GPT model was used to process the input data and provide an answer based on the document's content.
* Output Parsing: The model's output is returned as the answer to the query, ensuring that it provides relevant responses based on the content extracted from the PDF.
