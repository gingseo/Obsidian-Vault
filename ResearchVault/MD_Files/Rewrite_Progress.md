# 분석 노트 전면 재작성 진행 상황

`Schema.md`의 새 본문 템플릿(2026-08-28 갱신 — 파이프라인 다이어그램+tensor shape, 코드 블록, "정리" 표, Architecture Design 연동, 모든 제목마다 메모 콜아웃 등)에 맞춰, 기존 분석 노트를 PDF부터 다시 읽어 전면 재작성하는 작업의 진행 상황을 기록한다.

**다른 컴퓨터에서 이어서 하려면**: 이 파일에서 "미완료" 목록의 다음 파일을 골라 해당 PDF(`source` 필드 경로)를 읽고, `Schema.md`의 "본문 구성 — 분석 노트" 절 템플릿대로 재작성한다. 완료하면 이 파일의 체크박스를 갱신한다. 진행 방식과 각 항목의 세부 지침(정리 표 구성, 파이프라인 shape 표기, 코드 블록, 메모 콜아웃 배치 등)은 `Schema.md`를 그대로 따르면 되고, 참고용 완성 예시는 `Projects/논문_Object_Detection_tasks/DETR.md`.

## 완료 (Object_Detection, 이미 새 형식)
- [x] DETR — `Projects/논문_Object_Detection_tasks/DETR.md`

## 1차 배치 — Small_Object_Detection 22개 중 10개

- [x] BAFNet — `Projects/논문_Small_Object_Detection_tasks/BAFNet.md` (Architecture Design 신규 2개: Dilated_Convolution, Dual_Stream_Foreground_Background_Attention)
- [x] CDATOD-Diff — `Projects/논문_Small_Object_Detection_tasks/CDATOD-Diff.md` (Architecture Design 신규: CLIP_Conditioned_Diffusion_Anchor_Sampling.md; 검수 완료)
- [x] CoLR-Det — `Projects/논문_Small_Object_Detection_tasks/CoLR-Det.md` (Concepts/ 신규: Latent_Restoration_Regularization.md; Architecture Design 신규 없음(기존 재사용); 검수 완료)
- [x] DQ-DETR — `Projects/논문_Small_Object_Detection_tasks/DQ-DETR.md` (직접 재작성, 새 Architecture Design 없음(기존 재사용); 검수 완료)
- [x] DQA-DETR — `Projects/논문_Small_Object_Detection_tasks/DQA-DETR.md` (직접 재작성, 새 Architecture Design 없음(기존 재사용); 검수 완료)
- [x] DQP-DETR — `Projects/논문_Small_Object_Detection_tasks/DQP-DETR.md` (직접 재작성, 새 Architecture Design 없음(기존 재사용); jcr_quartile=arXiv는 SSRN 프리프린트 규칙상 정상; 검수 완료)
- [x] Density-Aware-DETR — `Projects/논문_Small_Object_Detection_tasks/Density-Aware-DETR.md` (직접 재작성, 새 Architecture Design 없음(기존 재사용); 검수 완료)
- [x] Detection_Oriented_Rectification — `Projects/논문_Small_Object_Detection_tasks/Detection_Oriented_Rectification.md` (Architecture Design 신규: Mixture_of_Experts_Top_k_Sparse_Gating.md; Concepts 신규: Degradation_Aware_Rectification.md; jcr_quartile null — venue "IEEE TPAMI" 등급 사용자 확인 필요; 검수 시 Discussion 섹션에 잘못 복사된 템플릿 지침 문구 1건 발견·제거)
- [x] FANet — `Projects/논문_Small_Object_Detection_tasks/FANet.md` (Architecture Design 신규: Squeeze_And_Excitation_Channel_Attention.md; title이 원제로 잘못 들어가 있던 것도 short slug로 수정됨; jcr_quartile null 상태 — venue "Remote Sensing (MDPI)" 등급 사용자 확인 필요; 검수 완료)
- [x] FFCA-YOLO — `Projects/논문_Small_Object_Detection_tasks/FFCA-YOLO.md` (Architecture Design 신규 3개: Global_Context_Modeling_GAP_GMP, Channel_Reweight_Concat, Partial_Convolution; 검수 완료)

## 2차 배치 — Small_Object_Detection 12개 중 4개

