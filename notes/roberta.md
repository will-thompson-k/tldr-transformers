# RoBERTa: A Robustly Optimized BERT Pretraining Approach

## Summary

| Model Name| Model Type (Encoder-Decoder, etc.)   | Pre-train Objective |  Tokenization  | Vocab Size | OOV Handling | Embeddings | Attention | Activations | Parameters | Training| Pre-Train Data | Batch Size |
|   :----: |   :----:   |     :----:   |    :----:   |  :----:   |  :----: |   :----:  |    :----: |    :----:   |    :----:   |:----:   |:----:   |:----:   |
| RoBERTa | Encoder-Only | **Masked Language Model** objective with dynamic masking (see below) + **No NSP or SOP** (NSP removal was shown to be better)| **Byte-level BPE (like GPT)** | 50k | Same as GPT? | Same as BERT | Same as BERT | Same as BERT | Model parameters are kept fixed: <ul><li> L=12, H=768, A=12 -> 110MM parameters (+~15MM for increase in vocabulary with byte-level BPE) </li></ul>  | They increase the pre-training steps from 100k (BERT) to up to 500k. They have a tweak on ADAM hyper-parameters. | They combine **5** datasets for **160MM GB in text**: <ul><li> Book Corpus + Wikipedia </li><li> CC-news </li><li> OpenWebText </li><li> Stories </li></ul>| ~2k batch size, max sequence length ~ 512 (less sometimes due to sampling technique) |


## TL;DR

First, the authors identify the problem that ELMo, GPT, BERT, XLM, etc, while impressive, are hard to shore up and objectively weigh their relative contributions given the fact that they are computationally expensive and usually use private training data. Thus, this paper is an attempt at replicating the BERT pre-training process. The authors propose a new approach to training BERT: 

1. training the model longer, with bigger batches, over more data
2. remove NSP objective (same as alBERT and others) 
3. dynamically changing masking pattern used in self-attention
4. Byte-level  BPE tokenization

This led to achieve SOTA across a number of benchmarks. **Their results are published in FairSeq**.

**Masking**: Original BERT paper implemented masking during _data pre-processing_; i.e., the masks were the same in every epoch. 
In this paper, they tried 2 variants:
- **Enhanced static masking**: during pre-processing phase, training data is **duplicated 10 times** so that each sequence is masked 10 different ways over the 40 epochs. 
- **Dynamic masking**: the mask pattern is generated every time a sequence is seen by the model. This helpful when increasing dataset sizes and steps.

**Notes on NSP**: The authors looked at a few different variants on this approach. They observe the original BERT paper's sampling approach is odd, where sentences are sampled from the same document, but are not necessarily contigous. They suggest a few alternatives with and without NSP, but settle upon "DOC-SENTENCES", which is where sentences are contiguously sampled from the same document but end if the paragraph ends. This allows for a **dynamic batch size** in cases where the number of tokens < 512.

**Larger batch sizes**: Looking at different batch sizes, perplexity is lower on held-out training data sets when increasing the batch size from 256 to 2K.

**Text Encoding**: The original BERT uses a character-level Byte-Pair Encoding ~ 30k tokens called **WordPiece**. This is learned after pre-processing the input with heuristic tokenizations rules. However, they use a method similar to **GPT** with **byte-level BPE** vocabulary containing 50K sub-word units (increasing the number of parameters ~ 15MM). The byte-level implementation uses bytes instead of unicode characters as the base sub-word units.


## Art