금융데이터 분석을 위해, RNN model을 LSTM을 통해서 조사를 해보자.

## LSTM

- 소개

![image](https://user-images.githubusercontent.com/74058047/227490119-55df7aff-25ed-4a4c-a7d9-577f56b504f1.png)


이전에 일어난 사건을 바탕으로 나중에 일어나는 사건을 생각하지 못하는 전통적인 neural network의 단점을 극복하기 위해서 나온 모델이다.

RNN의 모델의 parameter 조정을 위한 역전파 과정중에서 tanh의 미분 과정에서 인해 발생할 수 있는 vanishing gradient problem 의 문제가 발생할 수 있는 단점을 해결한 모델이 LSTM이다.

⇒ RNN의 히든 state에 cell-state를 추가하여 state가 꽤 오래 경과하더라도 그래디언트가 비교적 전파가 잘 되게 된다.

![image](https://user-images.githubusercontent.com/74058047/227490250-6a5339d7-fc11-4b07-9cee-aab76523540a.png)

- forget gate : 과거 정보를 잊기 위한 게이트. 이전 시간에 대한 h와 현재의 값 x를 받아 sigmoid를 취해주면 forget gate가 내보내는 값이 되는데, 0이면 이전 상태 정보는 잊고 1이면 이전 상태의 정보를 온전히 기억
- input gate : 현재 정보를 기억하기 위한 게이트. 이전 h값과 x를 받아 시그모이드를 취하고 같은 입력으로 tanh를 위한 값을 요소별 곱셈을 진행하면 결과가 된다.

기존에는 tanh를 미분하는과정에서 역전파 과정에서 기울기 소실 문제가 발생했다면 하나의 활성함수만 하이퍼볼릭탄젠트이고 나머지는 시그모이드이기에, 기울기 소실 문제가 발생하지 않는다.