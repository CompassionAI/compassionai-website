+++
author = "palni"
title = "Project Munsel"
toctitle = "Munsel"
subtitle = "Modern OCR for a wide range of fonts and image quality."
subsubtitle = "Status: on hold pending potential users"
weight = 5
+++

This project is on hold. We are looking for user partners to move forward together. Please [contact us](/contact) if you are interested in partnering on this project.

## Problem

Tibetan texts are generally recorded on woodblock prints. These woodblocks are a kind of fixed type printing press. The Tibetans used fixed type for their most important texts, especially religious texts, and moveable type for less important printing. As a result, there are thousands of woodblock volumes of Tibetan texts on metaphysics, epistemology, Buddhist praxis, songs of realization, advice and admonishment, and many more besides.

Efforts to catalog and digitize these texts have been ongoing for decades. The [Buddhist Digital Resource Center, or BDRC](https://www.tbrc.org/) (previously the Tibetan Buddhist Research Center), has been at the forefront of this work. Various universities have also contributed, such as [Columbia University](https://dlc.library.columbia.edu/tibetan_collection).

While mountains have been moved to digitize vast quantities of printed material into images, converting these images into Tibetan unicode text enjoys much more limited success. As far as I could tell, the only existing usable open source tool for this is [Namsel](https://github.com/zmr/namsel). Developed before 2016, Namsel is an OCR system based on the old OCR pipeline of detecting axis-aligned bounding boxes around characters and then classifying them. Neither of these steps uses modern techniques. The model seems to be trained largely on synthetic data generated from digital fonts.

In my personal experiments, Namsel works fairly well on high quality images of standard font Uchen script, such as recently printed book pages. It fails completely on lower quality images and prints, often producing blank pages. With high quality images of out-of-sample fonts I have observed error rates on the order of multiple characters per short section of text. Despite the impression given by [the Namsel paper](https://escholarship.org/uc/item/6d5781k5), the performance seems to me to be very poor. Sadly this can be typical of academic publications in machine learning.

There is now also a closed-source Google OCR system called [Vision AI](https://cloud.google.com/vision) that can process similar inputs to Namsel, with similar accuracy. It works OK, not great, on high quality images. It especially struggles on the extended Tibetan Sanskrit alphabet. In my experiments this system performed somewhat better than Namsel (I suspect it was developed by the same person). It again suffers from excessive memorization of fonts and cannot process low quality images or poorly preserved prints.

## Proposal

Use modern one-shot OCR techniques such as [TrOCR](https://arxiv.org/abs/2109.10282) that do not require object detector labels (bounding boxes) for training. These can be pre-trained on various digitizations of the Kangyur and Tengyur, as well as synthetic text with a wide variety of augmentations and corruptions. We can then experiment with fine-tuning them to rapidly improve their performance on specific texts that are being digitized.

This will enable archivists and Tibetologists to make tradeoffs during OCR between accuracy and effort, while still providing a high accuracy baseline. Collaboration with archivists such as the BDRC would be very useful.
