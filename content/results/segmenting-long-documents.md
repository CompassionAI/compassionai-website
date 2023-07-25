---
author: palni
title: "Segmenting Long Documents"
date: 2023-07-25T10:54:40-04:00
---

# Segmenting long documents

## The need for segmentation

Modern generative language models are trained on short sentences. While there are summarization models, they only accept long inputs, not outputs. Furthermore, while the inputs can be considered long, they are not entire books. To illustrate, we show the deciles of the distributions of the character lengths of English sentences in the Tibetan-English bitext that NLLB was trained on.

Quartile | Length
---------|-------
min | 7
25% | 23
50% | 32
75% | 49
max | 499

To make the problem more concrete, consider what happens when we use the original Facebook NLLB-200-distilled-600M checkpoint to translate some paragraphs from English to German (Project Gutenberg, _Crime and Punishment_ by Dostoyevski). First, we try a short sentence:

> On an exceptionally hot evening early in July a young man came out of the garret in which he lodged in S. Place and walked slowly, as though in hesitation, towards K. bridge.
>
> An einem außergewöhnlich heißen Abend Anfang Juli kam ein junger Mann aus der Garret, in dem er in S. Place übernachtete und ging langsam, als ob in Zögern, in Richtung K. Brücke.

The performance is good. Now we try a long paragraph:

> “If I am so scared now, what would it be if it somehow came to pass that I were really going to do it?” he could not help asking himself as he reached the fourth storey. There his progress was barred by some porters who were engaged in moving furniture out of a flat. He knew that the flat had been occupied by a German clerk in the civil service, and his family. This German was moving out then, and so the fourth floor on this staircase would be untenanted except by the old woman. “That’s a good thing anyway,” he thought to himself, as he rang the bell of the old woman’s flat. The bell gave a faint tinkle as though it were made of tin and not of copper. The little flats in such houses always have bells that ring like that. He had forgotten the note of that bell, and now its peculiar tinkle seemed to remind him of something and to bring it clearly before him.... He started, his nerves were terribly overstrained by now. In a little while, the door was opened a tiny crack: the old woman eyed her visitor with evident distrust through the crack, and nothing could be seen but her little eyes, glittering in the darkness. But, seeing a number of people on the landing, she grew bolder, and opened the door wide. The young man stepped into the dark entry, which was partitioned off from the tiny kitchen. The old woman stood facing him in silence and looking inquiringly at him. She was a diminutive, withered up old woman of sixty, with sharp malignant eyes and a sharp little nose. Her colourless, somewhat grizzled hair was thickly smeared with oil, and she wore no kerchief over it. Round her thin long neck, which looked like a hen’s leg, was knotted some sort of flannel rag, and, in spite of the heat, there hung flapping on her shoulders, a mangy fur cape, yellow with age. The old woman coughed and groaned at every instant. The young man must have looked at her with a rather peculiar expression, for a gleam of mistrust came into her eyes again.
>
> Wenn ich jetzt so Angst habe, was wäre, wenn es irgendwie passieren würde, dass ich es wirklich tun würde? Das ist eine gute Sache, dachte er sich, als er das vierte Stockwerk erreichte. Dort wurde sein Fortschritt von einigen Portierern verhindert, die sich mit dem Umzug von Möbeln aus einer Wohnung beschäftigt hatten. Er wusste, dass die Wohnung von einem deutschen Angestellten im öffentlichen Dienst und seiner Familie besetzt worden war. Dieser Deutsche bewegte sich dann, und so würde die vierte Etage auf dieser Treppe außer der alten Frau unbedeutbar sein.  Das ist eine gute Sache, dachte er sich, als er die Glocke der alten Frau ansah. Die Glocke gab ein schwaches Knack, als ob sie aus Zinn und nicht aus Kupfer war. Die kleinen Flächen in solchen Häusern hatten immer dunkle Kläger, die nicht so scharf waren. Er hatte die scharfe Note vergessen, die die die alten Augen und die Glocken offensichtlich von ihm offenbar waren, und ihn mit einem kleinen, scharfem Gesicht und einer kleinen, scharfem Gesichtsacheigen, scharfem Gesichtsacheigen und scharfem Gesichtsacheigen Gesichtsache, die sie mit einer kleinen, scharfemmenigeren, scharfemmen und scharfemmen Augen, und schwarzen Augen, die sie mit einer kleinen Frau, die mit einer kleinen, die mit einer schwarzen, schwarzen, schwarzen, schwarzen und schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, schwarzen und schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, schwarzen, und schwarzen, und schwarzen, und schwarzen, und schwarzen, und schwarzen, und eine kleine, und schwarze, und schwarze, und eine, und eine, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, die, [...]

