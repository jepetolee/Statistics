# 짧은 발성 연구 - Frozen FC

## 아이디어 전개

짧은 발성의 특징

* 화자 정보량이 적음 >> embedding boundary가 넓어짐![image-20220504153617012](C:\Users\gustj\AppData\Roaming\Typora\typora-user-images\image-20220504153617012.png)

* embedding 이후의 FC layer의 가중치(embd2라고 지칭)는 수식적으로 embedding boundary를 대표하는 정보이다.
* 긴 발성의 학습 정보를 TS learning 혹은 전이 학습을 통해 활용하여 짧은 발성의 성능을 향상시키려고 할 때, 마지막 계층을 고정하지 않으면 넓어진 boundary로 인해 embd2 값의 변화가 발생한다.
  * 그러나 짧은 발성 학습 시 embd2의 변화는 부정적 결과를 도출함



## 실험

* 전이 학습에서의 frozen fc 효과 (baseline: ECAPA-TDNN)

  1. teacher: 2초로 학습된 모델로 짧은 발성 평가

     | test length | EER   | minDCF |
     | ----------- | ----- | ------ |
     | full        | 2.879 | -      |
     | 5s          | 3.118 | 0.2194 |
     | 3s          | 3.738 | 0.2638 |
     | 2s          | 4.793 | 0.3147 |
     | 1.5s        | 5.636 | 0.3731 |
     | 1s          | 8.075 | 0.4860 |

  2. student: 1초로 모델 학습하여 짧은 발성 평가

     | test length | EER   | minDCF |
     | ----------- | ----- | ------ |
     | full        | 3.489 | -      |
     | 5s          | 3.680 | 0.2602 |
     | 3s          | 4.115 | 0.2887 |
     | 2s          | 4.857 | 0.3410 |
     | 1.5s        | 5.525 | 0.3855 |
     | 1s          | 7.338 | 0.4543 |

  3. student init by teacher: 2초 모델 가중치로 초기화 이후 1초 학습 모델로 짧은 발성 평가

     | test length | EER   | minDCF |
     | ----------- | ----- | ------ |
     | full        | 2.805 | -      |
     | 5s          | 2.975 | 0.2231 |
     | 3s          | 3.441 | 0.2543 |
     | 2s          | 4.210 | 0.2973 |
     | 1.5s        | 4.973 | 0.3428 |
     | 1s          | 6.569 | 0.4155 |

  4. proposed: 2초 모델 가중치로 초기화 이후 1초 학습 모델로 짧은 발성 평가 + FC 고정

     | test length | EER   | minDCF |
     | ----------- | ----- | ------ |
     | full        | 2.837 | -      |
     | 5s          | 2.985 | 0.2197 |
     | 3s          | 3.542 | 0.2549 |
     | 2s          | 4.242 | 0.2966 |
     | 1.5s        | 5.037 | 0.3319 |
     | 1s          | 6.591 | 0.4267 |

  5. random sampling: 1~2초로 학습된 모델로 짧은 발성 평가







* ECAPA-TDNN
  * 1s train 
    * 1s: 11.97
    * full: 3.35
  * 2s train
    * 1s: 
    * full: 
* ECAPA + SA(3s - 1s train)
  * 1s: 13.43
    * DA 조건 일치 시키기 + 평균 embd CCE loss 빼고
    * CCE 분리 실험

​				동일 발성에서 1s를 분류하고 

* 현재 진행 중인 실험
  * baseline: 2초 train
  * baseline 1024 + Vox2 학습 중 (1s, 2s) >> 고성능 모델에서 결과 보기
  * SA가 baseline보다 좋은 성능을 보여줄 수 있는가?
    * 실험: DA 조건 일치 + CCE loss없는 모델 vs baseline과 동일한 환경
    * 정보의 불확실성을 줄여주기 위해 SA 도입함



* 할만한 실험
  * feature map을 결합하여 긴발 embedding으로 사용하는 실험 설계
  * long embedding을 가지고 TTA 기법 적용도 실험해보면 좋을 듯함











# 22.06 실험

## SE-ResNet baseline

