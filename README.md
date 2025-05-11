# RAG-Backend

Live url : https://kart-rag-chat-bot.netlify.app

## HLD for the project
<img width="1104" alt="image" src="https://github.com/user-attachments/assets/1631782c-748b-4ad5-b4e3-21d4c6329a7e" />

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

#### Redis

- Firstly a redis server is initialised, UPSTASH redis server provider is used for the project.

```javascript
const { createClient } = require('redis');

// Create a Redis client and connect to Upstash Redis
const redisClient = createClient({
    url: process.env.UPSTASH_REDIS_URL
});
```
- A ```POST``` api named as ```/session``` route is created : This code defines a POST /session route that generates a unique session ID using uuidv4(). It then stores an empty array in Redis with the key session:{sessionId}. The newly created session ID is returned in the response. If an error occurs during the process, a 500 error is sent back.

- A ```POST``` api named as ```/query``` : It is responsible for getting all the related paragraphs from the vector db via the flask api. Sending the query and the paragraphs to gemini and get a response for the query for the provided context. The api also saves the user chat history in redis for future access.

- A ```GET``` api named as ```/history``` : This API is responsible for getting the whole chat history for the given session ID.

- A ```DELETE``` api for deleting the session : This API deleted the values for the given session ID and thus conversation can be deleted and a new session can be started.





