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

## 2차 배치 이후 — Small_Object_Detection 나머지 12개 (아직 미배정)

- [ ] FFSSTDNet
- [ ] Feature_Info_Driven_Gaussian
- [ ] IG-DETR
- [ ] LSOD-YOLO
- [ ] ORFENet
- [ ] QueryDet
- [ ] RS-TOD
- [ ] RTP-Net
- [ ] SR-TOD
- [ ] UAV-DETR
- [ ] Unc-SOD
- [ ] YOFOR

## 다른 프로젝트 (아직 미배정, 재작성 필요)

- [ ] Deformable-DETR — `Projects/논문_Object_Detection_tasks/Deformable-DETR.md`
- [ ] PaQ-DETR — `Projects/논문_Object_Detection_tasks/PaQ-DETR.md`
- [ ] Deformable_Convolutional_Networks — `Projects/논문_General_Deep_Learning_Techniques_tasks/Deformable_Convolutional_Networks.md`
- [ ] Anomaly_Detection 2개 (LogicAL, ReContrast)
- [ ] Instance_Segmentation 1개
- [ ] Salient_Object_Detection 2개
- [ ] Visual_Grounding 1개
- [ ] AI_Generated_Image_Detection 1개

## 이번 작업에서 함께 처리한 것 (완료)

- [x] Object_Detection 프로젝트/폴더 신설, DETR·Deformable-DETR·PaQ-DETR 이동
- [x] General_Deep_Learning_Techniques 프로젝트/폴더 신설, Deformable_Convolutional_Networks 이동
- [x] Small_Object_Detection에 남은 22개는 전부 제목/태그에 small·tiny object detection이 명시적이라 이동 대상 없음(재확인 완료)
- [x] Schema.md 본문 템플릿 대개편 (파이프라인+shape, 코드 블록, "정리" 표, Architecture Design 연동, 메모 콜아웃 규칙)
- [x] `ResearchVault/Architecture Design/`에 3개 노트 생성: `Multi_Head_Self_Attention.md`, `1x1_Convolution.md`, `Bipartite_Matching_Hungarian_Algorithm.md`

## 작업 중 발견해 복구한 사고

- Project Manager UI에서 title 저장 시 파일명이 title-slug로 자동 리네임되고 PaperWiki 속성이 사라지는 사고가 `DQ-DETR.md`에서 실제로 발생 → 파일명·frontmatter 복구 완료. Schema.md의 기존 경고(83-85번째 줄) 그대로 재발이니, Project Manager UI에서 title은 계속 편집 금지.
