+++
author = "palni"
title = "Project Garland"
toctitle = "Garland"
subtitle = "Neural machine translation of literary Tibetan into English and other languages."
subsubtitle = "Status: working prototype"
weight = 4
+++

This project has a working prototype! Performance on out-of-sample texts is messy but promising. [Contact us](/contact) for more details, we cannot publish the model until [Project Ozymandias](/projects/ozymandias) is complete.

## Problem

The contents, insight and praxis of Tibetan Buddhism is one of the most precious heritages of humanity. Much of the answers to the big questions facing us are in these texts, both in terms of our true place in the natural world and in practical terms for how to use these realizations to live a meaningful and happy life. Especially remarkable is the clarity and directness of the presentation, as well as the strong emphasis on rational argument honed through millenia of stringent debate. This complete system of learning and practice [has consistently produced](https://treasuryoflives.org/biographies/view/Khenpo-Munsel/9929) people [capable of experiencing](https://en.wikipedia.org/wiki/Garchen_Rinpoche) greater well-being [during 20 years in a concentration camp](https://en.wikipedia.org/wiki/Yangthang_Rinpoche) than most (perhaps all) untrained people with a very high standard of living.

Unfortunately, this knowledge is difficult to access. Much of the literature is only extant in Tibetan, and even for the material extant in other languages the Tibetan translation is often unambiguously best. The Kangyur and Tengyur alone amount to over 300 volumes. There is also vast commentarial literature written by highly accomplished Tibetans over more than a millenium of practice. Significant chunks of this literature are important for understanding the metaphysics, epistemology, praxis and other aspects of Tibetan Buddhism.

There are many outstanding translators, both in the [84,000 project](https://84000.co/) and [beyond](https://www.alanwallace.org/). Even so, 84,000 has budgeted 100 years just to translate the Kangyur and Tengyur into English. As of this writing they have 90 years left to go, and many more languages besides - Chinese with [the Kumarajiva project](https://khyentsefoundation.org/the-kumarajiva-project-is-launched/), but what about many more? What about Japanese? Russian? Hebrew?

## Proposal

Train state-of-the-art architectures such as [BART](https://ai.facebook.com/research/publications/bart-denoising-sequence-to-sequence-pre-training-for-natural-language-generation-translation-and-comprehension/) to translate classical literary Tibetan. Start with English and expand to other languages. Use all available techniques to accelerate the learning of the model, but especially investigate:

- Online learning for conditional text generation tuned so that the model learns from corrections quickly.
- Efficient manual disambiguation of the original text that's model-augmented to make the process fast.
- Novel decoding techniques to improve the long-term memory of the decoder and give more human control over the generated text.

Develop a UI that enables translators to effectively interact with the model in online mode. Partner with translator organizations to ensure that the models are used effectively and are genuinely accelerating the translation process.
