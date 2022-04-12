---
layout: post
title:  "[DL] Transformer: Attention is all you need (NIPS, 2017)"
excerpt: "Deep Learning - NLP - Transformer"

categories:
  - ds
tags:
  - [DL] 

toc: true
toc_sticky: true
 
date: 2022-04-12
last_modified_at: 2022-04-12


---

## Summary 
* No Recurrence and No Convolution --> add "Positional Encodings" to the input embeddings at the bottoms of the encoder and decoder stacks (paper)

# What is a Transformer?
* (2017, NIPS) Attention is all you need 논문에서는 Transformer 방법을 사용하여 번역기를 만듦
* A 언어로 쓰인 문장 -> Transformer (여러 개의 'encoder blocks & decoder blocks' - 논문에서는 각각 6개 사용) -> B 언어로 쓰인 같은 의미의 문장으로 출력
* Similar to the 'Convolutional Seq2Seq model', the Trnasformer does not use any recurrence. It also does not use any convolutional layers. 
* Instead the model is entirely made up of linear layers, attention mechanisms and normalization  

## The Transformer - Model Architecture
* Transformer 모델 구조
![image](https://user-images.githubusercontent.com/98376833/162879880-a7003657-e910-4e18-97e7-8b9ec367bf70.png)

* Transformer 모델 전체 개요
![image](https://user-images.githubusercontent.com/98376833/162950420-430d29f3-beac-4eda-bd96-358f56449b13.png)
    * 출처: https://blog.promedius.ai/transformer/

### Input Embeddings
* Source Tokens (Inputs) x = (x1,..., xn)  -> Standard Embedding Layer -> Positional Embedding Layer -> Embedded vector
* 1) Standard Embedding Layer 
    * The tokens are passed through 'a standard embedding layer'
    * ```python nn.Embedding(input_dim, hid_dim) ```
* 2) Positional Embedding Layer
    * As the model has no recurrent it has no idea about the order of the tokens within the sequence. We solve this by using a second embedding layer called a **positional embedding layer**.
    * (원본 논문) static positional encoding 사용 / (Modern Transformer like BERT) learned positional encoding 활용 가능: ```python nn.Embedding(max_length, hid_dim) ```
```python
### pytorch 
## __init__(self, ...)
self.tok_embedding = nn.Embedding(input_dim, hid_dim)
self.pos_embedding = nn.Embedding(max_length, hid_dim)

self.dropout = nn.Dropout(dropout)
self.scale = torch.sqrt(torch.FloatTensor([hid_dim])).to(device) # (paper): sqrt(d_model) where d_model : hidden vector dimension 

## forward(self, src, src_mask):
pos = torch.arange(0, src_len).unsqueeze(0).repeat9batch_size, 1).to(self.device) # pos = [batch size, srclen]

# src: Input of Encoder Layer
src = self.dropout((self.tok_embedding(src) * self.scale) + self.pos_embedding(pos)) # src = [batch size, src len, hid dim]
```

### Encoder Layer
* Embedded vector x' = (x'1, ..., x'n-> (self-attention) a Multi-Head Attention Layer -> (point-wise) a Position-wise fully connected feedforward layer -> (Encoder Output z = (z1, ..., zn))
* Employ '(add) a residual connection' around each of the two sub-layers (multi-head / position-wise feedforward), followed by '(norm) layer normalization'.
* 1) Multi-Head Attention Layer
* 2) (Feed Forward) Position-wise fully connected feedforward Layer

* 논문에서는 인코더 블록을 6개 사용


### Decoder Layer
* Similar to Encoder, however it now has two multi-head attention layers.
* Similar to Encoder, Employ '(add) residual connections' around each of the sub-layers, followed by '(norm) layer normalization'
* Target token y(m) (- shifted right; 한칸씩 순방향으로 이동해가며 하나 토큰씩 모델의 input으로 들어감) -> (self-attention) Masked Multi-Head Attention Layer (over the target sequence) -> (encoder attention) a Multi-Head Attention Layer (which uses _the decoder representation as the query_ and _the encoder representation as the key and value_) -> (Output of Decoder Layer) 
* 1) Masked Multi-Head Attention Layer
    * self-attention 시 현재 위치의 이전 위치에 대해서만 attention 할 수 있도록 이후 위치에 대해서는 '-무한대'로 마스킹
    * (=) (paper) This masking, combined with fact that the output embeddings are offset by one position, ensures that _the predictions for position i can depend only on the known outputs at positions less than i_
