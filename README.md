# RAG-Backend

Live url : https://kart-rag-chat-bot.netlify.app

## HLD for the project
![image](https://github.com/user-attachments/assets/c68d88fc-750a-4bac-9023-7f97d7a808e7)

---

## 📌 Creation of Embeddings and Ingestion into ChromaDB

This component of the backend is responsible for fetching news articles, extracting clean content, converting it into embeddings using a SentenceTransformer model, and storing those embeddings in a persistent Chroma vector database for semantic search.

### 📌 Model Used

```python
model = SentenceTransformer('all-MiniLM-L6-v2')
```

#### What it does?

This model converts sentences, paragraphs, or short text into dense vector embeddings (i.e., numerical representations in high-dimensional space) such that semantically similar texts are close together in vector space.

### 📌 Persistent ChromaDB Setup

```python
storage_path = os.path.join(os.getcwd(), "chroma_store")
if not os.path.exists(storage_path):
    os.makedirs(storage_path)

client = chromadb.PersistentClient(path=storage_path)

try:
    collection = client.get_collection("news_articles")
except chromadb.errors.NotFoundError:
    collection = client.create_collection("news_articles")
```

#### What it does?

The above code sets up a persistent local ChromaDB database and ensures a collection named news_articles exists to store text embeddings.

### 📌 NEWS Api

Once the above setup is done, api call is made to NEWS API and it returns a set of URL's of new articles.

### 📌 Convert each article to embeddings

Once article URL's are fetched trafilatura library is used to get the raw HTML content of the page and then this HTML content is converted to embeddings using the SentenceTransformer - all-MiniLM-L6-v2 model.

#### Structure of each entry

```python
collection.add(
    documents=[paragraph],                    
    ids=[document["id"]],                     
    embeddings=[document["embedding"]],        
    metadatas=[{
        "url": document["url"],                
        "text": document["text"]             
    }]
)
```

The above process is done for each document and thus embeddings are inserted into a persisted vector DB

---

### 🎯 Query api to extract top K related paragraphs

This is a flask server which is responsible to extract related paragrphs according the user queries and number of results the payload expects.

#### Overall flow

Once the user query is recieved to the /query api, the user query is converted to embeddings using the SentenceTransformer - all-MiniLM-L6-v2 model. This embedding is then used to query the vector db using the following command.

```python
collection.query(
        query_embeddings=[query_embedding],
        n_results=top_k
    )
```

The returned items are then sent as JSON response.

---

### 🧶 API's to save user sessions, clear session, save and fetch chat history



