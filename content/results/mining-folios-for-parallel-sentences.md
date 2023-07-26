---
author: palni
title: "Mining Folios for Parallel Sentences"
date: 2023-07-25T15:30:00-04:00
---

# Mining folios for parallel sentences

## Two available datasets

As of now, 84,000 publishes two datasets:

- Parallel folios. This takes the form of their translations in XML format, with folio boundary markers inserted as hypertext. These folio markers can be used to approximately align these parallel folios to the Kangyur and Tengyur (although the translations are almost entirely from the Kangyur as of now).
- Parallel sentences. These are manually extracted short Tibetan sentences, consisting of potentially more than one section, with English translation.

The parallel sentences are ideal for training a low-resource machine translation model. They appear to not be excessively biased in their dataset construction methodology. Unfortunately, 84,000 appears to have stopped updating this dataset as they press on with their translation efforts. The parallel folios dataset continues to be updated, but it is problematic for two reasons:

- The alignment with the Tibetan folios is only approximate.
- The much more serious problem is that there is only a small number (<20k in our cleaned dataset as of this writing) of these folios, while each folio consists of a relatively long segment of Tibetan text that is very different from the short sentences that [models like NLLB]({{< ref "results/no-language-left-behind.md" >}}) are trained on.

## Long models in the low-resource setting?

One way to attempt to resolve at least the second, serious difficulty of the folios consisting of long Tibetan segments is to use a long model. Unfortunately, this is unlikely to work, as we explain in this section.

As of today, there are no long models with acceptable multilingual performance on Tibetan. For example, ChatGPT produces results like this in the zero-shot regime:

> **Prompt:** Please translate the following from classical literary Tibetan to English, keeping in mind that this is a Buddhist text: ཀུན་དགའ་བོ་དེ་ལྟ་བས་ན་རྣམ་གྲངས་དེས་ན་ཅི་ནས་བྱང་ཆུབ་སེམས་དཔའ་སེམས་དཔའ་ཆེན་པོ་ཆོས་དེ་དག་ལ་གནས་ཤིང་འདི་ལྟ་སྟེ། །ཆོས་རྣམས་ཀྱིས་དོན་བྱེད་པ་ཡིན་པ་དེ་ལྟ་བུར་རིག་པར་བྱའོ།
>
> **ChatGPT:**: In order to provide an accurate translation, it's important to note that classical literary Tibetan can be quite nuanced and context-dependent. However, based on the provided text and keeping in mind its Buddhist nature, here's a translation:
>
> "KunDga bo de lta bsa na rnam grangs des na ci nas byang chub sems dpa' sems dpa' chen po chen po chen po chos de dag la gnas shing 'di lta ste // chos rnam sras kyi don byed pa yin pa de lta bur rig par bya'o."
>
> Please note that there may be alternative interpretations depending on the context and the intended meaning.