Note that the English paragraph tokenizes to 505 and tokens. While long, it is less than half the length of the encoder - 1,024.

Our goal is to translate long text, which means that we need a segmentation strategy. Unfortunately, our situation is even more difficult than this because of two features of classical Tibetan: there are no sentence delimiters, while short sections often correspond to highly ambiguous sentence fragments. This means that we cannot simply rely on segments[^1].

[^1]: Here by segments we refer to Tibetan sections terminated by a shad. Relying on opening shads is an alternative that works quite well, but there are still too many situations where it doesn't. For example, this strategy makes a mess of the Madhyamakavatara, where every line of every quatrain begins with a shad. We will discuss this in more detail below.

## Two directions

There are two possible directions we can go from here. We can try to make very long models, or we can try to segment long texts into short phrases. The long model approach may seem attractive because it is end-to-end, but it has some crucial flaws:

 - The architectures that handle this are still under very active research and there is no pre-trained massively multilingual model like NLLB available.
 - A more serious problem is that we are in a low resource setting. It is unclear how well extremely long models with sparse attention will perform in this setting.

More abstractly, we are sceptical of the need for very long models. Even in classical Tibetan, most of the information required to process a short section is contained within the section. The information required from prior sections is almost always used for basic anaphora resolution and the imputation of pronouns. Generally speaking, we would expect some form of memory of the most recent generated text to be sufficient for this.

There are situations in which this information is indeed very far from the current section, for example in the Manjusrinamasamgiti, but we are very sceptical that modern models will be able to reliably extract it. Even humans need help to understand such texts.

## Naive approaches to segmentation

There are many possible approaches to segmenting Tibetan. Consider the following example from the first folio of the Kangyur:

> །རྒྱ་གར་སྐད་དུ། བི་ན་ཡ་བསྟུ། བོད་སྐད་དུ། འདུལ་བ་གཞི། བམ་པོ་དང་པོ། དཀོན་མཆོག་གསུམ་ལ་ཕྱག་འཚལ་ལོ། །གང་གིས་འཆིང་རྣམས་ཡང་དག་རབ་བཅད་ཅིང་། །མུ་སྟེགས་ཚོགས་རྣམས་ཐམས་ཅད་རབ་བཅོམ་སྟེ། །སྡེ་དང་བཅས་པའི་བདུད་རྣམས་ངེས་བཅོམ་ནས། །བྱང་ཆུབ་འདི་བརྙེས་དེ་ལ་ཕྱག་འཚལ་ལོ།

Some obvious ones are:

- Rely on the native Tibetan segments. By this we mean the pre-existing segmentations along spaces: [།རྒྱ་གར་སྐད་དུ།, བི་ན་ཡ་བསྟུ།, བོད་སྐད་དུ།, འདུལ་བ་གཞི།, བམ་པོ་དང་པོ།, དཀོན་མཆོག་གསུམ་ལ་ཕྱག་འཚལ་ལོ།, །གང་གིས་འཆིང་རྣམས་ཡང་དག་རབ་བཅད་ཅིང་།, །མུ་སྟེགས་ཚོགས་རྣམས་ཐམས་ཅད་རབ་བཅོམ་སྟེ།, །སྡེ་དང་བཅས་པའི་བདུད་རྣམས་ངེས་བཅོམ་ནས།, །བྱང་ཆུབ་འདི་བརྙེས་དེ་ལ་ཕྱག་འཚལ་ལོ།]

