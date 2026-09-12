---
title: "연구 지도"
tags: [moc]
created: 2026-08-04
updated: 2026-09-01
---

# 연구 지도

## Task별 MOC
- [[Object_Detection_Moc]] — 26편, 가장 많이 다룬 분야
- [[Anomaly_Detection_Moc]] — 2편
- [[Salient_Object_Detection_Moc]] — 2편
- [[AI_Generated_Image_Detection_Moc]] — 1편
- [[Instance_Segmentation_Moc]] — 1편
- [[Visual_Grounding_Moc]] — 1편
- [[Scientific_Critique_Automation_Moc]] — 1편, 새로 생긴 분야(컴퓨터 비전 밖 — 커뮤니티 트렌드 추적 경로로 유입)

## Task를 가로지르는 개념
task 경계와 무관하게 여러 분야에서 반복 등장하는 개념·신호·메커니즘을 모은다. 새 논문을 읽을 때 이 목록부터 훑으면, 표면적 분야가 달라도 이미 아는 메커니즘인지 바로 확인할 수 있다.

- **Reconstruction error를 판별 신호로 쓴다** — [[Self_Reconstruction_Difference_Map]] / [[Latent_Reconstruction_Error]]: "무언가를 재구성해보고, 재구성이 실패하는 정도(오차)를 신호로 쓴다"는 원리가 세 task에서 각기 다른 대상에 적용된다. small-object-detection의 [[2024_ECCV_SR-TOD|SR-TOD]](FPN feature→원본 이미지, 원조)와 [[2025_Sensors_Reconstruction_Error_Guided_Instance_Segmentation|Reconstruction_Error_Guided_Instance_Segmentation]](같은 원리를 instance segmentation·적외선 도메인으로 확장), ai-generated-image-detection의 [[2024_CVPR_LaRE2|LaRE2]](사전학습 diffusion model의 노이즈 예측 오차), anomaly-detection의 [[2023_NeurIPS_ReContrast|ReContrast]]([[ReContrast_Dual_Encoder_Contrastive_Reconstruction]], encoder를 직접 학습시키는 방식이라 나머지와 대조). 재구성 "대상"이 원본 이미지인지 latent인지, 재구성기가 고정인지 학습되는지가 갈래를 가른다.
- **Object query를 이미지 내용에 따라 동적으로 조정한다** — [[Density_Guided_Dynamic_Query]] / [[Pattern_Quality_Aware_Query_Refinement]]: small-object-detection 안에서만 6편([[2024_ECCV_DQ-DETR|DQ-DETR]], [[2025_JSTARS_Density-Aware-DETR|Density-Aware-DETR]], [[2026_ICASSP_IG-DETR|IG-DETR]], [[2026_SSRN_DQP-DETR|DQP-DETR]], [[2025_arXiv_PaQ-DETR|PaQ-DETR]], [[2026_JSTARS_DQA-DETR|DQA-DETR]])이 이 상위 아이디어를 공유하지만, "무엇을 동적으로 조정하는가"(개수 vs 표현 vs 병합)가 갈래를 가른다. 아직 다른 task에서는 등장하지 않았지만, query 기반 구조(DETR 계열)를 쓰는 다른 분야(visual-grounding 등)로 확장될 가능성이 있는 축.
- **불확실성(uncertainty)을 명시적으로 모델링해 파이프라인에 반영한다** — [[Gaussian_Box_Uncertainty_Modeling]] / [[Perception_And_Interaction]] / [[Uncertainty_Masked_Refinement_Attention]]: small-object-detection의 [[2026_TIP_Unc-SOD|Unc-SOD]](박스 좌표 불확실성을 positive sampling 기준으로), salient-object-detection의 [[2025_TIP_Uncertainty_Guided_Refinement|Uncertainty_Guided_Refinement]](예측 saliency map에서 유도한 불확실성을 attention 마스크로). 둘 다 "어디가 불확실한지 알아내 그 부분에 자원을 더 쓴다"는 상위 전략은 같지만 불확실성을 유도하는 방식과 적용 지점이 다르다.
- **학습 시에만 존재하고 추론 시 제거되는 auxiliary branch** — small-object-detection 안에서만 이미 4개 변형([[2024_TGRS_ORFENet|ORFENet]]의 ORB, [[2026_TGRS_FFSSTDNet|FFSSTDNet]]의 FSR, [[2025_TGRS_BAFNet|BAFNet]]의 Boundary-Aware Branch, [[2026_arXiv_CoLR-Det|CoLR-Det]]의 latent restoration branch)이 계보를 이루며, 위 "reconstruction error" 항목과도 원리적으로 겹친다(재구성 기반 auxiliary branch는 사실상 이 계보와 reconstruction 계보의 교집합).

