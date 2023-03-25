# Temporal Fusion Transformer

### Multi-horizon forecasting

- **multi-horizon forecasting**(미래의 바로 다음 시점값만 예측하는 것이 아닌 여러 미래의 타임 스텝에 대한 예측을 진행하는것)의 한 종류

![image](https://user-images.githubusercontent.com/74058047/227696968-f6c91c8b-ba92-4e26-b576-3bc191dcf724.png)

- Observed Input : 미래에는 알 수 없는 관측해야하는 관측 변수
- Known input : 알고있는 변수로 시간에 따라 달라지긴 하지만, 현재에도 그 값을 알 수 있으며 미래에도 그 값을 수 있는 week와 같은 개념.
- Static Covariates : 변하지 않는 값

### 등장배경

**< feature가 다양한 multi-horizon forecasting>**

- Multi-horizon forecasting의 경우에는 input feature들이 다양한데 이를 효과적으로 처리할 수 있는 방법이 존재하지 않음

⇒ 다양한 형태의 input을 효과적으로 처리할 수 있는 방법이 필요한데 이를 해결하기 위한 방법을 TFT가 제시한다.

**<모델의 해석력 = 신뢰성>**

모델 예측 결과의 근거를 이해하는 것이 중요하다.

해석가능한 모델로 만들기 위한 방법론에는 LIME, SHAP등이 있는데 이러한 기존 모델은 시계열 예측 모델에서 시간 모델을 고려하지 않는다는 한계가 존재함.

따라서 2가지 접근 방법은 시계열 데이터에서 낮은 설명력을 갖게 된다.

⇒ 시계열 데이터에서 예측을 해석력을 갖게 할때 시간 모델을 고려하는 방법이 필요한데 이를 해결하기 위한 방법을 TFT가 제시한다.

+) static covariates의 중요도를 고려하지 않았다는 점이 존재하고 이를 해결하기 위한 방법 제시

## Temporal Fusion Transforer 살펴보기

<img width="746" alt="image" src="https://user-images.githubusercontent.com/74058047/227696995-59fb1f8c-b3bc-473b-bb08-d60561859058.png">


### Gating mechanism

> 사용하지 않는 component를 skip하는 매커니즘

![image](https://user-images.githubusercontent.com/74058047/227697062-289423ad-0e6e-4f6f-b604-27c6ee94de48.png)

input값과 context vector(외부값)이 함께 들어가서 층 2개를 거쳐서 결국 GLU를 만나게 된다.

GLU : 기여도를 조정하는데 **sigmoid**를 사용하고 따라서 사용할 것만 남기게 됨.

⇒ 모델의 각 component마다 해당 메커니즘 적용

### Variable Selection Networks

> 예측에 기여하는 성분들은 선택하고 불필요한 Nosie 입력 변수들은 제거하는 방법

![image](https://user-images.githubusercontent.com/74058047/227697096-849a0a72-3dce-4ba0-a285-6ce0f1f91f5c.png)

⇒ input은 variable selection network를 거친다. 해당 network를 통해서 processed Feature를 산축하게 되는데 이를 통해서 Variable Importance를 산출하게 되고 이것은 Encoder, Decoder에 사용한다.

### Static covariate encoders

static 메타정보를 context vector로 각 component에 input으로 사용하게 된다.

### Temporal processing

- GRN → Multi-head Attention(transformer와 다르게 각 헤드별로 구한 값에 평균을 사용) → GRN → 전체를 skip하는 과정을 한번더해줌

### Quantile forecast

- 다중예측에서는 입력값으로 **예측길이, 예측 변수, Observed inputs(관측값), Known inputs(알고있는 값), 공변량이 주어졌을 때 분위수를 예측한다.**

- 분위수를 예측하기 위해서는, Quantile Loss를 사용하게 된다.

- Quantile Loss : 데이터를 sort했을 때 해당 배율로 나눠지는 성질을 이용<br>
- EX)만약에 분위수가 (q) 가 0.9이고 해당 분위수를 가지게 하는 실제값이 0.95일때는?<br>

    case1 : 예측값이 0.1일때 0.95와 차이가 많이난다.<br>
    case2 : 예측값이 0.8일때 0.95와 차이가 나긴하지만 case1보다 차이가 적게난다.<br>

⇒ 분위수값의 target에 적합시키고자하는 loss function으로 활용한다.<br>
⇒ 예측을 위해서 보통 4분위를 사용한다. 

### 예제

[pytorch에서 제공하는 튜토리얼](
https://pytorch-forecasting.readthedocs.io/en/stable/tutorials/stallion.html)
을 보면, 결국 import해서 library를 사용하면 된다.<br>

- 그 와중에 결과를 확인해보면 TFT의 network 구조에 대해서 확인해볼 수 있는데 encoder, decoder, 그리고 Quantile Loss가 loss function으로 network에 사용된다는 것을 알 수 있다.<br>
- 뿐만 아니라 input feature를 정제하기 위해서 사용하는 매커니즘 GLU, VSN도 확인이 가능하다.<br>