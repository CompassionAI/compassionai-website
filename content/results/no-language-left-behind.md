---
author: palni
title: "No Language Left Behind"
date: 2022-10-20T14:13:02-04:00
draft: true
---

# No Language Left Behind

In July 2022, FAIR (Facebook AI Research) released a large multilingual transformer model that they call No Language Left Behind, or NLLB for short. This model builds on the work the FAIR team did for their submissions to the WMT machine translation competitions to train a state-of-the-art crosslingual model for 200 languages. The training protocol combines the best practices that FAIR uncovered in the last 5 years to produce a model that yields >30 BLEU for almost all the low resource languages included in it. One of these languages is standard Tibetan.

For more detail, see FAIR's blog post on NLLB at <https://ai.facebook.com/blog/nllb-200-high-quality-machine-translation/> as well as their paper at <https://arxiv.org/abs/2207.04672>.

## Can we use NLLB directly, without any training?

Simply, no. Consider some example sections from the Diamond Cutter Sutra.

Tibetan | SOTA translation | Zero-shot NLLB translation
--------|-------------------------------|----------------------------
།འདི་སྐད་བདག་གིས་ཐོས་པ་དུས་གཅིག་ན། | Thus did I hear at one time. | And I heard a voice in my ear, and it came to pass,
དེ་ནས་བཅོམ་ལྡན་འདས་སྔ་དྲོའི་དུས་ཀྱི་ཚེ་ཤམ་ཐབས་དང་ཆོས་གོས་བགོས་ཏེ། ལྷུང་བཟེད་བསྣམས་ནས། མཉན་ཡོད་ཀྱི་གྲོང་ཁྱེར་ཆེན་པོར་བསོད་སྙོམས་ཀྱི་ཕྱིར་ཞུགས་སོ། | Early one morning, the Blessed One donned his lower garment and Dharma robes, and, carrying his alms bowl, he went for alms in the city of Sravasti. | And he took his clothes, and his clothes, and his clothes, \[repeating forever...\]
།གཟུགས་ལའང་མི་གནས་པར་སྦྱིན་པ་སྦྱིན་ནོ། | They practice generosity without abiding in physical forms. | And he hath given his soul, and hath given his soul, and hath not kept his body.

We will stop here, as the situation really does not improve in any of the other texts. Since the results are almost completely unrelated to the "true" translations, we do not bother spending any time tweaking various generation parameters, such as repetition penalties and so forth.

Of note is the clear indication of the NLLB training data - it appears to have identified that we are in a religious corpus and is attempting to regurgitate passages from the New Testament. We will return to this in a later blog post when we discuss augmentation with NLLB data.

## Neural architecture and tractability

The full NLLB model uses a sparsely gated mixture-of-experts neural architecture that results in ~54 billion parameters. Thankfully, to make the model tractable FAIR released multiple dense distillations of NLLB. The architecture of the dense distillations is basically that of BART: it utilizes sinusoidal positional embeddings, dense attention layers with encoder cross-attention in the decoder, and two feature transformer layers in each transformer block. The smallest dense distillation is ~600 million parameters, at a cost of ~4 BLEU points on Facebook's FLORES-200 test set. Since the larger models are, for any practical purpose, intractable on modern hardware that would be available to typical users, we will focus on the ~600 million parameter dense distillation, which we will refer to by its Hugging Face name as NLLB-200-distilled-600M, or simply NLLB for short.

Even with Facebook's dense distillation, NLLB-200-distilled-600M still consumes a large amount of VRAM during training. A 16GB GPU, such as the V100s and A100s available on typical cloud services, only accomodates a batch size of 1, resulting in very slow and expensive fine-tuning.

However, if we are fine-tuning the model on a single language pair, the vast majority of the tokens never appear in our dataset. This means that their token embeddings are never updated by the back-propagation. Yet the embeddings for these tokens are loaded into GPU memory, wasting precious VRAM. We implement a simple token restriction trick: we remove the embeddings of tokens we will never use during training from the non-contextual embedding layer at the top of the transformer, as well as from the language modeling head at the bottom of the transformer. This reduces the number of parameters by almost 2x. The improvement in the backprop VRAM usage is quadratic, enabling us to use a batch size of ~4 on readibly available hardware without the need for expensive and brittle clustering.

The only potential performance cost of this strategy is in the removal of the parameters from the language modeling head. One simple way to avoid any performance cost to the token restriction trick is to freeze this LM head - this is precisely what happens to the parameters of the LM head that correspond to tokens that aren't included in the restriction. To estimate the impact of token restriction, we compare the validation set cross-entropy of fine-tuned NLLB-200-distilled-600M with and without freezing the LM head, see figure EUG!!!. As we can see, while there is some impact, it is quite small. Human evaluation of translations produced by models that are this close in cross-entropy is unlikely to register any difference. We conclude that the LM head can be safely ignored in practice.

