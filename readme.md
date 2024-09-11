# 如何实现RAG

1. 搜集文献资料 pdf
构造成文件夹树
2. 数据预处理
pdf -> txt  化学领域化学式提取 pdfminer
将提取的 texts 保存到本地 .txt 文件，作为中间结果留存
## 文档切分 将 documents 切分为 chunks
### Fixed size chunking
### Recursive Chunking
### Document Specific Chunking
### Semantic Chunking
SemanticChunker  RecursiveCharacterTextSplitter
3. 构建向量数据库
选择嵌入模型  BAAI/bge-large-en-v1.5
选择向量库 Chorma
将chunks 向量化，并保存进入向量库中，再持久化到本地
4. 检索
-- ChatPromptTemplate
question_template
question_prompt
RunnablePassthrough
StrOutputParser
semantic_rag_chain
semantic_chunk_retriever
-- 将生成的内容格式化为HuggingFace数据集格式
eval_dataset
5. chain 构建
semantic_rag_chain
## 重写-检索-读取
rewrite_retrieve_read_chain = (
    {
        "context": { "x": RunnablePassthrough() } | rewriter | retriever,
        "question": RunnablePassthrough(),
    }
    | base_prompt
    | model
    | StrOutputParser()
)
## step-back提示
step_back_chain = (
    {
        "normal_context": RunnableLambda(lambda x: x["question"]) | retriever,
        "step_back_context": step_back_question_chain | retriever,
        "question": lambda x: x["question"],
    }
    | response_prompt
    | ChatOpenAI(temperature=0)
    | StrOutputParser()
)
llm 选择
llm 词表扩充
## 扩充词表 sentencepiece
按照不同类别的选取一定量的文献资料，要保证每种类别都有足够的样本
使用 sentencepiece 指定好词表大小，字符集覆盖率，分词模型算法类型以及模型的名称等之后，进行训练
会获得 *.model 和 *.vocab 文件
### 合并llm 和 chemical 词表
将合并后的 *.model文件保存到本地
测试对比 llama_tokenizer chinese_llama_tokenizer 的分词结果
6. Ragas评估比较
metrics answer_relavancy, faithfulness, context_recall, context_precision
llm = OpenAI(temperature=0)
embeddings = OpenAIEmbeddings()
eval_dataset
result_df = result.to_pandas()
naive chunker vs semantic chunker
