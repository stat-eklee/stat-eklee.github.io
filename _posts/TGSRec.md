---
title:  "[논문리뷰] TGSRec"
excerpt: "Recommandation"

categories:
  - Recommandation
tags:
  - Recommandation
last_modified_at: 2022-03-28T12:11:00-15:00
---

# TGSRec

- Abstract
    - sequential parttern & temporal collaborative signals를 unify 하여 recommandation quality를 향상 > 어려움
    - 이유
    
    1) sequential patterns 과 collaborative signals을 simultaneously encode 하기 어려움
    
    2) collaborative signals의 temporal efects를 express 하는 것은 사소하지 않음.
    
    - Temporal Graph Sequential Recommender (TGSRec) 제안 : continuous-time bi-partite graph 를 정의
        - novel Temporal Collaborative Transformer (TCT) layer 제안 > users 와 items에서, collaborative signals을 동시에 capture.  또한, sequential patterns안의 temporal dynamics을 고려.
        - TCT 계층에서 학습한 정보를 temporal graph를 통해 propagate하여 sequential patterns과 temporal collaborative signals를 통합.
        - novel collaborative attention을 채택함으로써 self-attention mechanism 향상
        
- 1. introduction
    - users에 대한 historical time-ordered item purchasing sequences to predict future items. > sequential recommendation (SR) problem
    - self-attention model은  infers sequence
    embeddings at position 𝑡 by assigning an attention weight to each
    historical item and aggregating these items. The attention weights
    reveal impacts of previous items to the current state at time point 𝑡.
    - 이유 : crucial temporal collaborative signals을 무시해서.
    
    ![Untitled](TGSRec%2089377/Untitled.png)
    
    1) sequence의 collaborative signal을 model하기 부족. 동시에 encode하기 어려움
    
    - SASRec : Self-attention > users에서만 효과
    - SSE-PT : collaborative signals, same embedding(sequential encoding) > user-item interaction fail. & 명시적 collaborative signal을 express 불가
    
    2) time gap이 있는데 이걸 같은 contribution 으로 볼건가 
    
    - contribution
    
    1) graph sequential recommendation : sequential patterns 과 temporal collaborative signals 를 통합하는 graph embedding method를 학습 
    
    2) temporal collaborative transformer : collaborative signals와 temporal effect를 공동으로 모델링하는 temporal node embeddings을 추론하는 새로운 temporal collaborative attention mechanism.
    
    3) extensive experiments : five real-world datasets으로 실험-비교
    

- 3. Definitions and Preliminaries
    - 일반적인 SR과 다른 user interaction sequence 사용 > Continuous-Time Bipartite Graph (CTBG)
        
        ![Untitled](TGSRec%2089377/Untitled%201.png)
        
        - edge : timestamp as attribute
        - every user/item node in this graph preserve the sequential order via the timestamps at edges
    - Definition 3.1 (Continuous-Time Bipartite Graph). 𝑁 nodes 와 𝐸 edges 연속적인 시간 bipartite graph 는$\mathcal{B} = {\mathcal{U}, \mathcal{I}, \mathcal{E}_{\mathcal{T}}}$,
        - 여기서 $\mathcal{U}, \mathcal{I}$는 two disjoint node sets of users and items.
        - 모든 edge $e \in \mathcal{E}_{\mathcal{T}}$ 는  **tuple 𝑒 = (𝑢,𝑖, 𝑡)** 로 정의. where $𝑢 ∈\mathcal{U} , 𝑖 ∈ \mathcal{I}, and 𝑡 \in \mathcal{R}^+$
        as the edge attribute.
        - Each triplet (𝑢,𝑖, 𝑡) denotes the interaction of a user 𝑢 with item 𝑖 at timestamp 𝑡.
    - Definition 3.2 (Continuous-Time Recommendation). At a specific timestamp 𝑡, given user set U, item set I, and the associated CTBG, the **continuous-time recommendation** of 𝑢 is to generate a ranking list of items from $\mathcal{I} \setminus \mathcal{I}_𝑢(𝑡)$, where the items that 𝑢 is interested will be ranked top in the list.
    - Definition 3.3 (Continuous-Time Seqential Recommendation). For a specific user 𝑢, given a set of future timestamps$\mathcal{T}_𝑢 > 𝑇$ , the continuous-time sequential recommendation for this user is to make a continuous-time recommendation for every timestamp $𝑡 ∈ \mathcal{T}_𝑢$

