---
title: "[논문 리뷰] SAM: Segment Anything"
venue: ICCV (2023)
draft: false
tags: ["paper-review", "image-segmentation"]
categories: ["Paper Review"]
summary: "prompt encoder를 통해 모델이 points나 bbox, mask, texts와 같은 prompt를 입력받아 학습하고 학습에 사용되지 않은 새로운 이미지나 테스크에 대해 zero-shot performance를 측정함. 특히 데이터 구축과 모델 학습이 한 사이클이 되어 함께 발전되는 Data engine도 인상적임"
math: true
---

## 논문 정보

- **제목**: SAM: Segment Anything
- **저자**: Alexander Kirillov et al.
- **소속**: Meta AI Research, FAIR
- **학회/저널**: ICCV 2023
- **링크**: https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10378323
- **깃허브**: https://github.com/facebookresearch/segment-anything

## 읽은 계기(목적)
현재 서울대학교 IMSI_Lab의 SANDL팀에서 내시경 video 내의 용종(polyp)을 localization 하는 연구를 진행하고 있다. 그러나 용종은 이미지 내에서 매우 작은 픽셀로 존재하며 video-level이 되었을 때(실제 내시경 영상) 여러 프레임 중 일부(짧은 순간)에만 용종이 존재하기 때문에 기존 모델들로 이를 해결하기에는 매우 어려운 과제라고 생각했다. <br>"모델 학습 시 image뿐만 아니라 point나 bbox, mask와 같은 prompt를 함께 모델의 입력으로 넣는다면 해결할 수 있지 않을까?" 라는 생각에 해당 논문을 읽었다. 현재는 이와 관련한 논문을 라이팅하고 있으며, 해당 논문은 읽은지 몇 주 지났기에 아래 내용들에 오류가 포함될 수 있다.

## 한 줄 요약

> prompt encoder를 통해 모델이 points나 bbox, mask, texts와 같은 prompt를 입력받아 학습하고 학습에 사용되지 않은 새로운 이미지나 테스크에 대해 zero-shot performance를 측정함. 특히 데이터 구축과 모델 학습이 한 사이클이 되어 함께 발전되는 Data engine도 인상적임

## 배경 및 문제 정의
이 논문은 어떤 이미지든 valid mask를 생성하는 Segment Anything(SA) project를 소개하며 이를 위한 task, model, dataset을 구축한다. (모두 novelty 요소들)<br><br>
![figure1](/static/images/sam/figure1.png)

일반적인 컴퓨터 비전 논문은 저자가 제안한 모델을 벤치마크 데이터셋으로 학습하고 성능을 측정하여 task를 해결하는 방식으로 작성된다.
그러나 이 논문은 모델 개발과 더불어 11M개의 이미지와 1.1B개의 마스크로 구성된 SA-1B 데이터셋을 구축한 것이 특징적이다.
SA-1B 데이터셋의 "마스크 1.1B개" 라는 숫자는 기존의 어느 데이터셋의 마스크보다 400배를 넘는 수치라고 한다.

![논문 인용](/static/images/sam/text-03.png)
![figure5](/static/images/sam/figrue5.png)

또한 논문의 저자들이 구축한 SA-1B 데이터셋은 평균 3300x4950 이라는 고해상도 이미지로 구성되어 있으며 타 데이터셋들과 mask quality를 비교했을 때 전혀 꿇리지 않는다고 한다.

![논문 인용](/static/images/sam/text-04.png)

![figure 6](/static/images/sam/figure6.png)

text corpora가 풍부한 web-scale dataset에서 scaled and trained된 foundation model들은 파인튜닝된 모델과 비교하여 zero-shot과 few-shot 성능이 뛰어나다는 것이 실험적으로 검증되었다.

![논문 인용](/static/images/sam/text-02.png)
![논문 인용](/static/images/sam/text-01.png)

논문 제안한 SAM 모델도 마찬가지로 대규모 데이터셋으로 모델을 학습시켜 foundation model을 구축하는 것을 목표로 하고 있다. 이는 무엇이든 분할하는(Segment Anything) 논문의 목표에 부합한다.

foundatation model을 학습하기 위한 일반적인 방식은 온라인 데이터셋을 획득하는 것 이다. 하지만 인터넷에는 원본 이미지에 상응하는 mask set이 풍부하지 않기 때문에 이들은 해결책으로 독자적인 데이터 엔진을 구축한다. 이에 대한 내용은 흥미로우나 본문에서는 크게 다루지 않겠다. 

## 제안 방법
Data engine을 통해 대량의 마스크를 확보하여 promptable한 모델을 학습시켜 foundation model을 구축하고 무엇이든 분할(segment anything)할 수 있도록 한다.


### 아키텍처
![figrue1-(b)](/static/images/sam/figure1-(b).png)<br><br>
![figure4](/static/images/sam/figure4.png)<br><br>
위 이미지 우측의 빨간색 박스를 보면 mask decoder가 3개의 mask를 출력하는 것을 알 수 있다.
이를 통해 모델이 모호성을 다룰 수 있게 하여 저자가 task로 설정한 "retrun a valid segmentation mask given any prompt."를 달성한다.<br><br>
![논문 인용](/static/images/sam/text-05.png)<br>

