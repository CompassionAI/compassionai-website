---
author: palni
title: "Word Segmentation and Part-of-Speech Tagging Prototype Results"
date: 2022-08-08T21:52:08-04:00
---

# Word segmentation and part-of-speech tagging prototype performance

## Executive Summary

As part of our prototyping we trained our monolingual model for classical Tibetan to perform a combination of two tasks:

- Word segmentation: break up raw Tibetan text into words. This may seem simple but it is not, since the word boundaries depend on the semantics of the text. This task is similar to the same task in Chinese where high-powered models are also needed.
- Part-of-speech tagging: once the words have been found, label them according to whether they are nouns, verbs, adjectives, etc.

We use the datasets prepared by SOAS. The previous state of the art for these tasks is by Meelen and Hill, 2017, _Segmenting and POS tagging Classical Tibetan using a Memory-Based Tagger_, Himalayan Linguistics, Vol. 16(2): 64-89.

There are two categories of words to consider when evaluating such models. Words seen during training are less important, all models should perform very well on such words. The hard part is to extrapolate from the training data to words that are not in the training set. The model of Meelen and Hill generally works well on training words, and ours does too. However, for new words our model is dramatically better:

|                          | Meelen-Hill | Ours | Improvement |
|--------------------------|------------:|-----:|------------:|
| _Word segmentation_      |         47% |  93% |      +46 pp |
| _Part-of-speech tagging_ |         40% |  64% | +24 pp      |

{{% caption %}}
Table: accuracy[^1] comparison for words not seen during training
{{% /caption %}}

[^1]: Weighted average F1 scores of various token classifiers. See the rest of the post below for additional details.

The cause of this improvement is that our model has seen the entire Kangyur and Tengyur during pre-training, which allows it to extrapolate to unseen, but similar, words. This is very important, since labeling all Tibetan words in all contexts is practically impossible.

Our work is only a prototype. If we receive funds to pre-train better models that are richer and align better with their tasks, we will get better performance, potentially much better. We can also incorporate other similar tasks. The most important is named entity recognition - find the names in a long string of text, for example “and great awakening being <u>one who always watches</u>[name] answered, <u>shariputra</u>[name], anyone who wishes to practice[…]”.

## Additional details

### Previous state of the art

We will compare our model against the previous state of the art (as far we can tell), which is the work by Meelen and Hill, 2017, _Segmenting and POS tagging Classical Tibetan using a Memory-Based Tagger_, Himalayan Linguistics, Vol. 16(2): 64-89.

We use their published dataset for fine tuning our pre-trained model. We use the eKangyur and the OpenPecha Tengyur to pre-train our model. Details are available upon request.

We have had to modify their reported performance numbers slightly to make them consistent with the standard way of reporting performance for token classifiers and comparable with our own report. See notes section below for details.

### Basic CompassionAI dataset statistics

|                              | Test data | Training data |
|------------------------------|----------:|--------------:|
| _Split fraction_             |       10% |           90% |
| _# examples_                 |    28,167 |         3,129 |
| _# unique words_             |     5,703 |        20,628 |
| _# words not in other split_ |     1,261 |        16,186 |

We will refer to the words that appear in test data but not in training data as out-of-vocabulary words. Meelen and Hill refer to them as unseen words.

Below we plot the token length distributions for the words in the training and test sets. As we can see, the distributions are similar.

![Figure: distribution of lengths of tokenized words, in SentencePiece tokens, in the training set](/images/results/word-segmentation-and-part-of-speech-tagging-prototype-results/image1.png)

{{% caption %}}
Figure: distribution of lengths of tokenized words, in SentencePiece tokens, in the training set
{{% /caption %}}

![Figure: distribution of lengths of tokenized words, in SentencePiece tokens, in the test set](/images/results/word-segmentation-and-part-of-speech-tagging-prototype-results/image1.png)

{{% caption %}}
Figure: distribution of lengths of tokenized words, in SentencePiece tokens, in the test set
{{% /caption %}}

### Word segmentation

To report performance on word segmentation, we report performance on the tags that indicate that a token belongs to the middle of a word:
In Meelen and Hill, we combine the categories M (middle of word), E (end of word), and ES (end of word + single syllable) of their small tag set on page 74.
In our system, we report the performance on the ‘MIDDLE OF WORD’ token category.
We report performance from Meelen and Hill on the small tag set. We present the support (their frequency) weighted averages of their precision, recall and F1 scores. This gives the correct recall of the combined tags, but there is no way to infer the correct precision. We report the support as a percentage of the total token count in the test sets, this makes it comparable.

|                   | Precision | Recall | F1       | Support |
|-------------------|----------:|-------:|---------:|--------:|
| _Meelen and Hill_ |      0.88 |   0.86 | 0.87     |     24% |
| _Ours_            |      0.96 |   0.97 | 0.97     |     32% |
| ***Improvement*** |  **0.08** |**0.11**| **0.10** |       - |