- [x] FFSSTDNet — `Projects/논문_Small_Object_Detection_tasks/FFSSTDNet.md` (새 Architecture Design/Concepts 없음(CFD/FSR은 논문 내부 구현 디테일로 판단); 검수 완료, 수치 오류 없음)
- [x] Feature_Info_Driven_Gaussian — `Projects/논문_Small_Object_Detection_tasks/Feature_Info_Driven_Gaussian.md` (새 문서 없음, 기존 Position_Gaussian_Saliency_Map·Gaussian_Box_Uncertainty_Modeling 재사용; 검수 시 Table 2 AI-TOD RFLA+ours 행 수치 오류 발견·수정: AP 21.7→22.6(+0.9)/AP_vt 8.3→8.2(-0.1)였던 것을 21.7→23.9(+2.3)/8.3→8.5(+0.2)로 정정)
- [x] IG-DETR — `Projects/논문_Small_Object_Detection_tasks/IG-DETR.md` (새 문서 없음, 기존 Density_Guided_Dynamic_Query에 이미 등장 논문으로 등재되어 있어 링크만; 검수 완료, 수치 오류 없음)
- [x] LSOD-YOLO — `Projects/논문_Small_Object_Detection_tasks/LSOD-YOLO.md` (Architecture Design 신규 3개: Large_Separable_Kernel_Attention, Normalization_Based_Attention_Module, Dysample_Dynamic_Upsampling; 기존 Lightweight_Cross_Layer_Output_Reconstruction 재사용; jcr_quartile null — venue "Expert Systems With Applications (Elsevier)" 등급 사용자 확인 필요; 검수 완료)

## 3차 배치 — Small_Object_Detection 8개 중 4개

- [x] ORFENet — `Projects/논문_Small_Object_Detection_tasks/ORFENet.md` (새 Architecture Design/Concepts 없음(MRFAFEM은 논문 내부 세부 튜닝, Object Reconstruction Branch는 기존 Self_Reconstruction_Difference_Map과 메커니즘 달라 미생성); 검수 완료, 오류 없음)
- [x] QueryDet — `Projects/논문_Small_Object_Detection_tasks/QueryDet.md` (새 문서 없음, 기존 Cascade_Sparse_Query 재사용; 검수 완료, 오류 없음)
- [x] RS-TOD — `Projects/논문_Small_Object_Detection_tasks/RS-TOD.md` (새 문서 없음, 기존 Remote_Sensing_Attention_Module 재사용; Fig. 3 block 번호 B1~B34 파이프라인에 반영; jcr_quartile null — venue "Remote Sensing Applications: Society and Environment (Elsevier)" 등급 사용자 확인 필요; 검수 완료)
- [x] RTP-Net — `Projects/논문_Small_Object_Detection_tasks/RTP-Net.md` (새 문서 없음, 기존 Collaborative_Receptive_Field_Texture_Optimization 재사용; 검수 시 AWEM 단독 ablation mAP50:95 오기재(66.3→66.6) 및 DIOR 결과표 YOLOv8n→YOLOv8 표기 오류 발견·수정)

## 4차 배치 — Small_Object_Detection 마지막 4개 (완료, 22편 전체 재작성 마무리)

- [x] SR-TOD — `Projects/논문_Small_Object_Detection_tasks/SR-TOD.md` (새 문서 없음, 기존 Self_Reconstruction_Difference_Map 재사용; 검수 완료, 오류 없음)
- [x] UAV-DETR — `Projects/논문_Small_Object_Detection_tasks/UAV-DETR.md` (새 문서 없음, 기존 Frequency_Domain_Feature_Enhancement 재사용; PDF Fig. 2 캡션에 삽입된 프롬프트 인젝션 문구 재발견·무시, 보안 참고에 기록; 검수 완료)
- [x] Unc-SOD — `Projects/논문_Small_Object_Detection_tasks/Unc-SOD.md` (새 문서 없음, 기존 Gaussian_Box_Uncertainty_Modeling·Perception_And_Interaction 재사용; jcr_quartile null — venue "IEEE TIP" 등급 사용자 확인 필요; 검수 완료)
- [x] YOFOR — `Projects/논문_Small_Object_Detection_tasks/YOFOR.md` (새 문서 없음, 기존 Class_Balanced_Spatial_Copy_Paste 재사용; 검수 중 CBM copy distance 서술(1/4배 vs 1배) 논문 자체의 내적 불일치 발견 — 단일 값으로 확정하지 않고 애매함을 명시하도록 YOFOR.md와 concept 문서 둘 다 수정)
- [x] Small_Object_Detection_Moc.md·Comparisons/Small_Object_Detection_Approaches.md 최종 점검 — 22편 전체 이미 정확히 반영되어 있어 추가 갱신 불필요

## Small_Object_Detection 22편 전면 재작성 완료 (2026-08-31)

## 5차 배치 — 타 task 9개 중 5개 (완료)

