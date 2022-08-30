---
author: palni
title: "Sequencing Long Text From Sentence Fragments"
date: 2022-08-30T16:28:01-04:00
---

# Sequencing long text from sentence fragments

## What is sequencing?

There are two datasets available from 84,000:

- The raw English translations. This is English text _only_, it is not in any way aligned to the Tibetan.
- Translation memory. This is essentially a dataset of parallel sentence fragments that 84,000 recorded for two purposes: to standardize their translations, and to train machine learning tools. As of this writing, there are approximately 140,000 sentence fragments. Some examples:

| Tibetan | English |
|-|-|
| འདི་སྐད་བདག་གིས་ཐོས་པ་དུས་གཅིག་ན། | thus did i hear at one time: |
| དེ་དག་གིས་ཁྱེད་རྣམས་ལ་གནོད་པ་ཆེན་པོ་བྱེད་པར་འགྱུར་དུ་འོང་གིས། | they have come to bring great harm to you. so: |
| ནི་ར་གད་ཙྪ་ཏ། ནིར་གད་ཙྪ་ཏ། ནི་ར་གད་ཙྪ་ཏ། ནི་ར་ར་གད་ཙྪ་ཏ། ནི་ར་གད་ཙྪ་ཏ། | nirgacchata nirgacchata nirgacchata nirgacchata |
| ཐབས་ལ་མཁས་པ་ཡིན། | skill in methods, |
| སྐྱ་བའི་རྐྱན་གྱིས་རྒ་ཤི་དང་། | conditioned by birth, aging and death come into being, |
| མྱ་ངན་དང་། སྨྲ་སྔགས་འདོན་པ་དང་། | as well as misery, lamentations, |
| སྡུག་བསྔལ་བ་དང་། ཡིད་མི་བདེ་བ་དང་། | suffering, unhappiness, |
| འདི་མེད་པའི་ཕྱིར་འདི་མི་འབྱུང་། | without these conditions, it does not come into being. |

As you can see, while these are curated, well formed sentences, they have several crucial flaws.

- These are not compelete sentences. During fine tuning on translation, pre-trained generative models like BART will attempt to complete these into sentences.
- There is still ambiguity on the Tibetan side. The English sentence fragments are not fully determined by their Tibetan counterparts.

Sequencing is the problem of constructing longer parallel text from these sentence fragments.

## Why is this needed

We are interested in processing long Tibetan texts. When we train a model on the raw sentence fragments and attempt to use it on long texts, we get a model that drops entire sections from the translation. For example, the text

> སྐྱ་བའི་རྐྱན་གྱིས་རྒ་ཤི་དང་། མྱ་ངན་དང་། སྨྲ་སྔགས་འདོན་པ་དང་། སྡུག་བསྔལ་བ་དང་། ཡིད་མི་བདེ་བ་དང་། འདི་མེད་པའི་ཕྱིར་འདི་མི་འབྱུང་།

should translate to 

> Conditioned by birth, aging and death come into being, as well as misery, lamentations, suffering, unhappiness. Without these conditions, they do not come into being.

but instead we get

> Conditioned by birth and death and misery come into being, otherwise they do not come into being.

In order to stand any chance in translating an entire book, we need to find a way to sequence these short segments and fine-tune a model on this dataset.

## Possible approaches

There are two broad categories of approaching this: model based and not.

The model based approaches we tested are:

- NLI: We use an NLI model to find related sentence fragments. We describe this algorithm in more detail below.
- Consecutive NLI: The 84,000 dataset tends to have related sentence fragments following each other. We could use a model to try to identify when consecutive fragments are related so that we can concatenate them.
- Consecutive cosine similarity: This is similar to consecutive NLI, except that instead of using an NLI model we use a BERT-like encoder and compare cosine similarities of consecutive sentence fragments.

The non-model based approaches are:

- In-order: take the same sequence of sentences as in the original 84,000 parallel dataset.
- Follows-anywhere: a sentence fragment can follow another if it ever follows it in the raw English translations corpus, after some minimal preprocessing.

### Using an NLI model to sequence the sentence fragments

