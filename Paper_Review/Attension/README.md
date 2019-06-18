# "Attention is All You Need"  
> 관련 논문 : https://arxiv.org/pdf/1706.03762.pdf  
> Link : http://jalammar.github.io/illustrated-transformer/  
> Link : https://nlpinkorean.github.io/illustrated-transformer/ [번역]  
---
<br>


## Abstract  
CNN과 RNN을 없애고 Attention Mechanism에 기반을 둔 Transformer(Simple Network)에 대해 제안함  
  
<br>

## 1. Introduction  

- 참고. RNN의 기본 구조  
  - Sequence Model
  - Hidden_State(t-1)과 Input(t)을 통해 새로운 Hidden_State(t)를 생성함  
  - 따라서, 구조상 Sequential한 특성을 가지고 있기 때문에 Paralleliztion에 취약하다는 단점이 있음

본 논문에서는 RNN 또는 CNN 모델없이 전적으로 Attention Mechanism에 기반을 둔 모델(Transformer)을 구성하였으며, 이로 인해 학습시간이 매우 빠르다는 장점을 갖음
  
<br>
  
## 2. Background

- 참고 1. Sequential Computation을 줄이기 위한 방법
  - Extended Neural GPU, ByteNet, ConvS2S
  - 이는 Input과 Output의 모든 위치에서 Hidden Representation을 병렬로 처리하기 위해 CNN을 사용함
  - 그러나 distant position들의 dependency를 계산하기에는 수많은 연산이 요구되기 때문에 학습하기 어려움
  
- 참고 2. Self-attention
   - Attention은 전체 또는 특정 영역의 입력값을 반영하여, 그 중 어느 부분에 집중해야 하는지를 나타냄
   - Self-Attention (또는 Intra-Attention)
     - Sequence의 representation을 계산하기 위해서 하나의 Sequence에서 서로 다른 포지션과의 관계를 나타내는 메카니즘
     - 즉, 하나의 문장에서 현재의 단어와 이전 단어들 간의 상관 관계를 학습할 수 있음
     - ex. 문장 이해/요약 등에서 유용함
     
- 본 논문은 RNN이나 CNN 없이 Self-Attention만 사용한 Transformer 구조에 대해서 설명하며, 이 구조의 장점에 대해서 이야기하고자 함  
  
<br>

## 3. Model Architecture
- Overall Model Architecture of Transformer  
  모델의 전반적인 구조는 아래와 같이, 크게 Encoder와 Decoder로 구성되어 있음
  ![](../data/Transformer_Model_Architecture.PNG)

### 3-1. Encoder and Decoder Stacks
#### 1) Encoder
- 각각의 Layer는 두가지 sub-layer로 구성되어 있음 (multi-head self-attention +  fully connected feed-forward network)
- 각각의 sub-layer는 residual connection과 이를 Normalization하는 부분으로 구성되어 있음

#### 2) Decoder
- 기존 Encoder와 동일한 두가지 sub-layer 외 한가지 sub-layer를 추가함 (즉, 총 3개의 sub-layer로 구성되어 있음)
- Masked multi-head self-attention + multi-head self-attention +  fully connected feed-forward network
- encoder와 마찬가지로 각각의 sub-layer는 residual Connection 및 Normalization 을 수행함
- Maksing을 통해, 포지션 i의 prediction이 i보다 하위 포지션에 있는 output에만 의존할 수 있도록 함

### 3-2. Attention
- Attention은 Query 및 key-value Pair를 토대로 output을 매핑
- output은 Value의 weighted sum으로 계산함
- 각각의 Value의 weight 값은 Query-이에 상응하는 key의 compatability function에 의해 계산 가능함
<br>
  <img src="../data/Transformer_Attention.PNG" width="80%"> 


#### 1) Scaled Dot-Product Attention
- Input은 Query, Key, Value로 구성
- Q, K, V를 통한 dot-product 및 Softmax 등의 연산 과정을 통해 Value의 weight 값을 계산
- The matrix of outputs as:  
  - ![](../data/Transformer_Output_Matrix.PNG)  
<br>

- __Self Attention 계산 과정__
  <img src="../data/Transformer_Self_Attension_Matrix.png" width="80%">  
  <br>
  
  - STEP 1. Query, Key, Value Vector 생성
  - STEP 2. Score 계산 (Encoding 시 다른 단어들에 대해서 얼마나 집중 해야 할지를 결정함)
  - STEP 3. Score 값을 Root(Dimension Of Key)로 나눠줌
  - STEP 4. Softmax를 취함
  - STEP 5. STEP5에서 구한 Softmax 값과 Value 값을 곱함 (Weighted Value)
  - STEP 6. Weighted Value 벡터들을 모두 더함

#### 2) Multi-Head Attention
  <img src="../data/Transformer_Multi_head_attention.png" width="70%">

- 위의 구조와 같이 여러개의 Attention Layer를 Linearly Project 시킴
- 즉, Multi-Head Attention은 Attention Layer가 여러개의 'representation subspace'를 갖게 해줌



#### 3) Applications of Attention in our Model
Transformer는 아래 3가지 방법으로 multi-head Attention를 사용할 수 있음

1) "encoder-decoder Attetion" Layer에서,  
    Queries는 이전 Decoder layer로 부터 전달 받은 값이며 keys와 values는 encoder의 output으로 부터 전달받은 값.
    Decoder의 모든 포지션이 input sequence의 모든 포지션에 관여할 수 있음 (Sequence-to-Sequence Model)
    
2) 현재의 Self-Attnetion Layer의 Keys, Values, Queries는 이전 Encoder Layer에서 온 값을 사용함  
   즉, Encoder의 각 포지션은 이전 Encoder Layer의 모든 포지션에 관여할 수 있음  
   
3) Encoder와 유사하지만, Decoder의 각 포지션은 그 포지션을 포함한 곳 까지에 대해서만 관여할 수 있음.  
   Decoder에서는 Masking을 통해서 해당 포지션의 이후 정보를 차단함 (무한대 값으로 설정함으로써 정보를 손실시킴)  

### 3-3. Position-wise Feed-Forward Networks
각각의 Encoder/Decoder에는 Attension Sub-layer 외에도 Fully Connected Feed-Forward Network를 포함  
Fully connected Layer는 ReLU를 포함한 2개의 Linear transformation으로 구성됨  
  <img src="../data/Transformer_FFN.PNG" width="50%">

이 외, 다른 방법으로는 kernel size가 1인 2개의 Convolution을 사용  

### 3-4. Embeddings and Softmax
1) Input/Output token을 벡터로 변환하기 위해 learned Embedding을 사용함 (다른 Sequence Transduction Model과 동일)
2) Decorder Output에서 예측된 Next-token의 확률값으로 변환하기 위해 learned Linear transformation 및 Softmax 함수를 사용

본 논문에서는 두개의 Embedding Layer와 Pre-sofrmax Linear transformation의 weight Matrix를 공유해서 학습함  
(단, embedding Layer의 Weight Matrix에는 'd_model ** 0.5'를 곱함)  

### 3-5. Positional Encoding



<br>

## 참고 자료
1) Attention Mechanism 관련 자료
> https://www.youtube.com/watch?v=6aouXD8WMVQ  
> https://yjucho1.github.io/attention/attention/  

