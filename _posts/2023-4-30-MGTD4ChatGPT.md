---
title: "Machine-Generated Text Detection for ChatGPT"
date: 2023-04-30 00:22:22 -0800
categories: [portfolio]
tags: [portfolio, NLP, ChatGPT, Python] # TAG names should always be lowercase
image:
  src: /assets/img/blog/MGTD4ChatGPT/demo.jpg
  height: 400 # in pixels
  alt: Statistics on Different Datasets
---

## Paper Link

In this blog, we have provided a brief overview of this interesting project and a taste of the preliminary analysis. For more detailed experiments, please refer to our full paper [here](/assets/doc/MGTD4ChatGPT.pdf){:target="\_blank"}.

---

## Author

- [Haoquan Zhou](https://github.com/TaikiShuttle){:target="\_blank"} (haoquanz@umich.edu)
- [Kexuan Huang](https://github.com/kx-Huang){:target="\_blank"} (hkx@umich.edu)
- [Haoyang Ling](https://github.com/frankling2020){:target="\_blank"} (hyfrankl@umich.edu)

---

## Keywords

- Natural Language Processing (NLP)
- Large Language Model (LLM)
- Deep Learning
- Data Augmentation
- Machine-generated Text Detection
- ChatGPT

---

## 0. Background

The field of deep learning has made significant strides in the area of natural language processing (NLP), with large language models (LLM) being used to solve complex tasks such as natural language understanding and generation. OpenAI's newest conversational AI model, ChatGPT, has garnered considerable attention due to its ability to generate text on a variety of topics. While the model has demonstrated impressive performance across various domains, concerns have been raised about its usage and ethical implications. One concern is that ChatGPT could potentially be used to unfairly improve student performance in online exams. Additionally, ethical issues surrounding LLMs, such as biases hidden in the training dataset, also apply to ChatGPT. Despite these concerns, ChatGPT represents a significant milestone in the development of conversational AI models.

---

## 1. Project Goals

This research project aims to propose a machine-generated text detector for the GPT-3.5 model. The project goal is divided into three parts:

1. explore statistical information of machine-generated text
2. build a discriminator model on a mixture of machine-generated and human-written text
3. propose data augmentation methods for downstream tasks

To achieve this, a BERT-based classifier and an improved TF-IDF classifier are constructed to classify a given corpus into two classes: machine-generated and human-written text. The dataset is continuously fine-tuned to make it more robust. The proposed discriminator model will help schools identify effortless work and alert parents to their children's academic performance. Moreover, this project will contribute to understanding machine-generated text and raise ethical concerns about the power of AI in the NLP community.

---

## 2. Data

### 2.1 Data Source

The dataset used in this research includes the [Human vs. ChatGPT Comparison Corpus (HC3)](https://github.com/Hello-SimpleAI/chatgpt-comparison-detection){:target="\_blank"} under CC BY-SA 4.0 license, [MGTBench](https://github.com/xinleihe/MGTBench){:target="\_blank"} under MIT license, and two similar open datasets [ChatGPT-RetrieveQA](https://github.com/arian-askari/ChatGPT-RetrievalQA){:target="\_blank"} and [GPT-Wiki](https://github.com/aadityaubhat/wiki_gpt){:target="\_blank"}. Due to the upper limit imposed by the most recent GPT-3.5 API, these datasets enable us to study the differences between human experts and ChatGPT responses and to benchmark machine-generated text detection. The HC3 corpus is primarily used in this study for its focus on ChatGPT response features, while MGTBench presents a benchmark for detecting machine-generated text. After data preprocessing, the sample dataset is shown in Table 2 below.

![Sample Dataset after Preprocessing](/assets/img/blog/MGTD4ChatGPT/table_2.png)

### 2.2. Dataset Statistics

#### 2.2.1 Size

Table 1 provides the basic information of each dataset, including the number of human text and the number of machine-generated text.

![Basic Statistics of Dataset: Dataset Size](/assets/img/blog/MGTD4ChatGPT/table_1.png){: width="70%"}


#### 2.2.2 Answer Length

We performed a data exploration to compare the average answer length between human text and machine-generated text. To our surprise, we discovered that the machine-generated text was longer than the human response, as depicted in Figure 1. This discrepancy raises concerns about the quality of these datasets. Notably, we observed that GPT-Wiki had a shorter length than the human text, which may be a crucial point in detection. Additionally, we noted that despite the prompt requesting ChatGPT to generate approximately 200 words, the text was, on average, shorter than that, indicating that the model struggled to understand the numerical value in the prompt.

![The Distribution of Answer Length on Different Datasets](/assets/img/blog/MGTD4ChatGPT/figure_1.png)

#### 2.2.3 Sentence Length

We explores the distribution of the average number of words in a sentence for both machine-generated and human text. The findings indicate that the sentences in human responses tend to be shorter than those in machine-generated responses. This could be attributed to the fact that machine-generated text uses more complex or compound sentence structures, while human responses tend to use simplified grammar to create shorter sentences or break one long sentence into multiple short sentences. However, this pattern is not consistent in the GPT-Wiki dataset, where human responses exhibit a formal and rigorous writing style for Wiki-documenting, but a more casual expression for Q&A. The distribution of sentence length is shown in Figure 2.

![The Distribution of Sentence Length on Different Datasets](/assets/img/blog/MGTD4ChatGPT/figure_2.png)

#### 2.2.4 Word Frequency


For our evaluation of the word frequency in ChatGPT-generated text, we utilized the [Brown Corpus](http://icame.uib.no/brown/bcm.html){:target="\_blank"} as our corpus. We aimed to observe how often rare words appeared in the text, while also taking note of the average length of words generated by ChatGPT. Our findings, as shown in Figure 3, reveal that ChatGPT-generated text shares similarities with human-written text in terms of word distribution. However, the model is more inclined to generate frequent words. This observation may be due to the loss function of GPT, which prioritizes achieving high likelihood in the generated sentence and therefore favors the use of common words.

![The Distribution of Word Appearance in Drown Corpus on Different Datasets](/assets/img/blog/MGTD4ChatGPT/figure_3.png)

#### 2.2.5 POS Tagging

We used Part-of-speech (POS) tagging through the [Stanford stanza Python library](https://stanfordnlp.github.io/stanza/){:target="\_blank"}. Our analysis revealed that the distribution of machine-generated text and human text are strikingly similar, indicating that ChatGPT has learned the ability to mimic human text at the word level. The distribution of POS tagging is shown in Figure 4.

![The Distribution of POS Tagging on Different Datasets](/assets/img/blog/MGTD4ChatGPT/figure_4.png)

#### 2.2.6 Dependency Parsing

We conducted an analysis of dependency parsing in both human-generated and machine-generated text. The results showed a significant contrast between the two types of text, with the machine-generated text displaying a deviation in sentence structures due to the overemphasis on frequently-seen structures. The distribution of dependency parsing is shown in Figure 5.

![The Distribution of Dependency Parsing on Different Datasets](/assets/img/blog/MGTD4ChatGPT/figure_5.png)

---

## Experiments

For more details on experiments, please refer to our full paper [here](/assets/doc/MGTD4ChatGPT.pdf){:target="\_blank"}.

---

## Acknowledgement

This project is a course project of [SI630 Natural Language Processing: Algorithms and People](https://atlas.ai.umich.edu/course/SI%20630/){:target="\_blank"} in University of Michigan School of information. Great thanks to my teammates, [Prof. David Jurgens](https://jurgens.people.si.umich.edu){:target="\_blank"}, GSI Yutong Xie and all contributors of resources that are used or included in this project.