- Rely on the opening shad: [།རྒྱ་གར་སྐད་དུ། བི་ན་ཡ་བསྟུ། བོད་སྐད་དུ། འདུལ་བ་གཞི། བམ་པོ་དང་པོ། དཀོན་མཆོག་གསུམ་ལ་ཕྱག་འཚལ་ལོ།, །གང་གིས་འཆིང་རྣམས་ཡང་དག་རབ་བཅད་ཅིང་།, །མུ་སྟེགས་ཚོགས་རྣམས་ཐམས་ཅད་རབ་བཅོམ་སྟེ།, །སྡེ་དང་བཅས་པའི་བདུད་རྣམས་ངེས་བཅོམ་ནས།, །བྱང་ཆུབ་འདི་བརྙེས་དེ་ལ་ཕྱག་འཚལ་ལོ།]

- Append native Tibetan segments until the tokenization becomes too long, for example longer than 64 tokens. There are several parameters in this kind of segmentation, but one example with a short target token count could be: [།རྒྱ་གར་སྐད་དུ། བི་ན་ཡ་བསྟུ། བོད་སྐད་དུ། འདུལ་བ་གཞི། བམ་པོ་དང་པོ། དཀོན་མཆོག་གསུམ་ལ་ཕྱག་འཚལ་ལོ།, །གང་གིས་འཆིང་རྣམས་ཡང་དག་རབ་བཅད་ཅིང་། །མུ་སྟེགས་ཚོགས་རྣམས་ཐམས་ཅད་རབ་བཅོམ་སྟེ།, །སྡེ་དང་བཅས་པའི་བདུད་རྣམས་ངེས་བཅོམ་ནས། །བྱང་ཆུབ་འདི་བརྙེས་དེ་ལ་ཕྱག་འཚལ་ལོ།]

We can tabulate the pros and cons of these approaches as follows (for brevity we skip showing results for every single strategy, but our PyPI package implements all of these strategies for your convenience):

- Native Tibetan segments
  - Pros:
    - Very fast, does not require tokenization
    - Simple and easy to understand
  - Cons:
    - Individual segments can be highly ambiguous, for example individual entries in a long list connected by དང་
    - Generally low translation performance
    - Much longer translation running time due to many segments
- Opening shads
  - Pros:
    - Very fast, does not require tokenization
    - Relatively simple and easy to understand, although less so than the native segments
  - Cons:
    - Still ambiguous, many cases with low performance
    - Document style dependent. For example, as mentioned earlier, in the first few folios of the Madhyamakavatara every single section starts with a shad.
- Target token count
  - Pros:
    - Speeds up translation dramatically by batching many segments together.
    - Configurable, allowing to manually tune translation performance to some small degree.
    - Can still be fast if we use a fast implementation of the tokenizer.
  - Cons:
    - No single choice of parameters produces acceptable performance on all cases. For example, lists connected by དང་ are almost never segmented into a single segment containing the list.

Note that [context injection]({{< ref "results/injecting-context-during-decoding.md" >}}) helps with translation performance in all of these segmentation strategies but is not a panacea. This is because the model is trained on the pre-segmented 84,000 parallel sentence data, which does not look like any of the above segmentation strategies. For example, lists connected by དང་ that have been segmented in the middle of the list are not common in the training data. This leads us to the idea that works - segmentation modeling.

## Segmentation modeling - extrapolating from the 84,000 segmentations

Since we are in a low-resource setting, we cannot rely on increasing the size of the dataset to get around issues such as imperfect training-inference segmentation alignment. The closer our inference segments are to training segments, the better our model will perform in inference. The segmentation process applied by 84,000 is semantic and the naive strategies detailed above cannot reproduce it. So we tried to train a model that can. To be precise, we used our olive-cormorant AlBERT pre-trained Tibetan transformer to reproduce the 84,000 segmentations.