모호성이란 아래 이미지와 같이 point prompt가 타조의 머리부분을 가르킬 때 모델이 타조의 머리를 분할해야할지, 타조의 상체를 분할해야할지, 혹은 타조 전체를 분할해야할지 결정하기 어려운 상황을 의미한다. 저자는 사람이 납득할 수 있는 타당한 마스크(valid segmentation mask)를 출력하기 위해 whole, part, and subpart로 구성된 3개의 mask를 모델이 출력하도록 설계했다.<br>
논문에서는 마스크 갯수를 실험적으로 측정해봤을 때 3개인 경우에 한 객체의 대부분의 하위 요소를 포함할 수 있었다고 한다.

![논문 인용](/static/images/sam/text-06.png)<br><br>
![mask decoder](/static/images/sam/image.png)
output tokens은 IoU token(1개)와 mask token(3개)로 구성된다.<br>
### mask decoder 상세 설명
SAM은 프롬프트에 따라 그때그때 분류기 자체를 새로 생성하며 단계별로는 아래와 같다.
1. 정보 교환
- 프롬프트 임베딩(점/박스)과 이미지 임베딩이 두개의 수정된 transformer 디코더 블록을 통과한다. *mask decoder에서는 attention is all you need(트랜스포머 논문)에서 사용한 디코더 블록을 수정해서 사용하기 때문에 "수정된 transformer 디코더 블록" 이라는 용어를 사용함
- 이 때 self-attention(프롬프트 간의) + 양방향 cross-attention(prompt->image, image->prompt)이 적용되어 프롬프트 정보가 이미지 임베딩에 스며들고, 동시에 이미지 정보도 프롬프트(특히 output token)에 스며든다. 
2. 업샘플링
- 이미지 임베딩(다운 샘플링된 저해상도 특징맵)을 다시 업샘플링하여 더 높은 해상도의 픽셀별 특징맵을 만든다.
3. 동적 분류기 생성(정적 분류기를 가지는 CAM 방식과 다름, 아래에서 추가 설명)
- (1) 에서 프롬프트 정보를 흡수한 "output token"을 MLP에 통과시켜 특정 프롬프트에 특화된 1차원 가중치 벡터 생성
- 이 벡터가 바로 "동적 선형 분류기(dynamic linear classifier)"
4. 내적으로 마스크 생성
- 이 동적 분류기 벡터를 (2) 의 업샘플링된 픽셀별 특징맵의 각 위치와 내적한다.
- 그 결과값이 해당 픽셀의 "전경일 확률"이 되며 활성화 함수를 거쳐 분할 마스크가 된다.<br><br>

![decoder01](/static/images/sam/maskdecoder01.png)<br><br>
![decoder02](/static/images/sam/maskdecoder02.png)<br><br>
![decoder03](/static/images/sam/maskdecoder03.png)<br><br>
![decoder04](/static/images/sam/maskdecoder04.png)<br><br>
![decoder05](/static/images/sam/maskdecoder05.png)<br><br>

위 내용도 markdown 형식으로 정리하려다가 귀찮아서.. 당시 논문 읽을 때 정리했던 필기로 대체합니다.<br>
## 실험 결과
Segment Anything 본문에서는 5종류의 실험 중 6.1. Zero-Shot Single Point Valid Mask Evaluation과 6.2. Zero-Shot Text-to-Mask 만을 다룬다. 자세한 실험 내용은 full paper 에서 다룬다. arxiv.org/abs/2304.02643<br>

![논문 인용](/static/images/sam/text-07.png)
![figure7](/static/images/sam/figure7.png)<br>

![논문 인용](/static/images/sam/figure8.png)<br><br>
Text-to-Mask 실험에서 학습 단계에서는 CLIP 이미지 인코더에 데이터 엔진(1, 2단계)에서 사람이 만든 정답(ground truth) 마스크를 crop 하여 학습하며, 추론 단계에서는 텍스트 문자열을 입력으로 하여 CLIP의 텍스트 인코더를 통과시켜 텍스트 임베딩을 얻는다.

![CLIP](/static/images/sam/CLIP.png)<br><br>
![figrue8](/static/images/sam/figure8.png)



## 내 프로젝트에 적용 가능한 점

현재 논문 투고 기간이기 때문에 논문 제출 이후 다른 논문 리뷰에서 언급하겠다.

## 한계 및 개선 아이디어
1\. SAM 모델은 small disconnected components에 대한 환각 문제와 fine structure 를 놓치는 한계가 존재한다고 본문에 나옴. 게다가 domain-specific tools에 비해 성능이 떨어질 수 있다고 함. 내시경 영상의 video-level에서 약지도 학습으로 용종을 localization을 하기에 적합한지 의문이 들었음. but 팀원의 실험에서 좋은 성능이 나왔다고 들었기 때문에 실험환경을 어떻게 setting 했는지 들어볼 필요가 있다고 생각함

![논문 인용](/static/images/sam/text-09.png)

2\. SAM 모델은 image-level 이기 때문에 video-level에 대해 연구한 SAM2 논문을 읽어봐야하겠지만, 근본적으로 SAM은 SA-1B 라는 상대적으로 객체의 크기가 큰 이미지에 대해 학습한 모델인데, 내시경 데이터셋에서 용종을 잘 localization을 할 수 있을지 궁금함. 또한 SAM은 일반적인 지도학습보다 더 많은 지도 요소들 (GT mask + @segment prompt [point, bbox, text 등] ) 이 필요한데 mask가 상대적으로 부족한 용종 데이터셋에서 확보가능할지 알아봐야할 것 같음