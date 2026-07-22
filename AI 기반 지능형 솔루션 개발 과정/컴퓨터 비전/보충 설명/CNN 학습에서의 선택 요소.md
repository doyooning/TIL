
---
# 1. 활성화 함수 (Activation Function)
활성화 함수는 **뉴런의 출력값을 어떻게 변환할 것인가**를 결정합니다.
만약 활성화 함수가 없다면 CNN은 아무리 층을 많이 쌓아도 결국 하나의 선형 회귀와 다를 바가 없습니다.

| 함수         | 특징                  | 현재 사용 빈도 |
| ---------- | ------------------- | -------- |
| ReLU       | 가장 많이 사용            | ★★★★★    |
| Leaky ReLU | 죽은 ReLU 문제 해결       | ★★★★☆    |
| PReLU      | Leaky ReLU 개선       | ★★★☆☆    |
| ELU        | 음수값도 사용             | ★★★☆☆    |
| GELU       | Transformer에서 많이 사용 | ★★★★★    |
| Sigmoid    | 출력층 이진분류            | ★★☆☆☆    |
| Tanh       | RNN 등에서 사용          | ★★☆☆☆    |
| Swish      | Google 제안           | ★★★★☆    |
| Mish       | 성능 좋지만 계산량 큼        | ★★★☆☆    |

## (1) ReLU
가장 유명한 활성화 함수입니다.
```
입력
-3 -2 -1 0 1 2 3

출력
0  0  0 0 1 2 3
```

음수는 모두 0으로 만듭니다.

장점
- 계산이 매우 빠름
- Gradient Vanishing 완화
- CNN 대부분 사용

단점
- 음수가 계속 들어오면 뉴런이 죽을 수 있음(Dying ReLU)

## (2) Leaky ReLU
ReLU의 문제를 해결했습니다.
음수도 아주 조금 통과시킵니다.

예)
```
x=-3

ReLU
0

Leaky ReLU
-0.03
```

보통
```
0.01x
```

정도를 사용합니다.

장점
- 죽은 ReLU 감소

## (3) PReLU
Leaky ReLU에서
```
0.01
```

이 값을 사람이 정하는 것이 아니라
AI가 학습합니다.
```
ax
```

에서 a를 학습.

## (4) Sigmoid
출력을
```
0~1
```

사이로 만듭니다.
```
-∞ → 0

0 → 0.5

∞ → 1
```

사용처
- 이진분류 마지막 출력층

단점
- Gradient Vanishing이 심합니다.

## (5) Swish
Google이 만든 함수입니다.
```
x × sigmoid(x)
```

ReLU보다 성능이 좋은 경우가 있습니다.

# 2. 손실 함수 (Loss Function)

손실 함수는 **예측이 얼마나 틀렸는지 계산하는 함수**입니다.
CNN에서는 데이터 종류에 따라 달라집니다.

## (1) Binary Cross Entropy
이진 분류

예)
- 고양이
- 개
둘 중 하나

## (2) Categorical Cross Entropy
다중 분류

예) 숫자
```
0
1
2
...
9
```

## (3) Sparse Categorical Cross Entropy
Categorical Cross Entropy와 거의 같습니다.

차이
```
One-hot Encoding
```

을 하지 않아도 됩니다.

## (4) Mean Squared Error (MSE)
회귀 문제

예) 집값 예측
```
실제

5000만원

예측

4800만원
```

오차를 제곱해서 계산합니다.

## (5) Mean Absolute Error (MAE)
절댓값 오차
```
|예측-실제|
```

이상치에 강합니다.

# 3. 최적화 알고리즘 (Optimizer)
손실 함수를 최소화하도록 가중치를 수정합니다.

|Optimizer|특징|
|---|---|
|SGD|가장 기본|
|Momentum|SGD 개선|
|Nesterov|Momentum 개선|
|AdaGrad|학습률 자동 조절|
|RMSProp|AdaGrad 개선|
|Adam|가장 많이 사용|
|AdamW|최신 Vision 모델에서 많이 사용|
|AdaDelta|학습률 자동 조절|
|Lion|최근 연구되는 Optimizer|

## (1) SGD
가장 기본입니다.
```
현재 위치

↓

조금 이동

↓

반복
```

장점
- 단순
- 일반화 성능 좋음

단점
- 느림

## (2) Momentum
관성을 추가합니다.
```
공이 굴러가는 것처럼
```

업데이트합니다.

장점
- 지역 최솟값 탈출
- 더 빠른 수렴

## (3) RMSProp
각 변수마다 학습률을 다르게 조절합니다.
CNN에서도 많이 사용됩니다.

## (4) Adam
현재 가장 유명합니다.
Momentum과 RMSProp을 합친 알고리즘입니다.

장점
- 빠름
- 안정적
- 대부분의 문제에서 잘 작동

## (5) AdamW
Adam의 Weight Decay를 개선한 버전입니다.

최근
Vision Transformer
ResNet
EfficientNet
등에서도 많이 사용됩니다.

# 4. 학습률 스케줄러 (Learning Rate Scheduler)
학습이 진행되면서 **학습률(Learning Rate)을 자동으로 조절**합니다.

| 스케줄러                | 특징                |
| ------------------- | ----------------- |
| StepLR              | 일정 에폭마다 감소        |
| ExponentialLR       | 지수적으로 감소          |
| Cosine Annealing    | 코사인 곡선 형태로 감소     |
| ReduceLROnPlateau   | 성능이 정체되면 감소       |
| OneCycleLR          | 증가 후 감소           |
| Cosine Warm Restart | 반복적으로 학습률을 올렸다 내림 |
예를 들어 처음에는 큰 학습률로 빠르게 탐색하고, 후반에는 작은 학습률로 세밀하게 조정하면 더 좋은 성능을 얻는 경우가 많습니다.

# 실제 CNN에서 가장 많이 사용하는 조합
실무와 연구에서 많이 쓰이는 조합은 다음과 같습니다.

|목적|활성화 함수|손실 함수|Optimizer|
|---|---|---|---|
|이미지 분류(CNN)|ReLU|Cross Entropy|Adam 또는 SGD+Momentum|
|객체 탐지(YOLO)|Leaky ReLU / SiLU|Focal Loss, IoU Loss 등|SGD 또는 AdamW|
|이미지 분할(U-Net)|ReLU|Dice Loss + BCE|Adam|
|Vision Transformer|GELU|Cross Entropy|AdamW|
