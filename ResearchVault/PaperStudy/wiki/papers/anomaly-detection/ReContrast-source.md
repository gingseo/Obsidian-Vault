---
title: "ReContrast: Domain-Specific Anomaly Detection via Contrastive Reconstruction (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[ReContrast]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
대부분의 고급 비지도 이상 탐지(UAD) 방법은 대규모 데이터셋(예: ImageNet)으로 사전학습된 고정(frozen) encoder 네트워크의 feature representation을 모델링하는 데 의존한다. 그러나 자연 이미지 도메인에서 빌려온 encoder로부터 추출된 특징은 산업 검사, 의료 영상과 같은 target UAD 도메인에서 요구되는 특징과 거의 일치하지 않는다. 본 논문에서는 사전학습된 이미지 도메인에 대한 편향을 줄이고 네트워크를 target 도메인에 맞게 조정하기 위해 전체 네트워크를 최적화하는 새로운 epistemic UAD 방법인 ReContrast를 제안한다.

우리는 재구성 오차로부터 이상을 탐지하는 feature reconstruction 접근법에서 출발한다. 본질적으로, contrastive learning의 요소들을 feature reconstruction에 정교하게 내장하여 네트워크가 학습 불안정성(training instability), pattern collapse, identical shortcut에 빠지지 않도록 하는 동시에 encoder와 decoder를 target 도메인에서 함께 최적화한다.

다양한 이미지 도메인에 대한 전이 능력을 입증하기 위해, 널리 쓰이는 두 개의 산업 결함 탐지 벤치마크와 세 개의 의료 영상 UAD 태스크에 걸쳐 광범위한 실험을 수행했으며, 이는 현재의 state-of-the-art 방법들 대비 우리 방법의 우수성을 보여준다. 코드는 https://github.com/guojiajeremy/ReContrast 에서 확인할 수 있다.

# Introduction

#### UAD의 정의와 응용
비지도 이상 탐지(UAD)는 정상 이미지만으로 이상을 인식·위치화해, 산업 결함 탐지·의료 질병 스크리닝 등 라벨링이 어려운 응용 전반에서 쓰인다.

#### 기존 접근 계열 분류와 공통 전제
기존 SOTA는 feature reconstruction·feature distillation·feature memory & modeling 계열로 나뉘는데, 대부분 ImageNet 사전학습 encoder를 특징 추출기로 그대로 얼려 쓴다.

#### 기존 연구 한계 지적 — frozen encoder의 도메인 편향
Feature reconstruction·distillation은 pattern collapse를 막기 위해 encoder를 반드시 고정해야 하며, 이 때문에 자연 이미지로 학습된 frozen network와 다양한 UAD 도메인 사이의 semantic gap을 그대로 안고 간다.

#### 기존 연구 한계 지적 — 얕은 적응의 한계
CFA·SimpleNet 등이 전이 문제를 다루려 했으나 encoder는 고정한 채 뒤에 linear layer 하나만 붙여, 의료 영상처럼 자연 이미지와 먼 도메인에서는 이미 손실된 정보를 복원하기엔 역부족이다.

#### 제안 방법 개요
본 논문은 encoder를 직접 최적화하면 특징 다양성이 저하된다는 관찰에 기반해, contrastive learning의 세 요소(global cosine distance, stop-gradient, 두 encoder로 augmentation 없이 두 view 구성)를 feature reconstruction UAD에 도입하고 hard-mining 전략을 더한 ReContrast를 제안한다. SimSiam·SimCLR류의 사전학습용 SSL과 달리 end-to-end 이상 탐지 방법임을 명시한다.

#### 실험 결과 요약
MVTec AD·VisA 두 산업 벤치마크와 OCT·안저·피부 병변 세 의료 벤치마크에서, memory bank나 handcrafted pseudo anomaly 없이도 기존 SOTA 대비 우수한 성능(MVTec AD I-AUROC 99.5%, 기존 최고 대비 오차 거의 절반 감소)을 달성했다.

# Main Contribution
1. 큰 도메인 격차를 가진 UAD 이미지에서 frozen ImageNet encoder의 전이 능력 저하 문제를 다루는 간단하고 효과적인 방법을 제안한다. Contrastive learning의 세 요소를 feature reconstruction UAD에 정교히 도입해 전체 네트워크를 최적화한다.
2. Contrastive learning의 GAP에서 영감을 받아, point-by-point cosine distance에 globality를 도입한 새 목적함수로 학습 안정성을 높인다.
3. Positive-pair contrastive learning의 stop-gradient를 활용해 encoder의 pattern collapse를 방지한다.
4. 이미지 augmentation이 유발할 수 있는 잠재적 이상을 피하기 위해, contrastive pair를 생성하는 새로운 방법을 제안한다.

# Conclusion
본 연구에서 우리는 도메인 특화 비지도 이상 탐지를 위한 새로운 contrastive learning 패러다임인 ReContrast를 제안한다. 이 방법은 모든 파라미터를 end-to-end로 함께 최적화함으로써 사전학습된 encoder의 전이 능력 문제를 해결한다. Contrastive learning의 핵심 요소들이 epistemic UAD 방법에 정교하게 내장되어 pattern collapse, 학습 불안정성, identical shortcut을 피한다. MVTec AD, VisA, 그리고 세 개의 의료 영상 데이터셋에 대한 광범위한 실험이 우리 방법의 우수성을 입증한다. Encoder를 최적화한다는 아이디어는 자연 이미지 도메인과 거리가 먼 더 다양한 이미지 모달리티에 대한 UAD 방법의 응용을 한층 더 촉진할 수 있을 것이다.