It is likely that combining token restrictions with distillation would result in models that are deeper than NLLB-200-distilled-600M but still have a relatively small number of parameters. While this, of course, removes most of the languages other than the ones we are interested in, the transfer learning from those languages that occured during the training of NLLB is preserved. Such models are likely to be better, however, their improvement should be capped by the ~4 BLEU that was lost in the distillation process. We may investigate this in the future.

## Modularity - mixing and matching encoders and decoders

The NLLB model is a standard encoder-decoder stack. We would like to preserve this encoder-decoder structure but there is no _a priori_ reason not to mix and match encoders and decoders. The advantage of retaining the NLLB encoder is that the decoder is already adapted to it - the training will start further along the learning curve. However, our own olive-cormorant pre-trained encoder has a number of advantages as well:

 - Olive-cormorant uses the olive-big tokenizer. This tokenizer is guaranteed to have no UNKs on Tibetan text. The NLLB tokenizer has ~3% UNK rate on the Kangyur and Tengyur; this is not huge but it is not zero.
 - We pre-trained olive-cormorant on the Kangyur and Tengyur, this encoder may be more in-domain.
 - Olive-cormorant is based on AlBERT, thus it has substantially fewer parameters than the NLLB encoder and is much faster and more memory efficient in inference.

With this in mind, we compare three models:

 1. Olive-cormorant-BART: The olive-cormorant encoder and the BART decoder, both pre-trained by their respective authors.
 2. Olive-cormorant-NLLB: The olive-cormorant encoder and the NLLB decoder (with token restriction to English), both pre-trained by their respective authors.
 3. NLLB: NLLB-200-distilled-600M with token restriction to Tibetan on the encoder side and English on the decoder side.

### Without pooled context injection

We begin with a direct comparison of the encoder-decoder stacks without any context injection. The training curves are in figure EUG!!!, while table EUG!!! contains the minimum validation cross-entropy and corresponding step number.

As we can see, the gain from the NLLB decoder is large, while the gain from the NLLB encoder is significantly smaller. Note that this result is generally consistent with prior experience of people working with encoder-decoder models. This result also begs the question of what the difference between olive-cormorant-NLLB and NLLB is on test translations when evaluated by a human. We will investigate this question after comparing the models with context injection.

### With pooled context injection

We use our champion context injection methodology - BART-base context encoding with an additional attention layer on top, followed by BOS token pooling and injection into a second language identification token embedding in the decoder, see [our earlier work on context injection]({{< ref "results/injecting-context-during-decoding.md" >}}). The training curves are in figure EUG!!!, while table EUG!!! contains the minimum validation cross-entropy and corresponding step number.

We see a similar pattern to what we observed without context injection. Interestingly, and fortunately, the gain from context injection relative to BART is preserved relative to both versions of NLLB. This is likely due to the high ambiguity of many of the Tibetan examples.

## Human evaluation

We now translate our test document suite with olive-cormorant-NLLB and NLLB, both trained with context injection, and visually inspect the results.

 - Olive-cormorant-NLLB: [Diamond Cutter Sutra](EUG!!!), [Madhyamakavatara](EUG!!!), [Manjusrinamasamgiti](EUG!!!).
 - NLLB: [Diamond Cutter Sutra](EUG!!!), [Madhyamakavatara](EUG!!!), [Manjusrinamasamgiti](EUG!!!).

Generally, the difference is small, however, we do notice some patterns. Specifically, NLLB seems to be better at basic grammar such as numerical enumerations, while olive-cormorant-NLLB seems to be better at some unusual Buddhist phrases. The most clear example of this is in the Manjusrinamasamgiti:

Tibetan | Olive-cormorant-NLLB | NLLB | Reference[^1]
--------|----------------------|------|--------------
ཁྱབ་བདག་བདག་ལ་སྨན་པ་དང་། །བདག་དོན་བདག་ལ་ཐུགས་བེའི་ིར། །ུ་འུལ་དྲྭ་བས་མངོན་ོགས་པའི། །བྱང་ཆུབ་ཅི་ནས་བདག་ཐོབ་མཛོད། | lord, in order to help me and benefit me, out of compassion for me, may queen maya cause me to attain the perfect awakening. | EUG!!! | “O Master of the All-Pervasive, For my benefit, my purpose, from affection toward me, So that I may obtain Manifest enlightenment from illusion’s net
ཞུས་ལན་ཚིགས་བཅད་ུག | the answer to your question is six folded syllables. | EUG!!! | Six Verses in Reply

[^1]: <https://khyentsefoundation.org/manjushri-nama-samgiti-text-for-download-and-print/>

## Conclusions and future work

We conclude that the performance gain from NLLB is large and approximately the same as the gain from pooled context injection. Furthermore, the gain from context injection is preserved when we switch to NLLB. The gain from the NLLB encoder is small and largely comes from improved performance on basic sentences such as ones involving numbers.

It is likely that we may be able to augment the training of olive-cormorant-NLLB with data from the NLLB training dataset to attempt to combine the benefits of the two encoders. We investigate this in a future blog post. Furthermore, there may be fine-tuning strategies that involve MLM tasks that would allow us to combine the performance of the NLLB and our olive-cormorant encoders. We may investigate this in future work.