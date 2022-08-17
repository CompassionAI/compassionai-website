---
author: palni
title: "Future application: an app that teaches reading classical Tibetan"
toctitle: "Language teaching tools"
subtitle: "Follow established best practices to teach people to read classical Tibetan."
date: 2022-08-16T14:41:47-04:00
---

We will follow [Lingoal](https://www.lingoal.com) to design an app that teaches reading classical Tibetan, as well as physical teacher tools to help prepare lessons. The app can be made for smartphones as well as the Raspberry Pi, which would allow us to run it with very little electricity and bundle it with many other teaching tools for science, mathematics and programming.

## Outline of the app

The main capability of the app would be to use word segmentation, part-of-speech tagging and named entity recognition to display an overlay over Tibetan text like so:

| ཞིང་སྐལ་ | བྲེ་པེ་སྟན་ཆུང་ | བྱ་བ་   | མིང་   | མི་   | སྙན་   | རུང་   | སྟོན་ཐོག་ | གཞུན་པོ་ | ཡོང་པ་  | ཅིག་   | ཡོད་པ་  | དེ   |
|------|-------|------|------|-----|------|------|------|-----|------|------|------|-----|
| NOUN | PROPN | NOUN | NOUN | AUX | VERB | PART | NOUN | ADJ | NOUN | PART | NOUN | DET |

For the purposes of teaching the language, the app should start with the unsegmented text:

> ཞིང་སྐལ་བྲེ་པེ་སྟན་ཆུང་བྱ་བ་མིང་མི་སྙན་རུང་སྟོན་ཐོག་གཞུན་པོ་ཡོང་པ་ཅིག་ཡོད་པ་དེ།

The user is then requested to provide a word segmentation by indicating where the word breaks are. If the user is able to do this, the app uses the model to score the word segmentation quality and grade the result. If the user cannot do this, the app uses the model to provide a segmentation for them. Likewise, we can request the user to also label the word parts of speech, such as whether the word is a noun, a verb, an adjective, or a particle.

Once the segmentation step is completed, the app displays a red-yellow-green overlay over the words like so:

> <span style="background-color:red">ཞིང་སྐལ་</span>&nbsp;<span style="background-color:red">བྲེ་པེ་སྟན་ཆུང་</span>&nbsp;<span style="background-color:green">བྱ་བ་</span>&nbsp;<span style="background-color:green">མིང་</span>&nbsp;<span style="background-color:green">མི་</span>&nbsp;<span style="background-color:yellow">སྙན་</span>&nbsp;<span style="background-color:yellow">རུང་</span>&nbsp;<span style="background-color:yellow">སྟོན་ཐོག་</span>&nbsp;<span style="background-color:green">གཞུན་པོ་</span>&nbsp;<span style="background-color:red">ཡོང་པ་</span>&nbsp;<span style="background-color:green">ཅིག་</span>&nbsp;<span style="background-color:green">ཡོད་པ་</span>&nbsp;<span style="background-color:green">དེ</span>&nbsp;<span style="background-color:green">།</span>

A word is green if the user knows it, red if the user doesn't know it, and yellow if the user doesn't know it but is expected to know it according to their reading level. The word knowledge is determined by tests the user can take. Upon tapping/clicking on a word, a dictionary entry for that word is displayed. For further details, see the [Lingoal app](https://www.lingoal.com), especially the video on their homepage.

## Research needed

We first need to complete several research steps before we can develop this app.

- A model for Tibetan word segmentation, part-of-speech tagging and named entity recognition. This is likely a single model that combines all three tasks and uses morpheme-level or even character-level tokenization. Developing this model is part of our current roadmap for this year.

- A simplified contextual dictionary that has only a short list of possible English words for a given Tibetan word (not long sentences) and that takes the part-of-speech tag into account. We would like to first use unsupervised machine translation to discover a first draft of this dictionary automatically, and then partner with linguists to complete it.

- Reading levels for words in this dictionary. There is likely an automated way to do this, we will investigate.

## Delivery methods

Once the requisite research is complete, we can develop a universal codebase that allows us to deliver this application.

- Smartphone app, especially as a React Native or Flutter universal application.
- Raspberry Pi. Back of the envelope calculations suggest that the Raspberry Pi 4 should be a viable, low cost, low power platform to run these kinds of language teaching tools. A big advantage of the Raspberry Pi is that it is a full computer that's running Linux, with many additional teaching tools for science, mathematics, and computer science that can be deployed alongside our applications.
- Desktop app. This should be an easy adaptation of the smartphone app, if we develop it in a universal framework.
- Printouts. This would be a web application that can be used to produce printouts of exercises for in-person instruction.
