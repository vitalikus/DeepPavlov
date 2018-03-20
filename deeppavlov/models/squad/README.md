# Question Answering Model for SQuAD dataset

## Task definition
Question Answering on SQuAD dataset is a task to find an answer on question in a given context (e.g, paragraph from Wikipedia), where the answer to each
question is a segment of the context:

Context:
> In meteorology, precipitation is any product of the condensation of atmospheric water vapor that falls under gravity. The main forms of precipitation include drizzle, rain, sleet, snow, graupel and hail... Precipitation forms as smaller droplets coalesce via collision with other rain drops or ice crystals **within a cloud**. Short, intense periods of rain in scattered locations are called “showers”.


Question:
> Where do water droplets collide with ice crystals to form precipitation?


Answer:
> within a cloud

Datasets, which follow this task format:
* Stanford Question Answering Dataset ([SQuAD](https://rajpurkar.github.io/SQuAD-explorer/)) (EN)
* [SDSJ Task B](https://www.sdsj.ru/ru/contest.html) (RU)

## Model
Question Answering Model is based on R-Net, proposed by Microsoft Research Asia ("R-NET: Machine Reading Comprehension with Self-matching Networks" [[link]](https://www.microsoft.com/en-us/research/publication/mrc/)) and its realization by Wenxuan Zhou [[link]](https://github.com/HKUST-KnowComp/R-Net).
  
## Configuration
### Config components
* **squad_dataset_reader** - downloads and reads SQuAD dataset
  * data_path - path to save dataset
* **squad_iterator** - create batches from SQuAD dataset
* **squad_preprocessor** - preprocesses context, question by cleaning data and tokenizing
  * in: **c_raw**, **q_raw** - not processed contexts and questions
  * out: 
    * **c** - processed context (cleaned unicode symbols and quoting)
    * **c_tokens** - tokenized context
    * **c_chars** - tokenized context split on chars
    * **c_r2p** - mapping from raw context to processed context
    * **c_p2r** - mapping from processed context to raw context
    * **q** - processed question
    * **q_tokens** - tokenized question
    * **q_chars** - tokenized question split on chars
    * **spans** - mapping from word indices to position in text
  * context_limit - maximum length of context in words
  * question_limit - maximum length of question in words
  * char_limit - maximum number of chars in token
* **squad_ans_preprocessor** - preprocesses answer
  * in:
    * **ans_raw** - not processed answer
    * **ans_raw_start** - start position of not processed answer in context
    * **c_r2p**
    * **spans**
  * out: 
    * **ans** - processed answer
    * **ans_start** - start position of processed answer
    * **ans_end** - end position of processed answer
* **squad_vocab_embedder** - builds vocabulary and embedding matrix
  * in:
    * **c_tokens**
    * **q_tokens**
  * out
    * **c_tokens_idxs**
    * **q_tokens_idxs**
  * fit_on: **c_tokens** and **q_tokens**
  * level - token or char
  * emb_folder - path to store pretrained embeddings
  * emb_url - url to donwload embeddings
  * save_path - path to save vocabulary and embedding matrix
  * load_path - path to load vocabulary and embedding matrix
  * context_limit - maximum length of context in words
  * question_limit - maximum length of question in words
  * char_limit - maximum number of chars in token
* squad_model - model to find answer on question in context  
  * in: **c_tokens_idxs**, **c_chars_idxs**, **q_tokens_idxs**, **q_chars_idxs**
  * in_y: **ans_start**, **ans_end**
  * out:
    * **ans_start_predicted** - start position of predicted answer
    * **ans_end_predicted** - end position of predicted answer
  * word_emb - pretrained word embeddings
  * char_emb - pretrained char embeddings
  * context_limit - maximum length of context in words
  * question_limit - maximum length of question in words
  * char_limit - maximum number of chars in token
  * train_char_emb - update char_emb during training or not
  * char_hidden_size - size of word embedding built on characters
  * encoder_hidden_size - hidden size of encoder cells 
  * attention_hidden_size - hidden size to use to compute attention
  * learning_rate
  * keep_prob - dropout keep probability
  * grad_clip - gradient clipping value
  * save_path
  * load_path
* squad_ans_postprocessor - extracts predicted answer from context
 * in: **ans_start_predicted**, **ans_end_predicted**, **c_raw**, **c_p2r**, **spans**
 * out: 
   * **ans_predicted** - text of predicted answer in raw context
   * **ans_start_predicted** - start position of predicted answer in raw context
   * **ans_end_predicted** - end position of predicted answer in raw context

### Config file
Default config, which could be found at `deeppavlov/configs/squad/squad.json`

## Running model
**Tensorflow-1.4.0 with GPU support is required** to run this model.
## Training
**Warning**: training with default config requires about 10Gb on  GPU. Run following command to train the model:  
```bash
python -m deeppavlov.deep train deeppavlov/configs/squad/squad.json
```
## Interact mode
Interact mode provides command line interface to already trained model.

To run model in interact mode run the following command:
 ```bash
python -m deeppavlov.deep interact deeppavlov/configs/squad/squad.json
```
Model will ask you to type in context and question.

## Pretrained model on SQuAD
Model is available at the following [[link]](lnsigo.mipt.ru/export/deeppavlov_data/squad_model.tar.gz)

It achieves ~79 F-1 score and ~70 EM on dev set. 

## Training on SDSJ Task B
If you want to train this model on SDSJ Task B then you should follow these steps:
* Convert data to SQuAD format
* Use Russian [word](http://lnsigo.mipt.ru/export/embeddings/ft_native_300_ru_wiki_lenta_nltk_word_tokenize/ft_native_300_ru_wiki_lenta_nltk_word_tokenize.vec) and [character](http://lnsigo.mipt.ru/export/embeddings/ft_native_300_ru_wiki_lenta_nltk_word_tokenize-char.vec) embeddings