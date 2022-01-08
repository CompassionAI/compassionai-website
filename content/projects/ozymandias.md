+++
author = "palni"
title = "Project Ozymandias"
toctitle = "Ozymandias"
subtitle = "Evading bad actors and mass surveillance via non-adversarial attacks that are robust against model variance."
subsubtitle = "Status: research and prototyping"
weight = 1
+++

This project is under active research and a working prototype is being developed.

> *"...My name is Ozymandias, king of kings;*<br>
> *Look on my works, ye Mighty, and despair!"*<br>
> Nothing beside remains. Round the decay<br>
> Of that colossal wreck, boundless and bare<br>
> The lone and level sands stretch far away.<br>
> — <cite>Percy Shelley's "Ozymandias"</cite>

## Problem

It is safe to say that, as of this writing, some of the most prominent applications of artificial intelligence are by state bad actors. Common use cases of language understanding (henceforth NLU) models are mass surveillance, especially identifying dissidents in large amounts of harvested public text, but also locating individuals and their close contacts for intimidation and imprisonment.

The Tibetans are especially vulnerable to this, having already endured systematic cultural and corporeal genocide for decades at the hands of one of the most determined and least constrained bad actors in the modern world - the CCP. The Tibetan language is somewhat unique and currently hard to process. It is impossible to know whether the CCP has produced NLU models for modern Tibetan but, in my view, this is unlikely.

Unfortunately, it is likely that any pre-trained monolingual model for literary Tibetan will be easy to fine-tune on modern Tibetan. This means that simply releasing NLU models for literary Tibetan may put the Tibetan people in harm's way.

See for example the [famous paper](https://dl.acm.org/doi/10.1145/3442188.3445922) by Emily Bender, Timnit Gebru *et al* that got them expelled from Google. A more recent catalog of potential harms from language models by DeepMind can be found [here](https://arxiv.org/abs/2112.04359). At its broadest, the call here is to do more than just catalog the harms but to actually design tools to defeat them.

## Proposal

Produce non-adversarial attacks that rely on structural assumptions made by the models about the data. For example:

> It deosn't mttaer in waht oredr the ltteers in a wrod are, the olny iprmoetnt tihng is taht the frist and lsat ltteer be at the rghit pclae. The rset can be a toatl mses and you can sitll raed it wouthit porbelm. Tihs is bcuseae the huamn mnid deos not raed ervey lteter by istlef, but the wrod as a wlohe.

If possible, any such methods must be carefully benchmarked, on a rolling basis, against state-of-the-art model architectures that are most likely to defeat the attack - as of this writing, for the example above this would be CANINE. At the very least we should provide a method to defeat any Tibetan models we release.

Free, easy to use tools must be made publically available.