In this algorithm, we start by loading an NLI model. In our case we settled on the Hugging Face model name [typeform/distilbert-base-uncased-mnli](https://huggingface.co/typeform/distilbert-base-uncased-mnli). An NLI model classifies pairs of sentences (A, B) into three possible classes: A entails B, B contradicts A, and A and B are unrelated. For further details see the Hugging Face [zero-shot classification pipeline](https://huggingface.co/docs/transformers/v4.21.2/en/main_classes/pipelines#transformers.ZeroShotClassificationPipeline).

Our sequencing algorithm is as follows:

1. Suppose we have so far generated A_i sentence fragments as being consecutive. We are looking for the next consecutive fragment B.
2. Concatenate some lookback window of the last few A_i, in our case 2. Call this segment A.
3. Pick a large set of candidate samples for B, in our case 64.
4. Score the samples for B against the A segment with the NLI model and keep the scores for all three classes.
5. Modify the score logits with a temperature factor, see [temperature sampling](https://nlp.stanford.edu/blog/maximum-likelihood-decoding-with-rnns-the-good-the-bad-and-the-ugly/).
6. Convert the logits to probability scores but multiply by the number of candidate samples. This is to normalize the probability, since it always sums to one. This produces scores in arbitrary units.
7. Apply a cutoff on the scores for the different classes. In our case:
   - Entailment: 1
   - Contradiction: 12
   
   We apply a larger cutoff on the contradiction class since, emprically, we observe that the contradiction classifier is of much lower quality.
8. Sample a contradiction with some probability (in our case 0.1) and entailment otherwise, according to the temperature scaled logits from each of the classes. This is our B.
9. Append this B to the sequence of the A_i and repeat for the length of the concatenation window. We tested windows of lengths between 3 and 10.

In simple terms, we sample the next sentence fragment according to the logits of some NLI model, except that we use temperature sampling and sample occasional contradictions in order to increase the diversity of our results.

The two other variants we tested are:

- Consecutive NLI: instead of sampling a large set of candidate samples for B, simply take the next consecutive sample from the 84,000 dataset. The NLI model is then essentially used to decide whether this sample is admissible as the next sentence or not.
- Consecutive cosine similarity: instead of using an NLI model, use BERT cosine similarities with some threshold (0.05 for us).

### Some detail on the follows-anywhere strategy

The non-model based approaches are mostly self explanatory. The follows-anywhere strategy relies on preprocessing the full English translation texts first:

1. Remove unicode accents from the English text.
2. Remove any consecutive whitespace.
3. Convert to lower case.
4. Remove everything except lower case English letters and spaces.

After this, two sentence fragments are allowed to follow each other if, after the same preprocessing, they follow each other in any full English translation.

## What works and what doesn't, and why

We found that, no matter how we tweaked the model based sequencing strategies, the trained models produced low quality translations. The validation set cross-entropy of the fine-tuned casual language model was approximately 2.1 or so for every version of model-based sequencing. The translations were generally poor quality, compatible with BLEU scores as low as 25.

The in-order non-model based strategy also performed very poorly, with mid-20s BLEU. This is likely due to the many unrelated sentences following each other in the 84,000 data and many sentences being out of order. However, the follows-anywhere strategy worked extremely well, producing models with BLEU in the mid-30s and a much lower validation cross-entropy of 1.9.

This is essentially the famous problem with beam search decoding - the output of text generative models often lacks diversity. Humans write text that contains strategically placed linguistic surprises, whereas maximum likelihood methods like beam search do not. For example, see recent work at <https://aclanthology.org/2021.humeval-1.3/> and <https://aclanthology.org/2021.eacl-main.25.pdf> or older work at <https://arxiv.org/pdf/1610.02424v2.pdf>. While large models like GPT-3 can overcome this problem when generating very long text from short prompts by regurgitating natural-looking training data that they have memorized, such models still produce unnatural translation-ese when generating text of similar length to their prompts.

For example, the following sequence is allowed by the follows-anywhere strategy, while the model-based strategies are much more strict:

| Text | NLI thinks can follow? |
|:-|-|
| Maitreya, how do such bodhisattvas have skill in dedicating merit? | - |
| Well, maitreya, such bodhisattva mahasattvas physical actions are kind, | N |
| their verbal actions are kind, | Y |
| and their mental actions are kind. | Y |
| This, maitreya, is how the bodhisattva mahasattvas have skill in their methods. | N |
| Well, maitreya, such bodhisattva mahasattvas are skilled in conventional truth, | N |
| they are skilled in the absolute truth, | Y |
| they are skilled in both. | N |

When trained on these kinds of sequences, the causal language model overfits very quickly.

## Conclusion

All this suggests that, to train high quality language models, we need training text that satisfies two competing objectives. The training data should be a concatenation of diverse sentence fragments so as to prevent catastrophic memorization by the decoder. However, the sentence fragments have to be semantically related. We were hoping to be able to use semantic relation models, such as NLI, to diversify our training data. Unfortunately, the "understanding" of semantic relationships that such models display is far too limited, at least in the zero shot regime.
