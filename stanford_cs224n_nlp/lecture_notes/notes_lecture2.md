# Lecture 2: Word Vectors and Word Senses

- ### **A look-back of Word2vec**
    - the main idea of word2vec
        - initially start with random word vectors
        - then iterate through each word in the whole corpus
        - at last, it try to predict (in the skip-gram mode) the context using the center word.
        - the algorithm learns word vectors that capture word similarity and meaningful directions in the word space.

          <img src="pics_lecture2/Untitled.png" width = "300" height = "100" alt="d" align=center />

        - it maximizes objective function by putting similar words nearby in space.
        - why called 2vec: there are 2 model variants, Skip-grams and CBOW, word2vec just average both at the end.
    - optimization with word2vec
        - SGD: repeatedly sample windows, and update after each one.
            - input one sample and adjust parameters based on gradient descent.
            - consider there is a very big matrix of word vectors in the whole corpus, if we take gradients of a window of a certain size (let's say 5), the gradient J(\theta) on the whole corpus will be very sparse. In this way, we should update only the words appear in the window, so we only update certain rows in the matrix.
            - negative sampling
                - additional efficiency in training
                - randomly choose some words outside the window as the negative samples, then train the model and update only negative and positive words following the strategy of maximizing the probability of words within the window and minimizing the words randomly chosen.
                - speed up the process and enhance the quality.
                - paper: Distributed Representations of Words and Phrases and their Compositionality
    - can use mini-batch on GPU, e.g. sample 20 from the whole corpus.
    - shuffle before every epoch
- ### **Why not capture co-occurrence counts directly?**
    - co-occurrence matrix: two ways to construct
        - **window based matrix**: use window around each word to capture both positional and semantic (context) information.
        - **word-document based matrix**: use the whole corpus gives the general topic information of the word. It leads to the topic of Latent Semantic Analysis (LSA/LSI).
           - large matrix  
    - window-based co-occurrence matrix
        - window size: 5-10 commonly.
        - symmetric: irrelevant whether left or right context. otherwise it's asymmetric.
        - problems: less robust
            - large in size with large vocabulary
            - high dimension, larger storage required
            - sparsity issue
        - solution: low dimensional vectors
            - store most of the important information in a fixed number of dimensions, which is a dense vector.
            - 25-1000 dimensions commonly, similar to word2vec.
            - methods of dimension reduction
                - matrix decomposition

                    <img src="pics_lecture2/Untitled 1.png" width = "400" height = "200" alt="d" align=center />

                    - SVD: Single Value Decomposition.
                        - 奇异值分解
                        - $X=U*\Sigma*V^T$
                        - observe the singular values (the diago- nal entries in the resulting S matrix), and cut them off at some index k based on the desired percentage variance captured.
                        - retain only k singular best ranked values after decomposition.
                        - the dimensionaliry of the three matrix changes from $|v|*|v|*|v|*|v|*|v|*|v|$ to $|v|*|k|*|k|*|k|*|k|*|V|$, where |k|<|V|
                    - Problems
                        - costy to perform SVD.
                        - requires some hacks on X for the imbalance of word frequency.
                        - matrix is sparse since most words don't co-occur and the dimensionality is large.
                        - new words input may change the dimensionality of the matrix.
                    - Solutions
                        - scaling the counts in the cells.
                            - problem: scale to some words like 'the', 'he', etc. that are too frequent.
                        - ramped windows that count closer words more.
                        - use Pearson correlations instead of counts.
- ### **GloVe**
    - both word2vec and GloVe captures the co-occurrence information to embed the word into a vector.
    - **count based vs. direct prediction: two ways to do word embedding**
        - **direct prediction**
            - a.k.a iteration-based
            - word2vec is an example
            - use window-based training methods are such as SG and CBOW, learning the probability, ability of capturing complex linguistic schema.
        - **count based**
            - a.k.a SVD based (details and optimizations are discussed in the previous chapter)
            - LSA based on counts and decomposes the co-occurrence matrix based on **SVD**.
                - make use of global information.
                - more complex than GloVe.

                <img src="pics_lecture2/Untitled 2.png" width = "500" height = "200" alt="d" align=center />

    - GloVe
        - Global Vectors for Word Representation.
            - GloVe: combines the advantages of two main model families: global matrix decomposition and local context window. The model only trains the non-zero elements in the word word co-occurrence matrix, instead of the whole sparse matrix or large corpus with a single context window, so as to effectively utilize the statistical information. The model generates a vector space with meaningful substructures, and its performance in the most recent word analogy task is 75%. In terms of similarity task and named entity recognition, it is also superior to related models.
            - GloVe uses windows to do counting for each word.
            - Count based models, such as glove, essentially reduce the dimension of co-occurrence matrix and learn low-dimensional dense-vector representation.
            - GloVe pretrained model is based on Common Crawl dataset and well-preprocessed. It's more common used.
        - characteristics of GloVe
            - fast training(low cost), statistical, huge corpus, good performance even corpus and the dimension of vectors are small.
            - used to capture word similarity but disproportionate to large counts.
        - in practice, there are packages like
            - glove for GloVe
            - gensim for word2vec and glove2word2vec
            - spacy
        - the process of training GloVe model
            - **no neural network training in GloVe.**
            - encode meaning in vector differences.
            - first construct word vector and the co-occurrence matrix. use the following formula to approximately represent the relationship with the vector and the matrix.
                - $w_i^Tw_j+b_i+b_j=log(X_{ij})$
                    - X_{ij} stands for the number of appearance of j in the context of i.
                    - 比值代表a和b哪个与x更相关，说明通过概率比例而不是概率本身去学习词向量可能是一个更恰当的方法。
                        - ratios of co-occurrence probabilities can encode meaning components

                            <img src="pics_lecture2/Untitled 3.png" width = "500" height = "100" alt="d" align=center />

                            - so the form of w_i*w_j=logP(i|j) is constructed.
                    - w_i and w_j is the same intuitively in matrix decomposition, we average them as the final vector.
                    - Every word in n_dim is corresponding to a vector in k_dim, so we can use vector_k*vector_k to fit the original vector. that's the idea.
            - then construct the loss function and learn and update.
                - $w_i*w_j=logP(i|j)\\ J=\sum_{i,j=1}^{V}f(X_{ij})(w_i^Tw_j+b_i+b_j-logX_{ij})^2$
                - f(X) is a weight function used as a restriction, the parameter is set to 0.75 after many times of experiments.
                - AdaGrad for optimization
        - conclusion on GloVe: Glove uses global statistics to predict the probability of word J appearing in the context of word I with least squares as the target.
- ### **How to evaluate word vectors?**
    - general evaluation in NLP
        - Intrinsic
            - evaluation on an intermediate task
            - fast to compute
            - not sure of the utility in the real tasks
        - Extrinsic
            - evaluation on a real downstream task
            - time consuming to become accurate enough

    - the Intrinsic way
        - word vector analogies
        - evaluate word vectors by how well their cosine distance after addition captures intuitive semantic and syntactic analogy (类比) questions.
            - semantic
            - syntactic
            <img src="pics_lecture2/Untitled 4.png" width = "400" height = "60" alt="d" align=center />
            <img src="pics_lecture2/Untitled 5.png" width = "400" height = "250" alt="d" align=center />

        - problem: what if the relationship is not linear?
        - analogy evaluation and hyperparameters
            - dimension, 300 is good.
        - another way: measure word vector distances and their correlation with human judgements.
            - example: WordSim353, MC, RG, RW, SCWS. dataset.
    - the Extrinsic way
        - apply directly into a downstream task like named entity recognition.
- ### **Word senses and word sense ambiguity**
    - 歧义，词义消歧
    - cluster word windows around words, retrain with each word assigned to multiple different clusters. So as to refine different meaning of the same word.
        - so $v_{pike}=a_1v_{pike1}+a_2v_{pike_2}+a_3{v_{pike3}}$
        - thesis: 'Improving Word Representations Via Global Context And Multiple Word Prototypes'



- ### **Matetrial 1: GloVe**
    - paper name: *GloVe: Global Vectors for Word Representation*
    - links: [https://nlp.stanford.edu/pubs/glove.pdf](https://nlp.stanford.edu/pubs/glove.pdf)
    - **GloVe: Global Vectors for Word Representation**
        - Two main model families for learning word vectors:
        - global matrix factorization methods
            - LSA: Latent Semantic Analysis, rows correspond to words and columns correspond to the documents in the corpus and the values correspond to the tfidf value or sth based on the co-occurrence characteristics.
            - HAL: Hyperspace Analogue to Language, rows and columns correspond to words and the values correspond to the number of occurrence.
            - they utilize low-rank approximations to decompose large matrices.
            - they leverage the statistical information of the corpus but performs bad at word analogy tasks (an intrinsic evaluation of word vector quality).
            - a main problem may be that some meaningless but frequent words may contribute a bigger amount to the similarity or sth.
        - local context window methods
            - skip-gram model: learn word representations by making predictions with local context windows, train them with an objective.
            - CBOW
            - they behave well on word analogy tasks.
            - it poorly uses the statistics of the corpus (it trains on global co-occurrence counts, no global information)
    - GloVe is a weighted least squares model that trains on global word-word co-occurrence  counts. And performs better on similarity and NER tasks.
    - **the GloVe Model**
        - set up a word-word co-occurrence matrix X whose entries X_ij is the number of times word j occurs in the context (actually within a window).
        - set up the objective function, which is a weighted least squares regression model
        - $J=\sum_{i,j=1}^{V}f(X_{ij})(w_i^Tw_j+b_i+b_j-logX_{ij})^2$
            - adding f(X_ij), which is a cut-off function to avoid the overweighting problem of frequent meaningless words.
            - intuitively the learned vectors in the model: w_i and w_j, are the same because the matrix is supposed to be symmetric, but different actually because of the initialization.
        - model training
            - train by stochastically sampling non-zero elements from X and intialize a learning rate to 0.05 using AdaGrad. Run 50 iterations for vectors smaller than 300 dims and 100 iterations otherwise till convergence.
        - complexity of the model
            - no worse than O(n^2)
            - O(n^0.8), somewhat better than the on-line window-based methods which wcale like O(n).
    - Model Analysis
        - experiments on word analogy, word similarity and name entity recognition tasks, significantly better.
        - vector length and context size
            - symmetric is a context window that extends to the left and right of a target word, while the one which extends only to the left will be called asymmetric.
            - in the results below
                - diminishing returns for vectors larger than 200 dimensions.
                - performances on syntactic context is better with small window size and asymmetric
                - semantic information is more frequently non-local and easy captured with long window size.
                <img src="pics_lecture2/Untitled 6.png" width = "500" height = "200" alt="d" align=center />

            - semantic and syntactic
                - The word analogy task consists of questions like, “a is to b as c is to?” The dataset contains 19,544 such questions, divided into a semantic subset and a syntactic subset. The semantic questions are typically analogies about people or places, like “Athens is to Greece as Berlin is to?”. The syntactic questions are typically analogies about verb tenses or forms of adjectives, for example “dance is to dancing as ﬂy is to ?”.
        - corpus size

            <img src="pics_lecture2/Untitled 7.png" width = "500" height = "200" alt="d" align=center />

            - 300 dimensional vectors trained on different corpora. best overall and subtask-individually on Common Crawl dataset.
            - syntactic subtask increases as the size of corpus increases.
            - semantic subtask don't, maybe because it needs corpus full of knowledge like wikipedia.
        - Runtime
        - outperforms than Word2Vec

            <img src="pics_lecture2/Untitled 8.png" width = "500" height = "200" alt="d" align=center />

            - comparison experiments: the training time is hard to keep, for GloVe is iterations while w2v is the numbers of training epochs. the paper find adding negative samples actually increases the number of training sords and thus analogous to extra epochs.
            - test on the analogy tasks. w2v performs a drawback as the number of negative samples increases, mainly because negative sampling does not approximate the target probability distribution well.

- ### **Material 2: Improving Distributional Similarity with Lessons Learned from Word Embeddings**
    - link: [https://www.aclweb.org/anthology/Q15-1016.pdf](https://www.aclweb.org/anthology/Q15-1016.pdf)
    - The performance improvement of word embedding is largely due to the specific system design selection and super parameter optimization, rather than the word embedding algorithm itself

- ### **Material 3: Evaluation methods for unsupervised word embeddings**
    - link: [https://www.aclweb.org/anthology/D15-1036.pdf](https://www.aclweb.org/anthology/D15-1036.pdf)
    - two main schemes of evaluating the quality of word embeddings
        - extrinsic evaluation
            - use word embeddings in downstream tasks and measure the quality of word embeddings by the performance of downstream tasks.
            - downstream tasks are like name entity recognition, POS-tagging.
            - disadvantage: only one method of evaluation and has uncertainty with other evaluation methods.
        - intrinsic evaluation
            - evaluating word embeddings from semantic and syntactic perspectives.
    - Embedding
        - the mapping of word and the numerical vector.
        - this paper uses cosine similarity to calculate the distance between word embeddings.
        - every experiments are conducted using 6 popular unsupervised word embedding methods
            - CBOW, C&W: motivated by a probabilistic prediction approach, formulate the embedding task as that of ﬁnding a representation that is good at predicting w from the context representations.
            - GloVe, Hellinger PCA(H-PCA), TSCCA(CCA) and Sparse Random Projections: follow a reconstruction approach: word embeddings
            should be able to capture as much relevant information from the original co-occurrence matrix as possible.
    - Intrinsic evaluation
        - absolute intrinsic evaluation
            - embeddings are evaluated individually and only their ﬁnal scores are compared
            - evaluation from 4 following aspects
                - relatedness: relatedness is the cosine similarity between word embeddings. the similarity score is human-defined, Spearman or Person correlation co-efficiency usually.
                - analogy: given (a,b) and (x,y) with a,b,x and map y.
                - categorization: cluster words and classify every word. the similarity score is based on the accuracy of categorization.
                - selected preference: The goal of evaluation is to determine whether a noun is more likely to be the subject or object of a verb.

                <img src="pics_lecture2/Untitled 9.png" width = "500" height = "200" alt="d" align=center />

        - comparative intrinsic evaluation
            - this paper present a new scenario, comparative intrinsic evaluation, in which we ask people directly for their preferences among different embeddings. This paper demonstrate that we can achieve the same results as ofﬂine, absolute metrics using online, comparative metrics.
            - this part of the evaluation uses 100 different query words, and measures the evaluation from
                - the word frequency
                - part of speech
                - whether it is an abstract word
                - KNN (k = 1,5,50)
            - the query word and candidates from models are presented to users and users are asked to **artificially** select the **most similar words** to query word to calculate the score of each model word vector.

              <img src="pics_lecture2/Untitled 10.png" width = "500" height = "200" alt="d" align=center />

    - Coherence 一致性评估
        - in this novel **coherence task** we assess whether groups of words in a small neighborhood in the embedding space are mutually related. (**intrusion task**)
        - in this part, this paper consider that for a query word, two words with the highest semantic similarity and one interference word with unrelated semantics are artificially selected to see whether the above six embedding methods can find irrelevant words from four words, and the data set uses 100 query words with relative intrinsic evaluation.

          <img src="pics_lecture2/Untitled 11.png" width = "300" height = "200" alt="d" align=center />

    - Extrinsic evaluation
        - conclusion: the assumption that there is a globally best word embedding for every downstream task does not hold.
        - conclusion drawn from two tasks: Noun Phrase Chunking and Sentiment Classification.
    - Discussions
        - The word embedding originally only represents the semantic information of words, but the author expresses that the above embedding method more or less contains the word frequency information, and the nearest neighbor of query word is related to the word frequency. At the same time, the rationality of cosine similarity as a measure of word semantic similarity is questioned.
            - Word frequency also interferes with the commonly-used cosine similarity measure.
        - contribution:
            - this paper present a novel evaluation framework based on direct comparisons between embeddings that provides more ﬁne-grained analysis and supports simple, crowdsourced relevance judgments. We also present a novel Coherence task that measures our intuition that neighborhoods in the embedding space should be semantically or syntactically related. We ﬁnd that extrinsic evaluations, although useful for highlighting speciﬁc aspects of embedding performance, should not be used as a proxy for generic quality.
- ### **Material 4: A Latent Variable Model Approach to PMI-based Word Embeddings**
    - link: [https://www.aclweb.org/anthology/Q16-1028.pdf](https://www.aclweb.org/anthology/Q16-1028.pdf)
    - In this paper, the author proposes a latent variable model, which is the first attempt to explain the word class ratio arithmetic strictly.
    - continuous latent variable model is called dimensionality reduction.
    - continuous representation is better representative than discrete representation.
- ### **Material 5: Linear Algebraic Structure ofWord Senses, with Applications to Polysemy**
    - link: [https://transacl.org/ojs/index.php/tacl/article/viewFile/1346/320](https://transacl.org/ojs/index.php/tacl/article/viewFile/1346/320)
    - 对于一词多义的向量表示研究，polysemous
    - It is shown that multiple word senses reside in linear superposition within the word embedding.
        - $V=\sum_{i=1}^{n}\alpha_iW_i+b$
        - where n stands for n kinds of meaning of a polysemous, and sparse coding is learn another vector representation of polysemous words that can represent the original word using a linear formula.
        - Simple sparse coding can recover vectors that approximately capture the senses.
- ### **Material 6: On the Dimensionality ofWord Embedding**
    - link: [https://papers.nips.cc/paper/2018/file/b534ba68236ba543ae44b22bd110a1d6-Paper.pdf](https://papers.nips.cc/paper/2018/file/b534ba68236ba543ae44b22bd110a1d6-Paper.pdf)
    - This paper provides a theoretical understanding of word embeddings and the choice of vector dimensionality
    - Based on the unitary-invariance of word embedding, this paper proposes the Pairwise Inner Product (PIP) loss, a novel metric on the dissimilarity between word embeddings.
    - Using techniques from matrix perturbation theory, we reveal a fundamental bias-variance trade-off in dimensionality selection for word
    embeddings. This may explain the optimal dimensionality selection based on PIP loss.