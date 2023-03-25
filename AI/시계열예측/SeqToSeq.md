## Transformer model

### Seq2Seq model

![image](https://user-images.githubusercontent.com/74058047/227698419-ace988ac-7c4e-4959-af9b-7263fe7ab2d6.png)
> RNN을 활용해 입력된 시퀀스로부터 다른 도메인의 시퀀스를 출력하는 다양한 분야에서 사용되는 모델로 번역이나 챗봇 등에 활용


- 훈련과정 : Encoder를에 첫번째 sequence를 입력하고 예측값인 다른 sequence를 강제로 주입시켜서 학습을 진행
    
    **teacher forcing을 활용**
    
- 테스트과정 : Encoder에만 Input(sequence)를 넣으면, context vector를 통해서 symbol<sos>만 넣었을때 Decoder는 Output(다른 sequnce)를 예측해야함.

### 문제점

Encoder에서 **Context Vector**로 만들 때 발생

인코더가 입력 시퀀스를 하나의 벡터로 압축하는 과정에서 입력 시퀀스의 정보가 일부 손실된다는 단점

⇒ 입력 시퀀스 길이가 길어지면 출력 시퀀스 정확도가 떨어지게 된다.

⇒ 고정된 크기의 context vector로 인한 병목현상이 발생하여 성능 하락의 원인이 된다.

### 해결방법

Attention

> 입력 시퀀스가 길어지면 출력 시퀀스의 정확도가 떨어지는 것을 보정해주기 위한 등장한 기법으로 **해당 시점에서 예측할 단어와 연관이있는 입력 단어 부분만 집중해서 참고하여 출력단어를 예측하는 기법이다.**
> 

- main idea
    
    Decoder에서 출력 단어를 예측하는 매 시점(time step)마다, 인코더에서의 전체 입력 문장을 다시 한 번 참고한다.
    
    입력 sequence 전체에서 정보를 추출하는 방향으로 발전
    
- 특징
    
    전체 입력 문장을 전부 다 동일한 비율로 참고하는 것이 아니라 **해당 시점에서 예측해야할 단어와 연관이 있는 입력 단어 부분을 좀 더 집중(attention)**해서 본다.
