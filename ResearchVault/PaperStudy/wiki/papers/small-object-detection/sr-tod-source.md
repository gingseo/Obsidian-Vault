---
title: "Visible and Clear: Finding Tiny Objects in Difference Map (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[sr-tod]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
Tiny object detection은 대부분의 generic detector가 겪는 핵심 난제 중 하나다. 가장 큰 어려움은 tiny object의 효과적인 feature를 추출하는 데 있다. 기존 방법들은 대개 생성(generation) 기반 feature enhancement를 수행하는데, 이는 허위 texture와 artifact에 심하게 영향을 받아 tiny-object-specific feature를 탐지를 위해 visible하고 clear하게 만들기 어렵다.

이 문제를 해결하기 위해, 우리는 self-reconstructed tiny object detection(SR-TOD) 프레임워크를 제안한다. 우리는 처음으로 탐지 모델에 self-reconstruction 메커니즘을 도입했고, 그것과 tiny object 사이의 강한 상관관계를 발견했다. 구체적으로, 우리는 탐지기의 neck 사이에 reconstruction head를 부과하여, 재구성된 이미지와 입력 사이의 difference map을 구성하는데, 이는 tiny object에 대해 높은 민감도를 보인다. 이는 difference map의 안내 아래 tiny object의 약한 표현을 강화하도록 우리에게 영감을 준다. 따라서 탐지기에 대한 tiny object의 visibility를 향상시킨다. 이를 기반으로, 우리는 tiny feature 표현을 더 clear하게 만들기 위해 Difference Map Guided Feature Enhancement(DGFE) 모듈을 추가로 개발한다.

또한, 우리는 새로운 multi-instance anti-UAV 데이터셋을 추가로 제안한다. 광범위한 실험이 우리 방법의 효과성을 입증한다. 코드는 https://github.com/Hiyuur/SR-TOD 에서 공개되어 있다.

# Introduction

#### 이 논문이 다루는 문제 영역
Tiny object detection(작은 크기의 물체를 식별·분류하는 object detection의 하위 분야, MS COCO 기준 32×32px 이하, AI-TOD 기준 very tiny/tiny/small로 세분화)은 자율주행·산업 검사·보행자 탐지 등 다양한 실세계 응용에서 자주 등장하며 여전히 어려운 과제로 남아 있다.

#### 근본 원인 진단
Tiny object detection의 가장 근본적인 어려움은 정보 손실이며, backbone의 downsampling이 tiny object 신호를 필연적으로 지워 특히 "very tiny" 물체는 탐지기가 식별하기 거의 불가능해진다(Fig. 1).

#### 기존 연구 한계 지적
다수 기존 방법이 GAN 기반 super-resolution으로 tiny object의 왜곡된 구조를 복원하려 시도하지만, 대량의 중대형 샘플이 필요하고 허위 texture·artifact를 만들며 연산 비용이 커 end-to-end 최적화를 복잡하게 만든다.

#### 제안 방법 개요 — 발견
저자들은 image self-reconstruction 메커니즘을 탐지 프레임워크에 처음 도입해, 재구성이 어려운 영역이 backbone에서 정보가 심하게 손실된 tiny object 영역과 대응함을 발견했다 — difference map이 tiny object의 강한 위치·구조 prior가 된다.

#### 제안 방법 개요 — 활용
이 발견을 바탕으로 difference map을 채널 방향으로 reweighting하는 Difference Map Guided Feature Enhancement(DGFE) 모듈을 개발해, reconstruction loss를 tiny object 전용 제약으로 전환한다.

#### 추가 기여 — 데이터셋
평균 크기 약 7.9픽셀(anti-UAV 데이터셋 중 최소)의 새 anti-UAV 데이터셋 DroneSwarms를 수집했으며, 이 데이터셋과 두 개의 다른 데이터셋에서의 실험이 경쟁 방법 대비 우수성을 입증한다.

# Main Contribution
1. Self-reconstructed tiny object detection(SR-TOD) 프레임워크 제안 — difference map과 tiny object 사이의 강건한 연관성을 처음 밝혀, 손실되는 tiny object 정보를 실행 가능한 prior guidance로 전환한다.
2. Difference Map Guided Feature Enhancement(DGFE) 모듈 설계 — tiny object의 feature 표현을 개선하며, generic detector에 쉽고 유연하게 통합 가능하다.
3. 현재 가장 작은 평균 물체 크기를 갖는 anti-UAV용 새 데이터셋 DroneSwarms 제안 — 자체 데이터셋과 두 개의 다른 데이터셋에서 경쟁 방법 대비 효과성을 검증했다.

# Conclusion
이 논문에서 우리는 tiny object detection에서 정보 손실 문제와, 이를 완화하려는 생성 기반 방법들이 직면한 한계를 분석했다. 이를 위해 우리는 image self-reconstruction 메커니즘을 도입하여, difference map을 tiny object의 prior 정보로 구성함으로써 feature를 탐지기에 더 visible하게 만들었다. 그런 다음 우리는 tiny object의 feature 표현을 개선해 더 clear한 표현을 제공하는 Difference Map Guided Feature Enhancement(DGFE) 모듈을 추가로 설계했다.

우리가 제안한 DroneSwarms와 두 개의 다른 데이터셋에서의 실험은 SR-TOD의 우수성과 견고성을 보여준다. 향후 우리는 tiny object에 대해 더 정확한 difference map을 구성하는 더 효과적인 방법을 탐색할 것이다.
