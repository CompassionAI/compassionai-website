---
author: palni
title: "Segmenting Long Documents"
date: 2022-10-21T10:54:40-04:00
draft: true
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

[^1]: Here by segments we refer to Tibetan sections terminated by a shad. Relying on opening shads is an alternative that works quite well, but there are still too many situations where it doesn't. For example, this strategy makes a mess of the Madhyamakavatara, where every line of every quatrain begins with a shad.

## Two directions

There are two possible directions we can go from here. We can try to make very long models, or we can try to segment long texts into short phrases. The long model approach may seem attractive because it is end-to-end, but it has some crucial flaws:

 - The architectures that handle this are still under very active research and there is no pre-trained massively multilingual model like NLLB available.
 - A more serious problem is that we are in a low resource setting. It is unclear how well extremely long models with sparse attention will perform in this setting.

More abstractly, we are sceptical of the need for very long models. Even in classical Tibetan, most of the information required to process a short section is contained within the section. The information required from prior sections is almost always used for basic anaphora resolution and the imputation of pronouns. Generally speaking, we would expect some form of memory of the most recent generated text to be sufficient for this.

There are situations in which this information is indeed very far from the current section, for example in the Manjusrinamasamgiti, but we are very sceptical that modern models will be able to reliably extract it. Even humans need help to understand such texts.

Therefore, we will begin by experimenting with strategies to segment the long text into shorter sentences and apply a model with context injection to the sequence of shorter segments.