---
author: palni
title: "Fine-Tuning Large Models On The Mined Dataset"
date: 2023-07-26T11:30:00-04:00
draft: true
---

# Fine-tuning large models on the mined dataset

We provide context, as detailed in our work on [context injection]({{< ref "results/injecting-context-during-decoding.md" >}}), for all sentences. We train a frozen BART encoder with an unfrozen top layer as our context embedder. We train olive-cormorant-NLLB, with the NLLB decoder from the 3B checkpoint, for a total model size of approximately 2.7B parameters. We use DeepSpeed in ZeRO-infinity mode for this.

This all results in a substantial improvement, both over the base olive-cormorant-NLLB model with 600M parameters, as well as over olive-cormorant-NLLB with 3B parameters but no mined augmentation.

EUG!!! SHOW TRAINING CURVES

Note the characteristic ladder shape of the training curves of large language models. The steps in the ladder occur at epoch boundaries.

We also note that the mined model of size 3B exceeds 30 BLEU for the first time on our (deliberately very challenging) validation set.