- 4. proposed model
    
    ![Untitled](TGSRec%2089377/Untitled%202.png)
    
    ## 4.1 embedding layer
    
    - **two types of embeddings** 를 encode
        - 1) one being the long-term embeddings of nodes
        - 2) the other being the continuous- time embeddings of timestamps on edges.
    
    ### 4.1.1 Long-Term User/Item Embeddings
    
    - long-term collaborative signals representation에 대한 Long-term embeddings 은 중요.
    - user(item) node 는 vector $e_u(e_i) \in \mathbb{R}^d$ 로 parameterized.
    - embeddings은temporal user/item embeddings inference을 위한 시작 상태 역할을 한다는 점에 유의.
    - During the training process, embeddings will be optimized.
    
    ### 4.1.2 Continuous-Time Embedding
    
    - The time encoding function embeds **timestamps(scalar)** into vectors so as to
    represent the time span as the dot product of corresponding encoded time
    embeddings.
    - Define the temporal effects as a function of time span in continuous time
    space
    
    ![Untitled](TGSRec%2089377/Untitled%203.png)
    
    - temporal correlation between two timestamps
    
    ![Untitled](TGSRec%2089377/Untitled%204.png)
    
    positional encoding
    
    ## 4.2 Temporal Collaborative Transformer
    
    - 2가지 장점
        - 1) correlations의 temporal effects을 explicitly하게 characterizes하는 user/item embeddings 및 temporal embedding 모두에서 information 구성.
        - 2) TCT layer는  user-item interactions간 collaborative attention을 채택
            - the **query input** to the **collaborative attention** is from the **target node (user/item),**
            while the **key and value inputs** are from **connected neighbors**
    
    ### 4.2.1 Information Construction.
    
    - long term node embedding + time embedding > TCT layer의 input으로 쓰기 위해 구성
    
    ![Untitled](TGSRec%2089377/Untitled%205.png)
    
    - The query input information at the 𝑙-th layer for user 𝑢 at time t (eq3)
    
    ![Untitled](TGSRec%2089377/Untitled%206.png)
    
    - $h_u^{(l-1)}(t) \in \mathbb{R}^{d+d_T}$ : user u 의 time t에서의 information
    - $e_u^{(l-1)} (t) \in \mathbb{R}^{d}$ : temporal embedding of u
    - $\Phi(t) \in \mathbb{R}^{d_T}$ : time vector of t
    - $||$  : concat operation
    
    - Note that when 𝑙 = 1, it is the first TCT layer. >  The temporal embedding $e_u^{(0)} (t) = E_u$
    - When 𝑙 > 1, the temporal embedding is generated from the previous TCT layer.
    
    - In addition to the query node **itself**, also propagate temporal collaborative
    information from its **neighbors**.
    - Randomly sample S different interactions of 𝑢 before time t
    
    ![Untitled](TGSRec%2089377/Untitled%207.png)
    
    t_s < t sampling (eq4) 각 $(i, t_s)$  pair l-th layer 
    
    ![Untitled](TGSRec%2089377/Untitled%208.png)
    
    ### 4.2.2 Information Propagation.
    
    - Propagate the information of **sampled neighbors** $N_𝑢(𝑡)$ to infer the temporal
    embeddings (Eq5) historical interaction $\pi ^𝑢_𝑡(i, t_s)$ : $(u,i,t_s)$  의 영향 반영
    
    ![Untitled](TGSRec%2089377/Untitled%209.png)
    
    - $W_v \in \mathbb{R}^{(d \times (d+d_T))}$ : linear transformation matrix
    
    ### 4.2.3 Temporal Collaborative Attention
    
    - Adopt the novel temporal collaborative attention mechanism to measure the
    **attention weight** $\pi ^𝑢_𝑡(i, t_s)$,
    
    ![Untitled](TGSRec%2089377/Untitled%2010.png)
    
    $W^{(l)}_k$와 $W^{(l)}_q$ : linear transformation matrices, factor $\frac{1}{\sqrt{d+d_T}}$ : dot-product from growing large with high dimensions
    
    eq3, eq4를 eq6의 오른쪽 side는 재표현 가능
    
    ![Untitled](TGSRec%2089377/Untitled%2011.png)
    
    temporal effect : eq1
    
    - more stacked layers, collaborative signals and temporal effects are entangled and tightly connected됨
    - dot-product attention can characterize impacts of temporal collaborative signals
    - **normalize** the attention weights across **all sampled** interactions by employing a softmax function
    
    ![Untitled](TGSRec%2089377/Untitled%2012.png)
    
    - $H_{\mathcal{N}_u}^{(l-1)}(t) \in \mathbb{R}^{(d+d_T) \times S}$ > fig3 (추정하고자 하는 user와 time과 연관된 sample로 만듬)
    
    ![Untitled](TGSRec%2089377/Untitled%205.png)
    
    - $K_u^{(l-1)} (t)= W_k^{(l)} H_{\mathcal{N}_u}^{(l-1)}(t)$, $V_u^{(l-1)} (t)= W_v^{(l)} H_{\mathcal{N}_u}^{(l-1)}(t)$, $q_u^{(l-1)} (t)= W_q^{(l)} H_{\mathcal{N}_u}^{(l-1)}(t)$  > key, query, value
    - For **simplicity** and without **ambiguity**, **superscripts** and **time 𝑡** 무시하고 combine Eq. (6) and Eq. (8).
    
    ![Untitled](TGSRec%2089377/Untitled%2013.png)
    
    - dot-product attention in Transformer
    
    1) multi-head attention operation
    
    2) concatenate the output from each head as the information for aggregation
    
    - self-attention X > temporal collaborative attention, user-item interactions 과 temporal information 를 결합해 modeling
    
    ### 4.2.4 Information Aggregation
    
    - temporal node embedding의 결과물로 , TCT layer의 final step으로 query information을 (eq3), neighbor information in Eq. (5) aggregate
    
    ![Untitled](TGSRec%2089377/Untitled%2014.png)
    
    - concat > FFN two linear transformation layers with ReLU
    
    ### 4.2.5 Generalization to items.
    
    - user query perspective에서만 TCT 레이어를 제시하지만 query 가 특정 시간의 항목인 경우에도 유사.
    - user query information과 item query information를 번갈아 사용하고, user time pair으로 Eq(4)와 Eq(5)의 neighbor information를 변경하면 됨.
    - 다음 layer로 보내지는 $e_i^{(l)} (t)$로서 시간 t, item i의 temporal embedding에 대한 추론 가능
    
    ## 4.3 Model Prediction
    
    - TGSRec model은 𝐿 TCT layers 로 구성. 각 test triplet (u,i,t) 에서, last TCT layer 의 시점 t에서 u와 i 의 temporal embedding > $e_u^{(L)}(t)$ , $e_i^{(L)}(t)$
    - prediction score $r(u,i,t) = e_u^{(L)} (t) \cdot e_i^{(L)} (t)$ (eq11)
        - $r(u,i,t)$ : score to recommend 𝑖 for 𝑢 at time t
        - generalized continuous-time embeddings과 제안된 TCT 계층을 통해, 어떤 타임스탬프에서도 사용자/항목 임베딩을 일반화 및 추론할 수 있으므로 기존 작업이 다음 항목만 예측하는 동안 여러 단계 권장 사항을 실현할 수 있음.
        - Definition 3.3을 기반으로 각 사용자에게 주어진 타임스탬프의 항목 순위 목록을 권장함. 따라서 Eq. (11)를 사용하여 모든 후보 항목의 점수를 계산하고 점수별로 정렬가능.
    
    ## 4.4 Model Optimization
    
    - pairwise BPR loss > top-N recommendation
    
    ![Untitled](TGSRec%2089377/Untitled%2015.png)
    
    - mini-batch Adam optimization > Binary Cross Entropy (BCE) loss
    
    ![Untitled](TGSRec%2089377/Untitled%2016.png)
    
- 5. Experiments
    
    5.1 datasets
    
    ![Untitled](TGSRec%2089377/Untitled%2017.png)
    
    5.2 Experimental Settings
    
    5.3 Performance Comparison (RQ1)
    
    ![Untitled](TGSRec%2089377/Untitled%2018.png)
    
    5.4 Parameter Sensitivity (RQ2)
    
    ![Untitled](TGSRec%2089377/Untitled%2019.png)
    
    5.5 Ablation Study (RQ3 & RQ4)
    
    ![Untitled](TGSRec%2089377/Untitled%2020.png)
    
    5.6 Temporal Correlations (RQ4)
    
    ![Untitled](TGSRec%2089377/Untitled%2021.png)
    
    ![Untitled](TGSRec%2089377/Untitled%2022.png)