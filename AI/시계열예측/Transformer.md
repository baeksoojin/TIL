# Transformer = Attention is all you need

> "Attention is all you need"에서 나온 모델로 기존의 seq2seq의 구조인 인코더-디코더를 따르면서도, 논문의 이름처럼 어텐션(Attention)만으로 구현한 모델


## Attention이란?

> 전체 input sequence의 각 단어별 State값을 output sequence에서 다시 한 번 참고하여 어떤 입력 문장의 단어와 가장 연관이 있는지, 즉 가장 집중(연관된)하고 있는 단어를 찾는 과정을 의미한다.


- attention의 3가지 핵심키워드

![image](https://user-images.githubusercontent.com/74058047/227697408-8803cf16-a433-433f-b50c-304f65980202.png)


Query → 질의하는 단어로 입력되는 단어 : 물어보는 주체

key → 인코더 셀의 은닉상태들  ⇒ keys : 인코터 셀의 모든 은닉 상태들 : 물어보는 대상

value → 인코더 셀의 은닉상태들  ⇒ values : 인코터 셀의 모든 은닉 상태들

(key는 방번호라면 value는 실제값)

## Transformer 원리

### 구조

<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/227697775-809b4846-31b6-45af-9cbe-9bbbb2abf03b.png">

encoder, decoder사이의 RNN에서 필요한 context vector를 제외하여 과거 데이터의 기억에 있어서의 문제를 해결한다.<br>

encoder, decoder는 각 파트에서 여러층이 쌓여서 만들어지고 하나의 encoder decoder에서는 attention이 수행되고 그 결과가 다음 층으로 넘어간다.<br>
<img width="300" alt="image" src="https://user-images.githubusercontent.com/74058047/227697944-a319e111-2fd0-4a3d-a0c3-1bdaa6c4de5f.png"><br>
해당 그림처럼 Feed Forward가 진행되는데, 잔차연결 및 전처리 과정을 진행하고 잔차를 통해서 이전 레벨에 대한 input값을 같이 가져감으로써 신뢰성을 키워줄 수 있다.<br>


### Positional Encoding

> 각 단어가 어떤 순서를 가지고 있는지에 대한 정보를 네트워크가 알 수 있도록 만드는 벡터이다.

RNN과 달리 Trasformer는 각 단어가 어떤 순서를 가지는지에 대한 순서를 보장하지 않는다.
<br>
=>  주기함수를 활용한 공식을 사용

### Attention 적용

- encoder : encoder&encoder self attention을 진행(여러번 진행하고 최종으로 나온 결과를 decoder에서 활용)<br>
- decoder : encoder에서 나온 attendion 결과와 decoder&decoder사이의 attention 결과를 모두 고려

- dot

### Multi-Head Attention

> attention의 과정을 쪼개서 병렬처리 함으로써 차원을 줄여서 다양한 관점에서 정보를 수집하는 과정이다.

<img width="200" alt="image" src="https://user-images.githubusercontent.com/74058047/227697412-a57c49b3-ed66-4648-a76e-66f38d5b25db.png">


- input vector와 query, key, value는 서로 dot product를 진행해야하니까 앞의 column size와 뒤의 row의 size는 동일해야한다는 특징을 가져야한다.

<br>

<img width="200" alt="image" src="https://user-images.githubusercontent.com/74058047/227697710-e516c2c9-5295-4b99-a787-a9af87dcc8a7.png"><br>

- 이후, 그림처럼 head를 쪼개서 나온 결과들을 모두 합쳐서 한번의 dot **product를 재진행해서 size를 처음과 동일하게 만들어주는 과정을 진행**(input값과 output값의 size가 동일해서 행렬덧셈연산이 가능)한다.<br>

- 논문에서는 단어의 차원을 512로 하였지만 num_heads를 8로 분해해서 64차원의 query, key, value vector를 하나의 attention value vector를 만들기 위해서 사용한 것을 확인할 수 있다.
    - 그림에서는 model 임베딩차원을 4라고하고 차원을 head를 2로했다고 하면 value차원(임베딩차원을 head개수로 나눈것)
- 효과 : 각 어텐션 헤드는 전부 다른 시각에서 볼 수 있게하여 않고 다른 시각으로 정보들을 수집한다. → 다양한 특징을 학습하도록 유도하는 효과가 존재한다.

### 잔차 연결(Residual connection)과 층 정규화(Layer Normalization)

- 트랜스포머에서 서브층의 입력과 출력은 동일한 차원을 갖고 있으므로, 서브층의 입력과 서브층의 출력은 덧셈 연산이 가능하다.<br>

    <img width="200" alt="image" src="https://user-images.githubusercontent.com/74058047/227698079-233ed3a8-2ee8-4f3c-8dea-8ac2e96730fd.png">

- 평균, 분산을 활용해서 정규화를 진행한다.<br>

    <img width="100" alt="image" src="https://user-images.githubusercontent.com/74058047/227698181-9a2bf0d7-dff0-4651-9bdd-136a9cfd802e.png">