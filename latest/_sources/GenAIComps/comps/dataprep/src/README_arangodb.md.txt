# Dataprep Microservice with ArangoDB

## Table of contents

1. [🚀Start Microservice with Docker](#start-microservice-with-docker)
2. [🚀Consume Dataprep Service](#consume-dataprep-service)

## 🚀Start Microservice with Docker

### Start ArangoDB Server

To launch [ArangoDB](https://github.com/arangodb/arangodb) locally, first ensure you have docker installed. Then, you can launch the database with the following docker command.

```bash
docker run -d -p 8529:8529 -e ARANGO_ROOT_PASSWORD=test arangodb/arangodb:latest
```

### Set Environment Variables

```bash
export no_proxy=${your_no_proxy}
export http_proxy=${your_http_proxy}
export https_proxy=${your_http_proxy}
export ARANGO_URL=${your_arango_url} # e.g. http://localhost:8529
export ARANGO_USERNAME=${your_arango_username} # e.g. root
export ARANGO_PASSWORD=${your_arango_password} # e.g test
export ARANGO_DB_NAME=${your_db_name} # e.g _system
export VLLM_ENDPOINT=${your_vllm_endpoint}
export VLLM_MODEL_ID=${your_vllm_model_id}
export VLLM_API_KEY=${your_vllm_api_key}
export TEI_EMBEDDING_ENDPOINT=${your_tei_embedding_endpoint}
export HF_TOKEN=${your_huggingface_api_token}
```

### Build Docker Image

```bash
cd ~/GenAIComps/
docker build -t opea/dataprep:latest --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy -f comps/dataprep/src/Dockerfile .
```

### Run via CLI

```bash
docker run -d --name="dataprep-arango-service" -p 6007:5000 --ipc=host -e http_proxy=$http_proxy -e https_proxy=$https_proxy -e ARANGODB_URL="http://localhost:8529" -e ... -e DATAPREP_COMPONENT_NAME="OPEA_DATAPREP_ARANGODB" opea/dataprep:latest
```

### Run Docker with Docker Compose

```bash
cd ~/GenAIComps/comps/dataprep/deployment/docker_compose/
docker compose up dataprep-arangodb -d
```

See below for additional environment variables that can be set.

## 🚀Consume Dataprep Service

```bash
curl http://${your_ip}:6007/v1/health_check \
  -X GET \
  -H 'Content-Type: application/json'
```

An ArangoDB Graph is created from the documents provided to the microservice. The microservice will extract entities from the documents and create nodes and relationships in the graph based on the entities extracted. The microservice will also create embeddings for the documents if embedding environment variables are specified.

```bash
curl -X POST \
    -H "Content-Type: multipart/form-data" \
    -F "files=@./file1.txt" \
    http://localhost:6007/v1/dataprep/ingest
```

By default, the microservice will create embeddings for the documents if embedding environment variables are specified.

You can also specify the `chunk_size` and `chunk_overlap` with the following parameters:

```bash
curl -X POST \
    -H "Content-Type: multipart/form-data" \
    -F "files=@./file1.txt" \
    -F "chunk_size=1500" \
    -F "chunk_overlap=100" \
    http://localhost:6007/v1/dataprep/ingest
```

We support table extraction from pdf documents. You can specify `process_table` and `table_strategy` with the following parameters:

- `table_strategy` refers to the strategies to understand tables for table retrieval. As the setting progresses from `"fast"` to `"hq"` to `"llm"`, the focus shifts towards deeper table understanding at the expense of processing speed. The default strategy is `"fast"`.
- `process_table` refers to whether to process tables in the document. The default value is `False`.

Note: If you specify `"table_strategy=llm"`, you should first start the [vLLM Service](https://github.com/opea-project/GenAIComps/tree/main/comps/third_parties/vllm).

```bash
curl -X POST \
    -H "Content-Type: multipart/form-data" \
    -F "files=@./your_file.pdf" \
    -F "process_table=true" \
    -F "table_strategy=hq" \
    http://localhost:6007/v1/dataprep/ingest
```

---

Additional options that can be specified from the environment variables are as follows (default values are in the `arangodb.py` file):

ArangoDB Connection configuration

- `ARANGO_URL`: The URL for the ArangoDB service.
- `ARANGO_USERNAME`: The username for the ArangoDB service.
- `ARANGO_PASSWORD`: The password for the ArangoDB service.
- `ARANGO_DB_NAME`: The name of the database to use for the ArangoDB service.

ArangoDB Graph Insertion configuration

- `ARANGO_INSERT_ASYNC`: If set to True, the microservice will insert the data into ArangoDB asynchronously. Defaults to `False`.
- `ARANGO_BATCH_SIZE`: The batch size for the microservice to insert the data. Defaults to `500`.
- `ARANGO_GRAPH_NAME`: The name of the graph to use/create in ArangoDB Defaults to `GRAPH`.

vLLM Configuration

- `VLLM_API_KEY`: The API key for the vLLM service. Defaults to `"EMPTY"`.
- `VLLM_ENDPOINT`: The endpoint for the VLLM service. Defaults to `http://localhost:80`.
- `VLLM_MODEL_ID`: The model ID for the VLLM service. Defaults to `Intel/neural-chat-7b-v3-3`.
- `VLLM_MAX_NEW_TOKENS`: The maximum number of new tokens to generate. Defaults to `512`.
- `VLLM_TOP_P`: If set to < 1, only the smallest set of most probable tokens with probabilities that add up to top_p or higher are kept for generation. Defaults to `0.9`.
- `VLLM_TEMPERATURE`: The temperature for the sampling. Defaults to `0.8`.
- `VLLM_TIMEOUT`: The timeout for the VLLM service. Defaults to `600`.

Text Embeddings Inferencing Configuration

- `TEI_EMBEDDING_ENDPOINT`: The endpoint for the TEI service.
- `TEI_EMBED_MODEL`: The model to use for the TEI service. Defaults to `BAAI/bge-base-en-v1.5`.
- `HF_TOKEN`: The API token for the Hugging Face Hub.
- `EMBED_CHUNKS`: If set to True, the microservice will embed the chunks. Defaults to `True`.
- `EMBED_NODES`: If set to True, the microservice will embed the nodes extracted from the source documents. Defaults to `True`.
- `EMBED_EDGES`: If set to True, the microservice will embed the edges extracted from the source documents. Defaults to `True`.

OpenAI Configuration:
**Note**: This configuration can replace the VLLM and TEI services for text generation and embeddings.

- `OPENAI_API_KEY`: The API key for the OpenAI service. If not set, the microservice will not use the OpenAI service.
- `OPENAI_CHAT_MODEL`: The chat model to use for the OpenAI service. Defaults to `gpt-4o`.
- `OPENAI_CHAT_TEMPERATURE`: The temperature for the OpenAI service. Defaults to `0`.
- `OPENAI_EMBED_MODEL`: The embedding model to use for the OpenAI service. Defaults to `text-embedding-3-small`.
- `OPENAI_EMBED_DIMENSION`: The embedding dimension for the OpenAI service. Defaults to `768`.
- `OPENAI_CHAT_ENABLED`: If set to True, the microservice will use the OpenAI service for text generation, as long as `OPENAI_API_KEY` is also set. Defaults to `True`.
- `OPENAI_EMBED_ENABLED`: If set to True, the microservice will use the OpenAI service for text embeddings, as long as `OPENAI_API_KEY` is also set. Defaults to `True`.`

[LangChain LLMGraphTransformer](https://api.python.langchain.com/en/latest/graph_transformers/langchain_experimental.graph_transformers.llm.LLMGraphTransformer.html) Configuration:

- `SYSTEM_PROMPT_PATH`: The path to the system prompt text file. This can be used to specify the specific system prompt for the entity extraction and graph generation steps.
- `ALLOWED_NODE_TYPES`: Specifies which node types are allowed in the graph. Defaults to an empty list, allowing all node types.
- `ALLOWED_EDGE_TYPES`: Specifies which edge types are allowed in the graph. Defaults to an empty list, allowing all edge types.
- `NODE_PROPERTIES`: If True, the LLM can extract any node properties from text. Alternatively, a list of valid properties can be provided for the LLM to extract, restricting extraction to those specified. Defaults to `["description"]`.
- `EDGE_PROPERTIES`: If True, the LLM can extract any edge properties from text. Alternatively, a list of valid properties can be provided for the LLM to extract, restricting extraction to those specified. Defaults to `["description"]`.
- `TEXT_CAPITALIZATION_STRATEGY`: The capitalization strategy applied on the node and edge text. Can be "lower", "upper", or "none". Defaults to "none". Useful as a basic Entity Resolution technique to avoid duplicates based on capitalization.
- `INCLUDE_CHUNKS`: If set to True, the microservice will include the chunks of text from the source documents in the graph. Defaults to `True`. If `False`, only the entities and relationships will be included in the graph.

Some of these parameters are also available via parameters in the API call. If set, these will override the equivalent environment variables:

```python
class DataprepRequest(BaseModel):
    ...


class ArangoDBDataprepRequest(DataprepRequest):
    def __init__(
        self,
        files: Optional[Union[UploadFile, List[UploadFile]]] = File(None),
        link_list: Optional[str] = Form(None),
        chunk_size: Optional[int] = Form(1500),
        chunk_overlap: Optional[int] = Form(100),
        process_table: Optional[bool] = Form(False),
        table_strategy: Optional[str] = Form("fast"),
        graph_name: Optional[str] = Form(None),
        insert_async: Optional[bool] = Form(None),
        insert_batch_size: Optional[int] = Form(None),
        embed_nodes: Optional[bool] = Form(None),
        embed_edges: Optional[bool] = Form(None),
        embed_chunks: Optional[bool] = Form(None),
        allowed_node_types: Optional[List[str]] = Form(None),
        allowed_edge_types: Optional[List[str]] = Form(None),
        node_properties: Optional[List[str]] = Form(None),
        edge_properties: Optional[List[str]] = Form(None),
        text_capitalization_strategy: Optional[str] = Form(None),
        include_chunks: Optional[bool] = Form(None),
        ...
```

See the `comps/cores/proto/api_protocol.py` file for more details on the API request and response models.
