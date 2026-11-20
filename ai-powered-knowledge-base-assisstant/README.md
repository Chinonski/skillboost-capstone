AI-Powered Knowledge Base Assistant (RAG) on n8n
A Retrieval-Augmented Generation (RAG) assistant built with n8n that answers employee questions using a company's own documents. It retrieves the relevant passages from a vector database, answers only from that content, and says so when the information isn't available.
Business Scenario
Novabridge Digital Solutions (a fictional company) keeps information about its services, pricing, HR policies and operations in separate documents. Employees waste time searching through them to answer simple questions. This assistant retrieves the right information and answers in seconds, without inventing facts.
Tech Stack
Component	Tool
Automation / orchestration	n8n
Document source	Google Drive
Vector database	Supabase (Postgres + pgvector)
Embeddings	OpenAI text-embedding-3-small (1536 dimensions)
Chat model	OpenAI gpt-4o-mini
Chat memory	Postgres Chat Memory
Architecture
The project is two pipelines on one n8n canvas.
INGESTION
Google Drive Trigger -> Download file -> Extract from File -> Supabase Vector Store
                                                               |- Embeddings OpenAI
                                                               |- Default Data Loader
                                                                    '- Recursive Character Text Splitter

CHAT
When chat message received -> AI Agent
                               |- OpenAI Chat Model
                               |- Postgres Chat Memory
                               '- Supabase Vector Store1 (retrieval tool)
                                    '- Embeddings OpenAI
What Each Node Does
Ingestion pipeline
Node	What it does	Output
Google Drive Trigger	Watches the knowledge base folder and starts the workflow whenever a new file is created.	File metadata (ID, name, type).
Download file	Downloads the new file from Google Drive using its ID.	The file as binary data.
Extract from File	Converts the downloaded text file into plain text.	The document text in $json.data.
Supabase Vector Store	Runs in Insert Documents mode. Takes each chunk, its embedding and its metadata and writes them to the documents table.	One row per chunk with content, metadata and embedding.
Embeddings OpenAI	Converts each text chunk into a 1536-dimension vector with text-embedding-3-small.	A vector per chunk.
Default Data Loader	Turns the extracted text into document objects and attaches metadata, including the source filename.	Documents ready for splitting and storage.
Recursive Character Text Splitter	Splits the text into chunks of 800 characters with a 150-character overlap, so context isn't lost at chunk boundaries.	Smaller text chunks.
Chat assistant
Node	What it does	Output
When chat message received	Provides the chat interface and passes the user's question to the agent.	The user's message and a session ID.
AI Agent	The brain of the assistant. Follows the system prompt, calls the retrieval tool to search before answering, and writes a response using only the retrieved content.	The final answer shown in the chat.
OpenAI Chat Model	The language model (gpt-4o-mini, low temperature) that reasons over the retrieved text and writes the answer.	Generated text.
Postgres Chat Memory	Stores recent messages in Postgres so follow-up questions are understood in context. Conversations persist across restarts.	Conversation history for the agent.
Supabase Vector Store1	Runs in Retrieve Documents (As Tool for AI Agent) mode. Embeds the agent's search query, calls the match_documents function in Supabase, and returns the most similar chunks.	The top matching chunks and their metadata.
The same embedding model must be used for ingestion and retrieval, otherwise the vectors cannot be compared.
Source Documents
Three fictional company documents are stored in the Google Drive folder:
1.	Services and Pricing: services offered, package prices, payment terms.
2.	Employee Policies: working hours, leave, expense reimbursement, probation, dress code.
3.	Operations and Client Support: support hours and response times, delivery process, refund policy, offices.
Key Design Decisions
•	Low temperature on the chat model to reduce invented answers.
•	Strict system prompt that forces a knowledge base search first and defines an exact fallback reply.
•	Chunk overlap of 150 characters to preserve context across chunk boundaries.
•	Source metadata stored with every chunk so answers can name the document they came from.
•	Google Drive as the source so the knowledge base can be updated by managing files in a shared folder.
•	Postgres Chat Memory instead of in-process memory so conversation history persists.
Screenshots
find links here:
•	screenshots/workflow.png: n8n canvas
•	screenshots/supabase-table.png: documents table with embeddings
•	screenshots/chat-tests.png: chat test results