{{% caption %}}
Table: performance on tags indicating the middle of a word, in-vocabulary words
{{% /caption %}}

|                 | Precision | Recall | F1   | Support |
|-----------------|----------:|-------:|-----:|--------:|
| Meelen and Hill |      0.51 |   0.43 | 0.47 |     29% |
| Ours            |      0.97 |   0.90 | 0.93 |     66% |
| Improvement     |      0.46 |   0.47 | 0.46 |       - | 

{{% caption %}}
Table: performance on tags indicating the middle of a word, out-of-vocabulary words
{{% /caption %}}

NB: we split our test set along sections, not words - we think it is a mistake to split on words, we think it causes training data to leak into the test set. Thus, for us, short words are much more likely to be in the training set. This causes the difference in supports for the out-of-vocabulary words. This makes our out-of-vocabulary test significantly harder than Meelen and Hill and we still outperform them. Details upon request.

We see meaningfully better performance even on the in-vocabulary words. This is surprising, and may indicate training problems with the system of Meelen and Hill. It may also be due to their word segmentation operating somewhat differently from ours, with a much more elaborate (and hence difficult to learn) tagging scheme.

What is less surprising is the enormous improvement we see in the out-of-vocabulary performance. This is due to the pre-training - for Meelen and Hill, the out-of-vocabulary words are completely new, whereas our model benefits from having seen these words during pre-training in the context of the Cloze “fill-in-the-blanks” task.

### Part-of-speech tagging

We report summarized performance that makes the overall trend clear. Detailed performance tables are available upon request.

- Meelen and Hill use their set of 79 morphosyntactic tags. We report the support weighted averages of their top 10 tables on page 78 of their paper. We report the total support of their top 10 tags by aggregating the complete table in their appendix.
- We use the Universal Dependencies simplification of the Meelen and Hill tags due to Esukhia, see their data at <https://github.com/Esukhia/Corpora/tree/master/POS>. There are 15 unique tags in their dataset. When averaging, we take all tags, not only the top 10. Note that, since performance is generally worse on less frequent tags, this disadvantages our reported performance.

|                          | Precision | Recall    | F1        | Support |
|--------------------------|----------:|----------:|----------:|--------:|
| Meelen and Hill (top 10) |      0.97 |      0.98 |      0.98 |     59% |
| Ours (all)               |      0.94 |      0.94 |      0.94 |    100% |
| ***Improvement***        | **-0.03** | **-0.04** | **-0.04** |       - | 

{{% caption %}}
Table: support weighted average performance on part-of-speech tags, in-vocabulary words
{{% /caption %}}

|                          | Precision | Recall   | F1       | Support |
|--------------------------|----------:|---------:|---------:|--------:|
| Meelen and Hill (top 10) |      0.40 |     0.42 | 0.40     |     91% |
| Ours (all)               |      0.63 |     0.69 | 0.64     |    100% |
| ***Improvement***        |  **0.23** | **0.27** | **0.24** |       - | 

{{% caption %}}
Table: support weighted average performance on part-of-speech tags, out-of-vocabulary words
{{% /caption %}}

While the in-vocabulary performance is slightly worse, both models perform in the high-90% range here. The difference is likely due to the fact that we report the performance on all tags corresponding to a word, while Meelen and Hill (appear to) report performance on pre-segmented words. Essentially, our part-of-speech tagging does not assume that we already know the (unknown) true words, while Meelen and Hill do appear to assume this.

The tremendous benefit of pre-training is clearly visible in the out-of-vocabulary generalization. We see a large lift across the board for the pre-trained model, with all performance metrics improving by >50% despite the more challenging testing protocol. We think this is a very promising performance. We expect that an investment in more careful pre-training will result in further improvement.

### Notes

#### Model choice

Our model is a pre-trained monolingual transformer, trained on SentencePiece tokens, that’s been “forced” to be a morpheme level model (details available upon request). This definitely reduces the performance significantly, especially on the out-of-vocabulary words. We went down this road for two reasons:

- We need a SentencePiece intra-morpheme tokenized model for our translation experiments. For technical reasons about what transformers tend to learn well, for small datasets like Tibetan, intra-morpheme models are likely to work significantly better for tasks like translation than morpheme level models (details upon request).
- Pre-training a second, morpheme-level transformer is expensive. We used stochastic tokenization (details upon request) when training the SentencePiece transformer, so while we expect significant performance degradation on out-of-vocabulary words, we still expect significant generalization as well.

#### Performance reporting

When reporting the performance of the Meelen and Hill work, we renamed their “Frequency” column to “Support”. Support is the established term for what they’re calculating and the term “frequency” implies some kind of rate.

A more significant difference is that we normalized their “Frequency” column by the number of tokens (according to their tokenization scheme) in their training and test sets. We also normalized our supports by the number of tokens (according to our tokenization scheme) in our training and test sets. This makes the reported numbers have similar meanings and be comparable across experiments.
