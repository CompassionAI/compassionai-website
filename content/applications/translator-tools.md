---
author: palni
title: "Future application: translator and dedicated student tools"
toctitle: "Translator/student tools"
subtitle: "An online translation tool intended to make translation faster and enable dedicated students to interact with source material."
date: 2022-08-26T15:06:45-04:00
---

## Outline of the tool

The tool will segment the text the user is interested in and display each segment separately like so (see underneath the figure for explanation):

|།དེ་ནས་ཚེ་དང་ལྡན་པ་རབ་འབྱོར་སྟན་ལས་ལངས་ཏེ། བླ་གོས་ཕྲག་པ་གཅིག་ཏུ་གཟར་ནས་པུས་མོ་གཡས་པའི་ལྷ་ང་ས་ལ་བཙུགས་ཏེ། བཅོམ་ལྡན་འདས་ག་ལ་བ་དེ་ལོགས་སུ་ཐལ་མོ་སྦྱར་བ་བཏུད་དེ། བཅོམ་ལྡན་འདས་ལ་འདི་སྐད་ཅེས་གསོལ་ཏོ།|
|:-|
|✅ Venerable Subhuti rose from his seat, draped his shawl over one shoulder, knelt on his right knee, and with palms pressed together said to the blessed one:|

|།བཅོམ་ལྡན་འདས་དེ་བཞིན་གཤེགས་པ་དགྲ་བཅོམ་པ་ཡང་དག་པར་རྫོགས་པའི་སངས་རྒྱས་ཀྱིས་བྱང་ཆུབ་སེམས་དཔའ་སེམས་དཔའ་ཆེན་པོ་རྣམས་ལ་ཕན་གདགས་པའི་དམ་པ་ཇི་སྙེད་པས་ཕན་གདགས་པ་དང་། དེ་བཞིན་གཤེགས་པ་དགྲ་བཅོམ་པ་ཡང་དག་པར་རྫོགས་པའི་སངས་རྒྱས་ཀྱིས་བྱང་ཆུབ་སེམས་དཔའ་སེམས་དཔའ་ཆེན་པོ་རྣམས་ལ་ཡོངས་སུ་གཏད་པའི་དམ་པ་ཇི་སྙེད་པས་ཡོངས་སུ་གཏད་པ་ནི་བཅོམ་ལྡན་འདས་ངོ་མཚར་ཏེ། བདེ་བར་གཤེགས་པ་ངོ་མཚར་ཏོ།|
|:-|
|✅ "Blessed one, it is wonderful that the Bhagavan, the Tathagata, the arhat, the perfectly enlightened Buddha gives to the bodhisattva mahasattvas as many boons as there are.|

|། བཅོམ་ལྡན་འདས་ བྱང་ཆུབ་སེམས་དཔའི་ ཐེག་པ་ ལ་ 👉<span style="background-color:#99ccff">ཡང་དག་པར་ [noun+particle: correctly]</span> ཞུགས་པས་ ཇི་ ལྟར་ གནས་པར་ བགྱི ། ཇི་ ལྟར་ བསྒྲུབ་པར་ བགྱི ། ཇི་ ལྟར་ སེམས་ རབ་ ཏུ་ གཟུང་བར་ བགྱི ། དེ་ སྐད་ ཅེས་ གསོལ་པ་ དང་ ། བཅོམ་ལྡན་འདས་ ཀྱིས་ ཚེ་ དང་ ལྡན་པ་ རབ་འབྱོར་ ལ་ འདི་ བཀའ་ སྩལ་ ཏོ །|
|:-|
|✏️ How should those who have entered the bodhisattvadyana abide, how should they practice, and how should their minds be properly trained?"|

|།རབ་འབྱོར་ལེགས་སོ་ལེགས་སོ།|
|:-|
|🤖💬 <span style="color:#777777">Excellent, Subhuti, excellent.</span>|

|།རབ་འབྱོར་དེ་དེ་བཞིན་ནོ།|
|:-|
|🤖💬 <span style="color:#777777">so it is, Subhuti.</span>|

|།དེ་དེ་བཞིན་ཏེ། དེ་བཞིན་གཤེགས་པས་བྱང་ཆུབ་སེམས་དཔའ་སེམས་དཔའ་ཆེན་པོ་རྣམས་ལ་ཕན་གདགས་པའི་དམ་པས་ཕན་བཏགས་སོ།|
|:-|
|🤖💬 <span style="color:#777777">it is just as you say. Thus it is that the thus-gone one benefits bodhisattva great beings by means of the sacred doctrine.</span>|

The work proceeds segment by segment:

- ✅: Completed segments influence the results of the segments following them due to the contextual nature of the model and the online generation of the text. This is a critical feature of such a tool. In general, it is very difficult to specify linguistic rules that formalize English sentences such as _translate the interlocutor as male_ or _the following sentences have the same referrent_. However, the model involved here is not syntactic, it is semantic. It should "understand" the corrections made by the translator and change the outputs of subsequent translations. Of course, it will not do so perfectly and some additional machine learning work may be required to get this to work well.
- ✏️: The section currently being worked on should be word segmented. The user can tap/click on a word and see its part-of-speech and a short, preferably single word, translation. This requires a new dictionary. We're hoping to use unsupervised machine translation to discover this dictionary, with human expert corrections incorporated later.
- 🤖💬: Future sections show candidate machine translations. As the user makes corrections to the translation, the translations of subsequent sections update automatically. The level of ambiguity of the translation should be color coded.

The tool will be configurable, for example the user should be able to configure the section segmentation strategy. It is not intended to be a simple, one-click translator like what we have in modern web browsers.

## Research needed

We first need to complete several research steps before we can develop this app.

- Similar requirements to the language teaching tools:

  - A model for Tibetan word segmentation, part-of-speech tagging and named entity recognition. This is likely a single model that combines all three tasks and uses morpheme-level or even character-level tokenization. Developing this model is part of our current roadmap for this year.

  - A simplified contextual dictionary that has only a short list of possible English words for a given Tibetan word (not long sentences) and that takes the part-of-speech tag into account. We would like to first use unsupervised machine translation to discover a first draft of this dictionary automatically, and then partner with linguists to complete it.

- Translation models:

  - A stylistic translation model for use by translators such as 84,000.

  - A literal translation model for use by students.

## Delivery methods

This would be a desktop app. It would be best to use a universal framework so that we can easily build one app for both Windows and Mac, as well as in-browser.