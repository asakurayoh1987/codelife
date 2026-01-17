# RAG学习

## 基础

Document(with a created id) -> Paragraphs -> Words -> Chunks(Create Chunk without exceeding chunk size using tokenizer, each with a created id)

The steps of create chunk：

1. **Initialize Variables:** Start with an empty string for the current chunk and an empty list to store all chunks.
2. **Iterate Over Words:** For each word, construct a new chunk by appending the word to the current chunk.
3. **Check Chunk Size:** Use a tokenizer to verify if the new chunk exceeds the desired size.
4. **Update Chunks:** If the new chunk is within the size limit, update the current chunk. Otherwise, append the current chunk to the list of chunks and start a new chunk with the current word.
5. **Final Check:** After processing all words, any remaining text in the current chunk is added to the list of chunks.

The steps of indexing：

1. **Tokenize the Text Chunks:** First, we need to tokenize the text from the chunks to prepare it for model processing.
2. **Generate Embeddings:** Next, we will use the model *“BAAI/bge-small-en-v1.5”* to generate embeddings from the tokenized text.
3. **Map Embeddings:** Once we have the embeddings, we will map them to their respective chunk IDs and document IDs.



## 资料

1. [Building Retrieval Augmented Generation (RAG) From Scratch](https://medium.com/red-buffer/building-retrieval-augmented-generation-rag-from-scratch-74c1cd7ae2c0) - Medium
2. [Building RAG from scrath](https://github.com/MahnoorNauyan/build_rag_from_scratch) - GitHub仓库，有对应的YouTube上的视频