- [x] Deformable-DETR — `Projects/논문_Object_Detection_tasks/Deformable-DETR.md` (새 문서 없음; 검수 시 Appendix A.1 복잡도 공식 오류 발견·수정, Table 2 BiFPN 행 누락 보완)
- [x] PaQ-DETR — `Projects/논문_Object_Detection_tasks/PaQ-DETR.md` (**중대 발견**: 옛 노트가 설명한 메커니즘 "PAC로 후보 클러스터링+QAP로 가지치기"가 실제 논문에 전혀 없는 완전한 허구였음 — PDF 전문 검색 결과 candidate/prune/cluster 등 0건. 실제 논문은 "공유 패턴 기반 query 표현 재구성 + 품질 기반 1:多 assignment". 이 허구 내용이 퍼져있던 8개 문서(Comparisons/Small_Object_Detection_Approaches.md, Moc/Small_Object_Detection_Moc.md, Moc/Small_Object_Detection_계보.md, Concepts/Pattern_Quality_Aware_Query_Refinement.md, DQ-DETR.md, IG-DETR.md, DQA-DETR.md) 전부 실제 내용 기준으로 정정 완료)
- [x] Deformable_Convolutional_Networks — `Projects/논문_General_Deep_Learning_Techniques_tasks/Deformable_Convolutional_Networks.md` (새 문서 없음, 기존 Deformable_Sampling_Offset 재사용; 검수 완료, Table 3 누락 행 2개 보완)
- [x] LogicAL — `Projects/논문_Anomaly_Detection_tasks/LogicAL.md` (새 Architecture Design 없음, 기존 Edge_Controlled_Anomaly_Synthesis 재사용; 옛 Project 링크 오류 수정)
- [x] ReContrast — `Projects/논문_Anomaly_Detection_tasks/ReContrast.md` (기존 원제 콜아웃이 없어 arXiv 링크 신규 확보(https://arxiv.org/abs/2306.02602); 검수 시 Table 6 Config.E−gl 행 누락 발견·보완; 새 Architecture Design 1x1_Convolution 링크 추가(기존 문서 재사용))

## 6차 배치 — 타 task 나머지 5개 (완료, 위키 전체 재작성 마무리)

- [x] Reconstruction_Error_Guided_Instance_Segmentation — 기존 서술이 PDF와 일치(허구 없음), Table 6 수치 표기 모호성 정정
- [x] AIMRINet — 기존 서술 정확, 원 논문 자체의 Table 1 EORSSD 행 내적 불일치 발견해 그대로 문서화(노트 오류 아님)
- [x] Uncertainty_Guided_Refinement — Table I 컬럼 매핑 오류(Sm/Fβw 혼동, ICON-R 수치) 발견·정정
- [x] VGRSS — 재작성 중 누락된 관련 링크(YOFOR, Density-Aware-DETR) 복구
- [x] LaRE2 — DIRE baseline ACC 오기재(67.2→74.0) 발견·정정, abstract-표 수치 불일치는 원 논문 자체 문제로 문서화
- [x] 5편 모두 PaQ-DETR류 허구 메커니즘 없음 확인

## 위키 전체 분석 노트 재작성 완료 (2026-08-31) — Small_Object_Detection 22편 + 타 task 10편(PaQ-DETR 오염 정화 포함) = 총 32편

## 부록 — 수식 표기 정정 (2026-08-31)

재작성 과정에서 일부 노트가 코드 인라인(백틱) 안에 LaTeX 첨자 문법(`^{...}`, `_{...}`)을 그대로 섞어 써서 Obsidian이 수식으로 렌더링하지 못하고 글자 그대로 깨져 보이는 문제가 발견됐다. Schema.md에 "수식 표기 규칙"(MathJax `$...$` 사용) 절을 신설하고, 아래 9개 파일의 깨진 인라인 수식을 MathJax로 정정했다:
Deformable-DETR, PaQ-DETR, CDATOD-Diff, FANet, DQP-DETR, CoLR-Det, Uncertainty_Guided_Refinement, Architecture Design/1x1_Convolution, Architecture Design/Squeeze_And_Excitation_Channel_Attention.

## 이번 작업에서 함께 처리한 것 (완료)

- [x] Object_Detection 프로젝트/폴더 신설, DETR·Deformable-DETR·PaQ-DETR 이동
- [x] General_Deep_Learning_Techniques 프로젝트/폴더 신설, Deformable_Convolutional_Networks 이동
- [x] Small_Object_Detection에 남은 22개는 전부 제목/태그에 small·tiny object detection이 명시적이라 이동 대상 없음(재확인 완료)
- [x] Schema.md 본문 템플릿 대개편 (파이프라인+shape, 코드 블록, "정리" 표, Architecture Design 연동, 메모 콜아웃 규칙)
- [x] `ResearchVault/Architecture Design/`에 3개 노트 생성: `Multi_Head_Self_Attention.md`, `1x1_Convolution.md`, `Bipartite_Matching_Hungarian_Algorithm.md`

## 작업 중 발견해 복구한 사고

- Project Manager UI에서 title 저장 시 파일명이 title-slug로 자동 리네임되고 PaperWiki 속성이 사라지는 사고가 `DQ-DETR.md`에서 실제로 발생 → 파일명·frontmatter 복구 완료. Schema.md의 기존 경고(83-85번째 줄) 그대로 재발이니, Project Manager UI에서 title은 계속 편집 금지.