Note that this performance improvement is not captured by our test set that we evaluate on during the training of our causal translation model. Our test set consists of parallel sentences from withheld Tohoku numbers, these are already segmented by 84,000. However, the benefit of segmentation modeling is clearly visible in our generated example translations.

### Generation with a segmentation model

To generate translations with a segmentation model, we follow the following simple algorithm:

1. Load the segmentation model and tokenizer.
2. Load the Tibetan unicode document and pre-segment it along the spaces into `pre_segmented=document.split(' ')`.
3. Decide on a maximum tokenized length (usually 128 in our case due to the length of our olive-cormorant encoder).
4. Set `results=[]` and `cur_segment=""`.
5. While `len(pre_segmented) > 0`:
   1. Find the next segment as follows:
      1. Pop the first segment `cur_short_segment` from `pre_segmented`.
      2. Append `cur_segment += ' ' + cur_short_segment`.
      3. Tokenize `cur_segment`, if the token length is larger than the maximum tokenized length, proceed to step 5-2 with the value of `cur_segment` in step 5-1-1.
      4. Otherwise, score `cur_segment` with the segmentation model.
         - If the score is larger than some threshold, proceed to step 5-2 with the `cur_segment`.
         - If desired, the needed score can be tweaked to discourage long segments by using the formula `min(1 - base score threshold * candidate length / remaining length from the maximum tokenized length)`, we do not use this in our current SOTA generation algorithm.
         - If the score is less than the threshold, return to step 5-1-1.
   2. Append `cur_segment` to `results`.
6. Return `results`.

Essentially, we pre-segment along spaces and generate model segmentations by continuously appending pre-segments until we exceed some model score threshold (defaults to 0.5).

### Segmentation model - first cut

To prepare the dataset for training a segmentation model, we started with a simple strategy:

- Positive examples are the Tibetan side of the 84,000 parallel sentences.
- Negative examples are prefixes of positive examples that have been cut at an intermediate space.

For example, a positive example may be:

> སེམས་ཅན་ཐམས་ཅད་ཀྱི་དོན་ཁོ་ནར་སྦྱན་པ་དག་སྦྱན། ཚུལ་ཁྲིམས་སྲུང་། བཟོད་པ་སྒོམ། བརྩོན་འགྲུས་རྩོམ། བསམ་གཏན་ལ་བསམ་གཏན་དུ་བྱེད། ཤེས་རབ་སྒོམ། ཐམས་ཅད་མཁྱེན་པ་ཉིད་ཀྱང་ཡང་དག་པར་སྒྲབ་མོད་ཀྱི།

and we generate negative examples from it by cutting it as follows:

> སེམས་ཅན་ཐམས་ཅད་ཀྱི་དོན་ཁོ་ནར་སྦྱན་པ་དག་སྦྱན།<br>
> སེམས་ཅན་ཐམས་ཅད་ཀྱི་དོན་ཁོ་ནར་སྦྱན་པ་དག་སྦྱན། ཚུལ་ཁྲིམས་སྲུང་།<br>
> སེམས་ཅན་ཐམས་ཅད་ཀྱི་དོན་ཁོ་ནར་སྦྱན་པ་དག་སྦྱན། ཚུལ་ཁྲིམས་སྲུང་། བཟོད་པ་སྒོམ།<br>
> ...

We then randomly sample negative examples to balance the dataset 50-50 and train our olive-cormorant transformer with a binary sequence classification head. We withhold the same Tohoku numbers as in our translation dataset as validation data. Our final best validation F1 score with this approach is approximately 82%, split roughly evenly between precision and recall.

### Why this doesn't work well

Unfortunately, using this strategy we obtain occasional segments that are very long. For example, depending on the exact parameters of the various algorithms involved, the Heart Sutra may be segmented like so:

> །གཟུགས་སྟོང་པའོ། །སྟོང་པ་ཉིད་ཀྱང་གཟུགས་སོ།
>
> (Form is emptiness, emptiness is form)
>
> གཟུགས་ལས་སྟོང་པ་ཉིད་གཞན་མ་ཡིན་ནོ།
>
> (Emptiness is not something other than form)
>
> །སྟོང་པ་ཉིད་ལས་ཀྱང་གཟུགས་གཞན་མ་ཡིན་ནོ།
>
> (Form is not something other than emptiness)
>
> །དེ་བཞིན་དུ་ཚོར་བ་དང༌། འདུ་ཤེས་དང༌། འདུ་བྱེད་དང༌། རྣམ་པར་ཤེས་པ་རྣམས་སྟོང་པའོ།
>
> (similarly, feelings, perceptions, formative predispositions, and consciousness
are empty.)
>
> །ཤཱ་རིའི་བུ་དེ་ལྟ་བས་ན་ཆོས་ཐམས་ཅད་སྟོང་པ་ཉིད་དེ། མཚན་ཉིད་མེད་པ། མ་སྐྱས་པ། མ་འགགས་པ། དྲི་མ་མེད་པ། དྲི་མ་དང་བྲལ་བ་མེད་པ། བྲི་བ་མེད་པ། གང་བ་མེད་པའོ། །ཤཱ་རིའི་བུ་དེ་ལྟ་བས་ན་སྟོང་པ་ཉིད་ལ་གཟུགས་མེད། ཚོར་བ་མེད། འདུ་ཤེས་མེད། འདུ་བྱེད་རྣམས་མེད། རྣམ་པར་ཤེས་པ་མེད། མིག་མེད། རྣ་བ་མེད། སྣ་མེད། ལྕེ་མེད། ལུས་མེད། ཡིད་མེད། གཟུགས་མེད། སྒྲ་མེད། དྲི་མེད། རོ་མེད། རེག་བྱ་མེད། ཆོས་མེད་དོ། །མིག་གི་ཁམས་མེད་པ་ནས་ཡིད་ཀྱི་ཁམས་མེད། ཡིད་ཀྱི་རྣམ་པར་ཤེས་པའི་ཁམས་ཀྱི་བར་དུ་ཡང་མེད་དོ། །མ་རིག་པ་མེད། མ་རིག་པ་ཟད་པ་མེད་པ་ནས་རྒ་ཤི་མེད། རྒ་ཤི་ཟད་པའི་བར་དུ་ཡང་མེད་དོ། [...]
>
> (therefore, Sariputra, ... all the way up to the maximum encoder length)

We hypothesized that the reason this is happening is in the way the target variable for the segmentation model was constructed - if, during inference, the model makes a mistake and incorrectly skips a segmentation boundary, then all subsequent candidate segmentations are invalid and have only a low probability of exceeding the stopping threshold. Thus, the high accuracy of our segmentation model works against us.

### A different target variable - our final segmentation model

To repair this, we tweak our target variable. In brief, instead of the positive examples being the exact segments from 84,000, our positive examples are those that contain a complete 84,000 segment. Then, if our segmentation model incorrectly skips a segmentation boundary during inference, the error does not propagate and the segmenter has a high probability of stopping on all the subsequent candidate segments. To accomplish this, we build on our earlier work on [follows-anywhere sequencing]({{< ref "results/sequencing-long-text-from-sentence-fragments.md" >}}).

To be precise, the dataset has two parts: 84,000 sentences that are related according to follows-anywhere sequencing, and 84,000 sentences that are not related and are chosen at random instead. For each of the parts, we begin by concatenating two of these sentences with a | character between them.

- We make positive examples by picking a random space after the | character and taking everything before this random space (positive examples always include the | character). We then remove the | character from the positive example and clean up any doubled and trailing spaces.
- We make negative examples by picking a space before the | character.

As before, we then randomly sample negative examples to balance the dataset 50-50, withhold the same validation data, and train our olive-cormorant transformer with a binary sequence classification head. Our final best validation accuracy score with this approach is 86%, again evenly split.

As can be seen from our published results, this approach produces excellent segmentations consistently, with no need to ever override the segmentation model with heuristics. All of our published SOTA results (as of 7/25/2023) are produced with this segmentation strategy.
