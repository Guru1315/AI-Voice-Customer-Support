# AI-Voice-Customer-Support


Streamlit : http://localhost:8501/


1.Configure API Keys

*Get OpenAI API key from OpenAI Platform.

*Get Qdrant API key and URL from Qdrant Cloud.

*Get Firecrawl API key for documentation crawling.

 
2.Use the Interface

Enter API credentials in the sidebar

Input the documentation URL you want to learn about

Select your preferred voice from the dropdown

Click "Initialize System" to process the documentation

Ask questions and receive both text and voice responses


3.Features in Detail:

A)Knowledge Base Creation

Builds a searchable knowledge base from your documentation

Preserves document structure and metadata

Supports multiple page crawling (limited to 5 pages per default configuration)


B)Vector Search:

Uses FastEmbed for generating embeddings

Semantic search capabilities for finding relevant content

Efficient document retrieval using Qdrant


C)Voice Generation:

High-quality text-to-speech using OpenAI's TTS models

Multiple voice options for customization

Natural speech patterns with proper pacing and emphasis
