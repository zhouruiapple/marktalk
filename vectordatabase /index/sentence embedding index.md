# Algorithms of Indexing of Sentence Embeddings

Indexing of Sentence Embedding are used for approximate nearest neighbor search. The goal is to find the vector in the database that is the most similar to the query vector. The similarity is measured by the cosine, dot product, or Euclidean distance between the vectors. The most common algorithms for approximate nearest neighbor search are:

- **LSH(Locality-Sentitive Hashing)**, LSH hashes vectors into a set of "buckets" based on their similarity, and then searches for similar vectors only in one or a few buckets by comparing vectors that belong to the same bucket. It is a probabilistic algorithm, meaning that the buckets are not guaranteed to contain all vectors that are similar to the query vector. However, it is possible to tune the algorithm such that the buckets will contain all similar vectors (at the cost of more computational resources). LSH is a popular algorithm for approximate nearest neighbor search, and is used in many libraries;
- **PQ(Product Quantization)**, PQ is a technique that reduces the dimensionality of the vectors by splitting each vector into subvectors and quantizing each subvector separately. The quantized subvectors are then concatenated into a single vector, which is used as an approximation of the original vector. ~~PQ is a compression technique that divides a high-dimensional vector into smaller subvectors, each of which is quantized into a small number of discrete values. The resulting quantized subvectors are stored in the databases using a tree-based data structure, and a query vector is similarly quantized before being used to search for approximate nearest neighbors.~~
- **PQR(Product Quantization with Refinement)**, 
- **HNSW(Hierarchical Navigable Small World)**, HNSW is a graph-based algorithm that builds a graph of vectors, where each vector is connected to a small number of other vectors. The graph is constructed such that the distance between connected vectors is small, and the distance between non-connected vectors is large. This allows the algorithm to quickly find vectors that are close to the query vector by traversing the graph. HNSW is a deterministic algorithm, meaning that it is guaranteed to find all vectors that are similar to the query vector. However, it is not guaranteed to find the closest vector.
- **IVF(Inverted File)**, IVF is a technique that divides the vectors into a set of clusters, and then searches for similar vectors only in one or a few clusters by comparing vectors that belong to the same cluster. It is a deterministic algorithm, meaning that it is guaranteed to find all vectors that are similar to the query vector. However, it is not guaranteed to find the closest vector.
- **IMI(Inverted Multi-Index)**,
- **K-NN(K-Nearest Neighbor)**, 
- **Annoy**,
- **IVFADC**,

# FAISS Algorithms

**FAISS(Facebook AI Similarity Search)**, FAISS is a library for efficient similarity search and clustering of dense vectors. It contains algorithms that search in sets of vectors of any size, up to ones that possibly do not fit in RAM. It also contains supporting code for evaluation and parameter tuning. FAISS is written in C++ with complete wrappers for Python/numpy. Some of the most useful algorithms implemented in FAISS are:

- **IndexFlatL2**, This is an exhaustive search algorithm that computes the distance between the query vector and all vectors in the database. It is the simplest algorithm, but it is also the slowest. It is useful for testing other algorithms, and for small databases.
- **IndexIVFFlat**, This is an approximate nearest neighbor search algorithm that uses inverted file indices. It is faster than IndexFlatL2, but it is also less accurate. It is useful for large databases.
- **IndexIVFPQ**, This is an approximate nearest neighbor search algorithm that uses inverted file indices and product quantization. It is faster than IndexIVFFlat, and it is also more accurate. It is useful for large databases.
- **IndexHNSWFlat**, This is an approximate nearest neighbor search algorithm that uses hierarchical navigable small world graphs. It is faster than IndexIVFPQ, and it is also more accurate. It is useful for large databases.
- **IndexLSH**, This is an approximate nearest neighbor search algorithm that uses locality-sensitive hashing. It is faster than IndexHNSWFlat, but it is also less accurate. It is useful for large databases.

# Indexing Capability Dimensions

- **Inverted Indexing**,
- **Hierarchical Indexing**,
- **Scalability**,
- **Increamental Indexing**,
- **Customizability**,

# Indexing Comparsion

- **Inverted File Indexing**,
- **Tree-based Indexing**,
- **Hierarchical Indexing**,
- **Graph-based Indexing**,
- **Scalability**,
- **Customizability**,

# Indexing Target

- **Performance**,
- **Scalability**,
- **Customizability**,
- **Cost and Resources**,