# AI Engineering

### Language models

    - A language model encodes statistical information about one or more languages. Intuitively, this information tells us how likely a word is to appear in a given context. 
    - For example, given the context “My favorite color is __”, a language model that encodes English should predict “blue” more often than “car”.

    - The basic unit of a language model is token. A token can be a character, a word, or a part of a word (like -tion), depending on the model.
    - The process of breaking the original text into tokens is called tokenization.
    - The set of all tokens a model can work with is the model’s vocabulary. You can use a small number of tokens to construct a large number of distinct words, similar to how you can use a few letters in the alphabet to construct many words.

    - There are two main types of language models:
        - Masked language model
            - A masked language model is trained to predict missing tokens anywhere in a sequence, using the context from both before and after the missing tokens. In essence, a masked language model is trained to be able to fill in the blank.
            - Masked language models are commonly used for non-generative tasks such as sentiment analysis and text classification. 
            - They are also useful for tasks requiring an understanding of the overall context, such as code debugging, where a model needs to understand both the preceding and following code to identify errors.
        - Autoregressive language model
            - An autoregressive language model is trained to predict the next token in a sequence, using only the preceding tokens.
        
    - A model that can generate open-ended outputs is called generative, hence the term generative AI.

    - Supervision refers to the process of training ML algorithms using labeled data, which can be expensive and slow to obtain. 
    - Self-supervision helps overcome this data labeling bottleneck to create larger datasets for models to learn from, effectively allowing models to scale up. 
    - In self-supervision, instead of requiring explicit labels, the model can infer labels from the input data.
    - Language modeling is self-supervised because each input sequence provides both the labels (tokens to be predicted) and the contexts the model can use to predict these labels.

    - A model’s size is typically measured by its number of parameters. 
    - A parameter is a variable within an ML model that is updated through the training process.

    - A model that can work with more than one data modality is also called a multimodal model. 
    - A generative multimodal model is also called a large multimodal model(LMM).

    - Three Layers of the AI Stack
        - Application development
            - With models readily available, anyone can use them to develop applications. This is the layer that has seen the most action in the last two years, and it is still rapidly evolving. Application development involves providing a model with good prompts and necessary context. This layer requires rigorous evaluation. Good applications also demand good interfaces.
        - Model development
            - This layer provides tooling for developing models, including frameworks for modeling, training, finetuning, and inference optimization. Because data is central to model development, this layer also contains dataset engineering. Model development also requires rigorous evaluation.
        - Infrastructure
            - At the bottom is the stack is infrastructure, which includes tooling for model serving, managing data and compute, and monitoring.

    - Model adaptation techniques can be divided into two categories, depending on whether they require updating model weights.
        - Prompt-based techniques, which include prompt engineering, adapt a model without updating the model weights.
        - Finetuning, on the other hand, requires updating model weights.