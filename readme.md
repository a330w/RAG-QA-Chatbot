# Medical Document QA System

## Problem Statement
This application is a medical document question-answering system that uses Retrieval-Augmented Generation (RAG) to provide accurate answers from medical documents. It allows users to:
- Upload medical documents in PDF or DOCX formats
- Process and index the documents using BERT embeddings
- Ask questions about the document content
- Receive concise, contextually relevant answers

The system combines document processing, semantic search, and language model capabilities to provide accurate responses while maintaining context from the source documents.

## Development Environment (Google Colab)

### Prerequisites
- A Hugging Face account and API token
- John Snow Labs license keys (for Spark NLP Healthcare)
- Google account for accessing Colab

### Colab Setup
1. Open Google Colab (colab.research.google.com)
2. Create a new notebook or upload the provided `rag_chatbot.py` notebook
3. Ensure you're using a runtime with GPU acceleration for better performance

### Development Dependencies Installation
Execute the following in your Colab notebook:

```python
!pip install -q gradio==3.50.2 pypdf2==3.0.1 python-docx==0.8.11 faiss-cpu==1.7.4 \
    transformers==4.30.2 torch==2.0.1 sentence-transformers==2.2.2 \
    langchain langchain-community ctransformers python-jose redis python-jose \
    pyspark==3.4.0 spark-nlp==${PUBLIC_VERSION} spark-nlp-display pymupdf

!pip install --upgrade -q spark-nlp-jsl==${JSL_VERSION} --extra-index-url https://pypi.johnsnowlabs.com/${SECRET}
```

## Docker Deployment

### Prerequisites
- Docker installed on your system
- Hugging Face API token
- John Snow Labs license keys

### Project Structure
```
medical-qa-system/
├── Dockerfile
├── requirements.txt
├── rag_chatbot.py
├── config/
│   ├── spark_nlp_license.json
│   └── .env
└── README.md
```

### 1. Create Dockerfile
```dockerfile
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY . .

# Expose port for Gradio
EXPOSE 7860

# Run the application
CMD ["python", "rag_chatbot.py"]
```

### 2. Create requirements.txt
```txt
gradio==3.50.2
pypdf2==3.0.1
python-docx==0.8.11
faiss-cpu==1.7.4
transformers==4.30.2
torch==2.0.1
sentence-transformers==2.2.2
langchain
langchain-community
ctransformers
python-jose
redis
pyspark==3.4.0
spark-nlp
spark-nlp-jsl
spark-nlp-display
pymupdf
numpy
pandas
python-dotenv
```

### 3. Environment Configuration

Create a `.env` file in the config directory:
```
HF_TOKEN=your_huggingface_token
SPARK_NLP_LICENSE_PATH=/app/config/spark_nlp_license.json
```

Create `spark_nlp_license.json` in the config directory:
```json
{
    "JSL_VERSION": "your_jsl_version",
    "PUBLIC_VERSION": "your_public_version",
    "SECRET": "your_secret_key"
}
```

### 4. Build and Run Docker Container

Build the image:
```bash
docker build -t medical-qa-system .
```

Run the container:
```bash
docker run -p 7860:7860 \
    --env-file ./config/.env \
    -v $(pwd)/config:/app/config \
    medical-qa-system
```

Access the application at `http://localhost:7860`

## Usage

1. Upload a medical document (PDF or DOCX format)
2. Wait for the "File processed successfully!" message
3. Enter your question in the text input field
4. Click "Get Answer" to receive a response

## Important Notes

### Memory Considerations
- The system requires significant computational resources
- Configure Docker with adequate memory (recommended: at least 8GB)
- Large documents may take longer to process

### Limitations
- Maximum file size depends on available memory
- Supported formats: PDF and DOCX only
- Response time varies based on document size and complexity

### Security Considerations
- Keep API tokens and license keys secure
- Never commit sensitive credentials to version control
- Use environment variables for sensitive information
- Implement proper authentication for production use

## Troubleshooting

### Common Issues
1. Memory errors:
   - Increase Docker memory allocation
   - Process smaller documents
   - Monitor resource usage

2. Model loading issues:
   - Verify Hugging Face token
   - Check internet connectivity
   - Ensure enough disk space

3. License-related errors:
   - Verify Spark NLP license configuration
   - Check license expiration
   - Ensure proper mounting of config directory

### Docker Logs
View container logs:
```bash
docker logs medical-qa-system
```

For detailed logs:
```bash
docker logs medical-qa-system --tail 100 -f
```

## Code Structure and Components

The application follows these major processing steps:

### 1. Document Processing
```python
def handle_file_upload(file):
    # Reads PDF/DOCX files
    # Converts document to text
    # Processes text into embeddings using BERT
    # Creates FAISS index for similarity search
```
- Handles document upload and text extraction
- Uses Spark NLP's BERT embeddings for text representation
- Creates FAISS index for efficient similarity search

### 2. Question Processing
```python
def handle_question(question):
    # Converts question to embeddings
    # Searches similar contexts using FAISS
    # Creates prompt with relevant context
    # Generates answer using language model
```
- Processes user questions
- Uses semantic search to find relevant document sections
- Creates context-aware prompts for the language model

### 3. Core Components
- **Document Processing**: Uses PyMuPDF and python-docx for file parsing
- **Embeddings**: Employs Spark NLP's BertSentenceEmbeddings for text representation
- **Vector Search**: Implements FAISS for efficient similarity search
- **Language Model**: Uses GPT-2/BLOOM for answer generation
- **UI**: Gradio interface for user interaction

### 4. Data Flow
1. Document → Text Extraction
2. Text → BERT Embeddings
3. Embeddings → FAISS Index
4. Question → Similar Context Retrieval
5. Context + Question → LLM Answer Generation

The system combines these components to provide accurate, context-aware responses to medical document queries.