* softmax+AP: 2.11% -----결과가 너무 안좋음. 논문과 1% 차이
  * bs: 150
  * epoch: 36
* AAM-softmax: 1.43% (예상 1.28%)
  * exp1: 1.49%
    * bs: 150 / epoch: 36 / 200frame / 400frame eval / 
  * exp2: 1.43%  
    * bs: 150 / epoch: 99 / 200frame / 400frame eval /  lr_decay는 위 실험과 비슷
  * 400frame: 1.45%?
    * bs: 150 / epoch: 36 / 400frame / 400frame eval /  lr_decay는 위 실험과 동일
* SE-ResNet + segging
  * 2.131%
    * bs: 150 / epoch: 36 / 100frame train / 200frame TTA / lr_decay는 위 실험과 동일
* SE-ResNet + FM segging
  * 2.105%
    * bs: 150 / epoch: 36 / 100frame train / 200frame TTA / lr_decay는 위 실험과 동일
  * random segging
* Dual TS

| Name     | exp     | 1s    | 2s   | 3s   | Full(TTA)   |
| -------- | ------- | ----- | ---- | ---- | ----------- |
| baseline | AP loss | -     | -    | -    | 2.11        |
|          | 200F    | 10.56 | 3.85 | 2.39 | 1.52 (1.43) |
|          | 400F    | -     | -    | -    | -           |
|          | 100F    | 8.91  | 4.37 | 3.22 | 2.47        |
| segging  | vanilla | 9.36  | 4.14 | 2.88 | 2.00 (2.13) |
|          | FM      | 9.13  | 4.05 | 2.81 | 2.03 (2.11) |
|          | 400T_FM | -     | -    | -    | -           |
|          |         |       |      |      |             |
|          |         |       |      |      |             |
|          |         |       |      |      |             |
|          |         |       |      |      |             |



## ISKConvNet

* 변경사항
  * Voice Active Detection(VAD) - Kaldi SRE16 recipe
  * cepstral mean normalization(CMN)
  * adaptive symmetric score normalization
    * 논문: Analysis of score normalization in multilingual speaker recognition
  * training에서 전체 길이 발성을 가지고 평균내서 화자 임베딩 찾는건 어떨까?

* ISKConvNet baseline 
  * bs: 150 / epoch: 30 / frame: 400 - concat이 성능이 더 좋아서 실험 취소 / loss 동향도 거의 비슷
  * bs: 150 / epoch: 15 / frame: 200
    * 2.381%
  * **bs: 150 / epoch: 30 / frame: 200~400 / random** 
    * **2.036%**
* ISKConvNet + feature_concat
  * ISK_concat:  2.041%
    * bs: 150 / epoch: 30 / frame: 400
  * ISK_concat_200f:  2.269%
    * bs: 150 / epoch: 20 / frame: 200
  * ISK_concat_random:  2.349% - ?? 위 실험에서 random은 성능 향상 but 여기서는 하락
    * bs: 150 / epoch: 15 / frame: 200~400
    * 30 에폭까지 찍어봐야 하나?
  * ISK_random_aug 실험
* ISKConvNet + segging
  * ISK_segging:
    * bs: 120 / epoch: 15 / frame: 400-200 / 
  * ISK_segging_fm: 
    * bs: 120 / epoch: 15 / frame: 400-200 / 

| Name         | exp       | 1s    | 2s   | 3s   | Full (TTA)  |
| ------------ | --------- | ----- | ---- | ---- | ----------- |
| vanilla ISK  | 1(400F)   | -     | -    | -    | -           |
|              | 2(200F)   | 13.02 | 5.86 | 3.76 | 2.35 (2.38) |
| ISK + concat | 1(400F)   | 17.88 | 7.34 | 4.18 | 2.13 (2.04) |
|              | 2(200F)   | 13.03 | 5.48 | 3.61 | 2.26 (2.27) |
|              | 3(random) | 14.97 | 5.76 | 3.42 | 2.02 (2.04) |
| ISK+segging  | vanilla   | -     | -    | -    |             |
|              | FM        | -     | -    | -    |             |
|              |           |       |      |      |             |
|              |           |       |      |      |             |