## 지금 무엇에 집중하고 있는가
지금까지 읽은 29편 중 대다수가 small/tiny object detection에 집중되어 있다. 그중 다수는 "기존 detector에 어떤 plug-in 모듈을 추가할 것인가"(feature 강화, label assignment, 연산 가속, 경량화, 구조 개선)를 다루고, 6편은 Deformable DETR을 baseline으로 삼아 object query의 개수·구성을 이미지 내용에 따라 동적으로 조정하는 "dynamic query DETR" 계열([[2024_ECCV_DQ-DETR|DQ-DETR]], [[2025_JSTARS_Density-Aware-DETR|Density-Aware-DETR]], [[2026_ICASSP_IG-DETR|IG-DETR]], [[2025_arXiv_PaQ-DETR|PaQ-DETR]], [[2026_JSTARS_DQA-DETR|DQA-DETR]], [[2026_SSRN_DQP-DETR|DQP-DETR]])이며, 나머지 3편([[2020_ECCV_DETR|DETR]], [[2017_ICCV_Deformable_Convolutional_Networks|Deformable_Convolutional_Networks]], [[2021_ICLR_Deformable-DETR|Deformable-DETR]])은 이 흐름들이 기반하는 순수 foundational 아키텍처 논문이다. 자세한 흐름과 빈틈은 [[Object_Detection_Moc]] 참고.

Salient object detection은 [[2026_Image-and-Vision-Computing_AIMRINet|AIMRINet]](원격탐사, 다중 레벨 feature 통합)이 추가되어 2편이 되었고, 자연 이미지 반복 정제([[2025_TIP_Uncertainty_Guided_Refinement|Uncertainty_Guided_Refinement]])와 서로 다른 접근을 대표한다. Anomaly detection도 [[2024_CVPRW_LogicAL|LogicAL]](edge 조작 기반 논리적/구조적 이상 합성)이 추가되어 2편이 되었으며, [[2023_NeurIPS_ReContrast|ReContrast]](탐지 아키텍처 개선)와 "어떻게 탐지할지" vs "무엇으로 학습시킬지"라는 상호보완적 축을 형성한다. Visual grounding은 [[2025_TGRS_VGRSS|VGRSS]]로 새로 열린 분야로, 원격탐사 선박 영상에서 자연어로 특정 객체를 지목하는 과제를 다룬다 — "모든 객체를 찾는다"는 small-object-detection의 완전성 문제와 달리 "특정 객체 하나만 정확히 찾는다"는 특정성 문제라는 점에서 이 위키에 새로운 축을 더한다. Ai-generated-image-detection, instance-segmentation은 각각 1편씩만 읽은 상태로, 아직 전체 지형을 파악하기엔 이르다. 다만 ai-generated-image-detection의 [[2024_CVPR_LaRE2|LaRE2]]가 쓰는 reconstruction error 기반 판별 신호는 small-object-detection의 [[Self_Reconstruction_Difference_Map]]과 원리적으로 겹치고, instance-segmentation의 [[2025_Sensors_Reconstruction_Error_Guided_Instance_Segmentation|Reconstruction_Error_Guided_Instance_Segmentation]]은 이 원리를 직접 detection에서 segmentation으로, 가시광에서 적외선 도메인으로 확장한 사례라 세 분야를 넘나드는 교차 관점이 뚜렷해지고 있다.

Scientific_Critique_Automation은 [[2026_arXiv_Tree-of-Concerns|Tree-of-Concerns]]로 새로 열린 분야로, 컴퓨터 비전이 아니라 "LLM 멀티에이전트로 논문이 스스로 밝히지 않은 한계를 찾아낸다"는 완전히 다른 주제를 다룬다 — 직접 정독하려던 논문이 아니라 커뮤니티에서 화제가 되어 트렌드 파악 차원에서 챙긴 논문(`source_type: community`)이라는 점도 다른 27편과 구분된다.
