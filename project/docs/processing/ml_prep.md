# Machine Learning Preparation

If the processed job posting data is intended for use in Machine Learning (ML) models (e.g., predicting salary, classifying roles, recommending jobs), specific preprocessing steps are typically required beyond the general cleaning and standardization. These steps usually occur *after* the main preprocessing workflow, often within the ML model training pipeline itself.

## Environment

*   These steps are almost always performed **outside** the primary PostgreSQL database, typically within a Python environment using libraries like `pandas`, `scikit-learn`, `NLTK`, `spaCy`, `Hugging Face Transformers`, `PyTorch`, or `TensorFlow`. Data is first queried from PostgreSQL and loaded into this environment (e.g., into a Pandas DataFrame).

## Common ML Preprocessing Steps

### 1. Feature Selection
*   **Action:** Choose the relevant fields (columns) from the processed data that will be used as input features for the ML model.
*   **Considerations:** Select features expected to have predictive power for the target variable. Exclude irrelevant IDs, raw text fields if processed versions exist, or leaky features.

### 2. Text Processing (for `description_text`, `job_title_raw`)

#### a. Further Cleaning
*   Remove stop words (common words like "the", "is", "in").
*   Perform stemming or lemmatization to reduce words to their root forms (e.g., "running" -> "run").

#### b. Tokenization
*   **Action:** Break down text into individual units (tokens), typically words or sub-words.
*   **Tools:** `NLTK`, `spaCy`, `Hugging Face Tokenizers`.

#### c. Vectorization / Embeddings
*   **Action:** Convert textual tokens into numerical representations that ML models can understand.
*   **Methods:**
    *   **Bag-of-Words (BoW):** Simple counts of word occurrences (e.g., `CountVectorizer`).
    *   **TF-IDF (Term Frequency-Inverse Document Frequency):** Weights words based on their importance within a document relative to the entire corpus (e.g., `TfidfVectorizer`).
    *   **Word Embeddings (Pre-trained):** Use dense vector representations learned from large text corpora (e.g., Word2Vec, GloVe, FastText) to capture semantic meaning. Average embeddings or use more complex methods.
    *   **Sentence/Document Embeddings (Transformers):** Use powerful models like BERT, RoBERTa, Sentence-BERT (via `Hugging Face Transformers`, `sentence-transformers`) to generate context-aware embeddings for entire descriptions or titles. These often yield state-of-the-art results for NLP tasks.
*   **Storage:** Generated embeddings might be stored back into PostgreSQL (using `pgvector`) or kept with the feature set during training.

### 3. Categorical Feature Encoding
*   **Action:** Convert non-numerical categorical features (e.g., `employment_type`, `categorized_role`, `location_city` if used as categories) into numerical format.
*   **Methods:**
    *   **One-Hot Encoding:** Creates a new binary (0/1) column for each category level. Can lead to high dimensionality if many unique categories exist (e.g., `pandas.get_dummies`, `sklearn.preprocessing.OneHotEncoder`).
    *   **Label Encoding:** Assigns a unique integer to each category level. Can imply an ordinal relationship that might not exist, often less suitable for nominal features in linear models but sometimes used in tree-based models. (`sklearn.preprocessing.LabelEncoder`).
    *   **Other Methods:** Target Encoding, Hashing Encoding (used for high cardinality features).

### 4. Numerical Feature Scaling
*   **Action:** Scale numerical features (e.g., parsed salary components, derived years of experience) to a common range. This is important for algorithms sensitive to feature magnitudes (e.g., linear models, SVMs, neural networks).
*   **Methods:**
    *   **Standardization (Z-score Scaling):** Rescales data to have a mean of 0 and a standard deviation of 1 (e.g., `sklearn.preprocessing.StandardScaler`).
    *   **Normalization (Min-Max Scaling):** Rescales data to a specific range, typically [0, 1] (e.g., `sklearn.preprocessing.MinMaxScaler`).

### 5. Handling Missing Values (ML Context)
*   While initial handling happens in the main workflow, specific ML-focused imputation strategies might be applied here using `scikit-learn`'s imputers (e.g., `SimpleImputer`, `KNNImputer`) based on the training data split.

### 6. Splitting Data
*   **Action:** Divide the dataset into training, validation, and testing sets to train the model and evaluate its performance on unseen data.

These steps transform the clean, structured data into a numerical format suitable for ingestion by machine learning algorithms. The specific steps and methods chosen depend heavily on the ML task and the selected algorithm.