We tried to engineer the prompt with the [Chain-of-Dictionary technique](https://arxiv.org/abs/2305.06575). The results simply regurgitate minimal grammatical sentences that contain the hints given to the model and have, for all practical purposes, zero accuracy:

> **Prompt:** Please translate the following from classical literary Tibetan to English, keeping in mind that this is a Buddhist text: ཀུན་དགའ་བོ་དེ་ལྟ་བས་ན་རྣམ་གྲངས་དེས་ན་ཅི་ནས་བྱང་ཆུབ་སེམས་དཔའ་སེམས་དཔའ་ཆེན་པོ་ཆོས་དེ་དག་ལ་གནས་ཤིང་འདི་ལྟ་སྟེ། །ཆོས་རྣམས་ཀྱིས་དོན་བྱེད་པ་ཡིན་པ་དེ་ལྟ་བུར་རིག་པར་བྱའོ།
>
> The translation starts with "that is why, ananda, according to that explanation,"
>
> Here are some relevant dictionary entries to help you:
>
> བྱང་ཆུབ་སེམས་དཔའ་  means "bodhisattva" <br>
> ཆོས་རྣམས་ཀྱིས means "phenomena"
>
> **ChatGPT:**: "That is why, Ananda, according to that explanation, since phenomena are such, the bodhisattva."
>
> Please note that the translation provided here is based on the given phrases and their meanings from the dictionary entries you provided. However, it's essential to consider the broader context and consult authoritative sources to ensure accuracy in translating classical literary Tibetan.

Note that the Chain-of-Dictionary paper, when translating into English, provides the model with a dictionary of key words *in the English target sentence*, rendering its results for English as the target meaningless.

There does not appear to be any way to improve these results with ChatGPT. We have not been able to find any other model with non-zero performance in the zero-shot or few-shot regime.

Moving on from the few-shot regime, fine-tuning a large model on the (approximately) aligned folios is unlikely to achieve anything. For example, introducing the folios into the training for olive-cormorant-bart does not appear to improve performance on the validation set at all. We do not include these results here for now since reproducing them is a waste of compute.

Intuitively, we should not need long context for most translation tasks. Most context encodes simple conditioning of the target sequence, such as the gender and plurality of referrents, or whether the sections are part of a list. [Context injection]({{< ref "results/injecting-context-during-decoding.md" >}}) should be sufficient to encode this conditioning information. On the other hand, by introducing long context we are making the translation task even more difficult. We are already in a low-resource setting and need to be looking for ways to make the task easier for the model, not harder. Even in the highest of resource contexts - English - modern SOTA long context models have poor performance, [especially in the middle of the text](https://arxiv.org/abs/2307.03172). According to this paper, we should expect that injecting an entire text into a translation model will not produce acceptable results even if we were not in a low resource setting. This means that we still have to segment the text and will still experience context fragmentation problems at the textual boundaries of the segments produced by whatever segmentation strategy we implement. But we are in a low resource setting, in fact we have so few examples of parallel folios that training any model with so little data is very likely to be a waste of time and compute effort. There may be some way to augment this dataset, for example by following an [XLM-like strategy](https://arxiv.org/abs/2012.15547), but this is unlikely to compensate for the fact that we have very few examples to train on.

While there may be some way around all these limitations, we decided to focus our attention on short models.

## Using the folio dataset to train a short model

We need to solve two problems in order to transform the folio dataset into a dataset of short parallel sentences: segmentation and alignment. We have an adequate solution for segmentation already, see [our previous work]({{< ref "results/segmenting-long-documents.md" >}}). Of course, alignment is generally regarded as difficult. In fact, one may argue that, in general, alignment is not an entirely well-defined problem, as the target may be too distantly related to the source to make sense of sentence-level alignment. Fortunately, the translators at 84,000 are very careful and deliberate, crafting their translations to be as semantically and, crucially, syntactically close to the original Tibetan as possible while preserving their English readability. This gives us hope that we may be able to solve the alignment problem for this particular dataset.

Broadly speaking, we can approach alignment in two ways: training a model to directly solve the alignment problem, or to try to use the existing causal bilingual translation model to few-shot the alignment problem in some way. While training a model to directly solve the alignment problem is, in principle, feasible, it should be possible (and potentially interesting more broadly) to few-shot alignment with the existing causal model. Note that this has been the approach of choice in [the past](https://aclanthology.org/D19-1453.pdf). Intuitively this makes sense - if a model can already translate, it should be able to align. It is reasonable to expect a model trained on a difficult task like translation to transfer well to an easier version of the same task, like alignment, especially since the causal translation model is (implicitly, due to the triangular mask in the loss function) trained on more examples than an alignment model would see.

### Alignment via explainability

One relatively obvious approach is to use explainability techniques to extract alignments from the causal model. In fact, there have been papers exploring the use of attention weights as an explanation mechanism for aligning words, for example:

- [Attention Weights in Transformer NMT Fail Aligning Words Between Sequences but Largely Explain Model Predictions](https://arxiv.org/abs/2109.05853),
- [On The Alignment Problem In Multi-Head Attention-Based Neural Machine Translation](https://aclanthology.org/W18-6318.pdf).

While somewhat interesting, the performance of these methods is generally poor, and the latter method relies on modifying the architecture of the causal model. In the hope of improving on these results, we attempted to use [integrated gradients](https://arxiv.org/abs/1703.01365) (implemented in [Captum](https://captum.ai/)) to explain the prediction of the next token in the translation by the causal model. We were hoping to see clear alignment, at least for some tokens, but were disappointed in the results, see below.

![Figure: Integrated Gradients attributions for causal model predictions](/images/results/mining-folios-for-parallel-sentences/sample-IG-causal-model-alignments.png)

{{% caption %}}
Figure: Integrated Gradients attributions for causal model predictions.
{{% /caption %}}

While there are signs of alignment, the signal does not appear clear enough to rely on even for word-level alignment. We even tried fine-tuning a model on an XLM-like mask substitution objective that's guided by target-side part-of-speech tags and using integrated gradients on its prediction. The results were even worse, see the figure below. Most of the signal is attributed to the first and last tokens, and the intermediate tokens do not appear to have any useful accumulation of attribution mass. While this is not unusual behavior for such models, it is disappointing nonetheless.

![Figure: Integrated Gradients attributions for mask substitution, guided with part-of-speech](/images/results/mining-folios-for-parallel-sentences/guided-mask-substitution-IG-attributions.png)

{{% caption %}}
Figure: Integrated Gradients attributions for mask substitution, guided with part-of-speech.
{{% /caption %}}

There may be many reasons for this poor performance, especially our choice of SentencePiece as a tokenization strategy for Tibetan. We do intend to eventually revisit our tokenization in order to produce a Tibetan transformer model that operates on syntactic structures that allow for solving tasks such as word segmentation and part-of-speech tagging more effectively than olive-cormorant can. However, this is still only word-level (token-level) alignment, and looking at the results here we see no reason to expect that we will get adequate results just from changing the tokenizer.

### Testing for alignment via the loss function

Instead of finding some regressor to *output* the alignment, we may start with the goal of *testing* for alignment and then produce aligned text by testing some relatively small number of candidates. To begin with, we are looking for some binary classifier function that takes a source and target sequence (a Tibetan and an English sentence) and produces a score for how closely aligned they are:

Source | Target | Score
-|-|-
།གཟུགས་སྟོང་པའོ། །སྟོང་པ་ཉིད་ཀྱང་གཟུགས་སོ། | form is emptiness, emptiness is form | 0.96
།གཟུགས་སྟོང་པའོ། །སྟོང་པ་ཉིད་ཀྱང་གཟུགས་སོ། | thus did i hear at one time | 0.13

{{% caption %}}
Table: Hypothetical example of an alignment classifier
{{% /caption %}}

But in fact we already have such a function - it is the training loss of our causal translation model! The score is the cross-entropy of the model logits, conditioned on the source sentence, and the target sentence. This score is not naturally normalized to be in [0, 1] but this is of no importance (it can be renormalized by interpreting it as a beam probability, if needed). We have examples of using the loss function this way below, after we describe the full alignment algorithm.

As an aside, another tempting avenue of approach is to try to use a model that embeds Tibetan and English sentences into the same latent space and use something like cosine similarity in this latent space as our test score. In our experience, this does not perform well without some sort of fine-tuning process to align the bilingual latent space. While this is, in principle, possible, it is not clear why we would prefer this method to simply using the loss function of our causal translation model. After all, this loss function is explicitly trained to solve the alignment testing problem.

### From testing to aligning

The critical next step is to move from testing for alignment to producing actual aligned sentences. The simplest way to design such an algorithm is to compare all possible pairs in a folio:

1. Begin with a Tibetan folio and its aligned English translation.
2. Split the Tibetan along spaces and the English along punctuation.
3. Generate all possible sequential concatenations of the Tibetan and English segments.
4. Calculate all scores between the candidate concatenations in Tibetan and English.
5. Pick the pairs whose scores exceed some threshold.

This naive algorithm has a number of problems:

- This is *extremely* computationally intensive. It is much more computationally intensive than translation. Most of this computation is wasted comparing sentences that have absolutely nothing to do with each other. It is probably better to just fine-tune an alignment regressor model.
- There is no good way to select the score threshold for step 5. No matter what we do, many sentences will have no aligned counterpart, while many sentences will have more than one.
- An important subtler problem is that the score thresholding may produce a biased dataset that's skewed towards certain keywords or syntactic structures. This would be particularly likely if we set the score threshold to a high value.

To address the last two issues, we modify the naive algorithm as follows. Assume we have a candidate set of Tibetan sections and English sentences, as well as a score for each pair in these candidate sets. This can be represented by a complete weighted bipartite graph whose vertices are Tibetan and English sentences and whose weights are the scores. We are looking for a matching on this bipartite graph the maximizes the sum of the weights. While there are many algorithms for solving this problem, we start with a simple greedy approach - we find the largest score in our current bipartite graph, add it to our matching, and then remove all the English sentences whose locations in the folio overlap with our chosen candidate. This exploits additional structure in our data beyond the bipartite matching problem, as well as ensuring the least duplication among the English.

Some examples of this strategy are shown below.

Tibetan | Olive-cormorant-NLLB translation | Match 1 | Score 1 | Match 2 | Score 2 | Match 3 | Score 3
-|-|-|-|-|-|-|-
སངས་རྒྱས་ཀྱི་ཞིང་བརྒྱ་ཕྲག་རྣམས་དང་། གང་གཱའི་ཀླུང་གི་བྱེ་མ་དང་མཉམ་པའི་ཞིང་དུ་རྒྱལ་པོར་འགྱུར་རོ། | one will become a king in hundreds of buddha realms and as many realms as there are grains of sand in the ganges | he will become king in hundreds of thousands of buddha fields. | -1.753 | he will become king in hundreds of thousands of buddha fields. equal in number to the grains of sand in the Gaṅgā river. He will also become the king of the devas and their heavenly hosts. | -3.334 | he will join the same family as the bodhisattvas and will perform the function of a king in every respect. Using the family vidyā. he will become king in hundreds of thousands of buddha fields. | -3.554
།ཆོས་ནི་གང་དུ་ཡང་འོང་བ་མེད། གང་དུ་ཡང་འགྲོ་བ་མེད་དེ། | phenomena do not come from anywhere, and they do not go anywhere. | they will understand that all three realms are nonexistent and unreal. | -2.635 | their lives will also be exhausted. | -2.981 | they will directly understand what is true. Knowing that all phenomena that arise are phenomena that cease. they will understand that all three realms are nonexistent and unreal. | -3.525

Unfortunately, the resulting algorithm is still far too computationally intensive. This method also still produces too many spurious matches, while the greedy matching approach causes errors made by the alignment test functions to compound and occasionally poison our final results. To solve this issue, we apply a number of heuristics that rely on the meticulous structure of the 84,000 translation process:

- The count of base English segments should be equal to the number of preceding Tibetan sections (not segments), fudged +/- by a location fudge of 5.
- Take concatenations of base English segments of width up to and equal to a width fudge of 2.
- Count the tsheg-separated Tibetan syllables in the Tibetan segment, call this the syllable count. Filter out all concatenations that don't have at least 0.9 \* syllable count or have more than 2.2 \* syllable count English words.

The location and width fudges were chosen by manual tuning of the results. The syllable count ratio multipliers of 0.9 and 2.2 were chosen by examining the dataset of parallel sentences:

Statistic | Value
-|-
mean | 1.416946
std | 0.580921
min | 0.103448
10% | 0.892857
20% | 1.000000
30% | 1.125000
40% | 1.214286
50% | 1.307692
60% | 1.416667
70% | 1.555556
80% | 1.750000
90% | 2.000000
max | 33.000000

{{% caption %}}
Table: Descriptive statistics of the ratio of Tibetan tshegs to English words in the 84,000 parallel sentences dataset.
{{% /caption %}}

![Figure: Scatter plot of tsheg counts versus word counts in the 84,000 parallel sentences dataset](/images/results/mining-folios-for-parallel-sentences/tshegs-to-spaces.png)

{{% caption %}}
Figure: Scatter plot of tsheg counts versus word counts in the 84,000 parallel sentences dataset.
{{% /caption %}}

## Results

This methodology increased our dataset size by approximately 2.5x:

Dataset | Total parallel sentence count
-:|:-
84,000 parallel sentences | 141,537
84,000 parallel sentences with mined folio augmentation | 362,139

We withhold the same dataset for validation as before. In particular, we emphasize that our validation data does not contain any mined sentences!

Tibetan | English | olive-cormorant-NLLB-600M cross-entropy
-:|:-|:-
།དག་ཅིང་ཉོན་མོངས་པ་ཞི་བར་བྱེད་པའི་ཆུའི་ཀླུང་ནག་པོ་བཟང་ཞེས་བྱ་བ་ནི་དེའི་ཚེ་གཙང་མ་བཟང་པོ་ཞེས་བྱ་བར་འགྱུར་ཏེ། | the river fine blackness, purified and with all its afflictions pacified, will at that time be called excellent purity, | -1.499
།ཁྱོད་ཅིའི་ཕྱིར་འཁོར་འདིར་འོངས། | understanding the householders exalted intention, | -2.956
།ཡུམ་ནི་འཇིག་རྟེན་པའི་གཟུགས་ཐམས་ཅད་ཀྱིས་ཟིལ་གྱིས་མི་ནོན་པའི་གཟི་བརྗིད་ཅེས་བྱའོ། | splendor that cannot be outshone by any mundane form will be his mother | -0.724
།སྙིང་གར་ཐིག་ལེ་བགྱིས་ནས་སྙིང་ལ་ཅུང་ཟད་ཅི་བསམས་པ་དང་། ཡིད་ལ་བསམས་ཤིང་བརྣག་པའི་དགོས་པ་དེ་དག་ཐམས་ཅད་མོད་ལ་འགྲུབ་པར་འགྱུར་རོ། | if the vidya holder marks his chest with a bindi, all his heartfelt hopes, desires, wishes, | -2.482
།དུད་པ་ཆེ་ཞིང་མེ་སྟག་རྒོད་པ་དང་། །འབར་བར་དཀའ་ཞིང་ཅུང་ཟད་ཅུང་ཟད་འབར། །མེ་ལྕེ་རྣམ་པར་ཆད་ཅིང་གཟི་མདངས་པོ་ཆེ་ཡི་མར་གྱིས་སྦྱིན་སྲེག་བྱ།། | if the fire is smokeless, shines with a golden color, has tremendous flames that swirl to the right, | -2.914
དེ་དག་ཉིད་མེད་ན་ལྟ་བར་གྱུར་པ་དག་མི་འབྱུང་ངོ་། | and not when those three are absent | -2.823
།ལག་པ་གཡོན་པ་བརྐྱང་བྱས་པའི། །མཁྲིག་མའི་ཚིགས་ལ་རབ་བཞག་པའོ། | at the wrist, with an upraised left hand | -1.593
།རྒྱལ་བ་མཆོད་པ་འདི་ནི་ལུང་སྟོན་ཏམ། །མི་ཡི་དབང་པོ་ལེགས་པར་ཆོས་གསུང་ངམ། | the jina to whom these offerings were made, will he give a prophecy | -1.794
།སྙིང་རྗེ་ཆེན་པོ་བྱུང་བར་གྱུར་ཏོ། | they repelled all obscurations, | -3.465
།དེ་ནས་འཇམ་དཔལ་གཞོན་ནུར་གྱུར་པས་གནས་གཙང་མའི་གཞལ་མེད་ཁང་ལ་ཡང་བལྟས་ཏེ། འཁོར་གྱི་དཀྱིལ་འཁོར་ཆེན་པོ་དང་། | then manjusri, the divine youth, again directed his gaze at the pure abode and the great assembly gathered there, | -0.571

{{% caption %}}
Table: Sample mined parallel sentences from our final dataset.
{{% /caption %}}

In our next blog post we will post the results of fine-tuning translation models on this dataset. We will fine-tune large (>2B parameter) models with this additional data. Our preliminary results show large improvement.