* 2) Multi-Head Attention Layer
    * * 1)의 layer의 통과된 vector 중 query vector 만 가져오고, key , value vector 는 encoder block의 output에서 가져옴.
* 3) (Feed Forward) Position-wise fullly connected feedforward Layer

* 논문에서는 디코더 블록을 6개 사용

### Softmax
* Output of Decoder Layer -> (linear) FCN (Fully Connected Network) & Softmax -> y(m+1) (predicted next-token probabilities)
* (paper) We also use the usual learned linear transformation and softmax function to convert the decoder output to predicted next-token probabilities.

## Attention (in Transformer)
### Self-Attention
* Input (of Self-Attention) 내 각 단어 (token) 의 vector 들끼리 **서로 간의** 관계가 얼마나 중요한지 집중 (attention)
* 예시 - (Encoder Block) query,key,value 모두 word embedded vector (X') 활용
    * Query vector: X' @ W(Q) where W(Q): weight vector of query, @: Matrix Production
    * Key vector : X' @ W(K)
    * Value vector : X' @ W(V)
* (1) Attention score: (paper) Scaled Dot Product Attention Score 사용 
    * (kaye) Score(Q, K(j)) 
    * Query vector 는 문장 내 다른 단어들의 Key vector 와 곱해짐 (내적)
    * 이 때 자기자신과의 key vector 와도 내적 <-- Self-Attention
    * (scaled-dot product) 값 scaling을 위해 key vector의 크기의 제곱근으로 나눠줌. 이렇게 하면 softmax를 적용했을 때 합이 1이 된다.
    * ==> Score(Q, K(j)) : ( Q @ Transpose(K) ) / (sqrt (dimension of keys)) 
* (2) Attention distribution 구함: softmax 취하기
    * (kaye) α(Q,K)
    * ==> α(Q,K): softmax( (1)에서 구한 scaled-dot product )
* (3 - 최종) Attention Value: 
    * (kaye) Attention (Q,K,V)
    * (2)에서 구한 값과 Value vector 를 곱한 뒤 모두 더해줌
    * ==> Attention (Q,K,V): Sum of { α(Q, K) * V }

#### Self-Attention의 장점
* 내가 알고싶은 단어 (Query) 와 같은 문장 내 자기 자신을 _포함한_ 다른 단어들 (Key) 과의 관계를
* Self-Attention 의 query 와 key (& value) 연산을 통해
* 입력된 문장 내 단어들이 멀든 가깝든 관계를 유추할 수 있게 된다. 

### (kaye) Seq2Seq with Attention (2015, ICLR) vs Self-Attention (2017, NIPS) 의 Q, K, V vector
* Seq2Seq with Attention (2015, ICLR): Query, Key, Value 를 decoder cell의 hidden state s, encoder cell의 hidden state h 값을 사용
    * Query: s(t) of decoder cell
    * Key, Value: all h of encoder cell
* Self-Attention (2017, NIPS): Query, Key, Value 를 attention layer의 input query, key, value vector 와 Weight vector (W(Q), W(K), W(V)) 의 곱을 통해 생성
    * Query: (query 로 사용할 vector) @ W(Q)
    * Key: (key 로 사용할 vector) @ W(K)
    * Value: (value 로 사용할 vector) @ W(V)
    * 
--- 

#### Reference LINK.
* (2017, NIPS) Attention is all you need: https://papers.nips.cc/paper/2017/hash/3f5ee243547dee91fbd053c1c4a845aa-Abstract.html
* Transformer pytorch Code: https://github.com/bentrevett/pytorch-seq2seq/blob/master/6%20-%20Attention%20is%20All%20You%20Need.ipynb
* Transformer는 이렇게 말했다, "Attention is all you need.": https://blog.promedius.ai/transformer/