# SDA-bootcamp-project

Stage 6 - **RAG** Chatbot with Chat history **(Dockerization)**

A RAG chatbot using streamlit and FastAPI. At this stage we will add the RAG function to the bot.
At this stage, we will dockerize our backend codes into a docker image and deploy it on the Azure Container App.
The functionality of the codes is same as the codes we were using in stage 8, which store the chat history in the CosmosDB.


For the database, we can still use the `advanced_chats_new` table(the table we used in stage 8):
```
CREATE TABLE IF NOT EXISTS advanced_chats_new (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    -- file_path TEXT NOT null,
    last_update TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    pdf_path TEXT,
    pdf_name TEXT,
    pdf_uuid TEXT
)
```

> **Note:** The codes in this branch is just the showcase that how to interact with CosmosDB, so we **only** store the chat history to the CosmosDB. Actually students can upload all the metadata to the CosmosDB to replace the PostgreSQL. In that case, we also make the database fully serverless.

Since we need to add the CosmosDB connection in the Azure Function, we also need to store the `PROJ-COSMOSDB-ENDPOINT`, `PROJ-COSMOSDB-KEY`, `PROJ-COSMOSDB-DATABASE`, `PROJ-COSMOSDB-CONTAINER` in the **Azure Key Vault**.


And since the front-end is still running on the instance and it needs to connect to the Azure Function APP, so let's store the Function URL in the Azure KeyVault as well.
In this case, to allow the front-end able to load the URL from secret, we need to update the front-end codes a little bit and store the `KEY_VAULT_NAME` in the `.env` file on the instance where we run the front-end.
Please make sure your instance has the permission to load the secret from the KeyVault.

**For the backend, since we will deploy it to the Azure Container App, so we need to set the `KEY_VAULT_NAME` when creating the Azure Container APP. Don't forget to give you Azure Container APP permission to access the Azure Key Vault.**

**And since the front-end codes will connect to the Azure Container APP, we need to store the `PROJ-AZURE-CONTAINER-APP-URL` in the Azure Key Vault as well**

Now, the following secrets should be created in your Azure KeyVault:

```
PROJ-DB-NAME
PROJ-DB-USER
PROJ-DB-PASSWORD
PROJ-DB-HOST
PROJ-DB-PORT
PROJ-OPENAI-API-KEY
PROJ-AZURE-STORAGE-SAS-URL
PROJ-AZURE-STORAGE-CONTAINER
PROJ-CHROMADB-HOST
PROJ-CHROMADB-PORT
PROJ-COSMOSDB-ENDPOINT
PROJ-COSMOSDB-KEY
PROJ-COSMOSDB-DATABASE
PROJ-COSMOSDB-CONTAINER
PROJ-AZURE-CONTAINER-APP-URL
```

PROJ-AZURE-CONTAINER-APP-URL is like `https://dev-aca-sp2.greendune-b96b6884.eastus2.azurecontainerapps.io`

All the requirements for the Docker image are in the `requirements.txt`

All the requirements for the VM are in the `requirements.vm.txt`

To use RAG, we need to start the chromaDB fisrt, using the follow command to start the Chroma server:
```
chroma run --host 0.0.0.0 --path /db_path
```
change `/db_path` to the path you want to store the data, for example: `chromadb`.

And then use 
```
streamlit run chatbot.py
```
to run the streamlit app.
