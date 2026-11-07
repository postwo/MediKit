<div align="center">

# 🏭 MediKit Twin

**병원 맞춤형 의료 키트 피킹·포장 공정을 Unity에서 재현하는 디지털 트윈 시뮬레이션**

주문 CSV를 읽어 키트를 만들고, 로봇이 구성품을 담아 검사·포장·라벨·하역까지 한 셀 안에서 처리합니다.

</div>

<br/>

## 📌 프로젝트 소개

주문 CSV(순번, 병원명, 키트 코드, 수량)를 읽어 키트(**EM / BL / DX**) 단위로 풀고, 로봇암이 자재 선반에서 구성품을 집어 키트 박스에 담습니다. 박스는 컨베이어로 이동해 CNN 검사를 받고, 정상 박스는 뚜껑 덮기와 라벨 부착을 거쳐 AGV가 하역장으로 옮깁니다. 불량 박스는 푸셔가 밀어 불량 박스에 모읍니다.

| 항목 | 내용 |
| :-- | :-- |
| 목적 | 의료 키트 포장 공정 전체를 가상 셀로 재현하고 검사 CNN까지 시험 |
| 엔진 | Unity 6000.6.0f1, URP 17.6 |
| 로봇 | 로봇암 A(FR5), 로봇암 B(UR3), 하역 AGV |
| 검사 | 슬롯 크롭 CNN(ONNX, Sentis 추론) + 규칙 판정 |
| 팀 | AURIX Robotics (2인) |
| 요구사항 | [AURIX_SRS.docx](docs/AURIX_SRS.docx) — 코드와 문서의 `FR-xx`, `NFR-xx`는 이 SRS의 ID |

<br/>

## 🎯 핵심 기능

| 기능 | 내용 |
| :-- | :-- |
| 📋 주문 처리 | CSV 주문을 키트 단위로 풀고, 재고가 모자라면 주문을 보류했다가 보충 후 맨 앞에서 재개 |
| 📦 재고 관리 | 자재 선반 12칸(A~C행 × 1~4열)의 재고를 세고, 소진되면 30초 뒤 자동 보충 |
| 🦾 키팅 | 이동 베이스 위의 로봇암 A(FR5)가 선반에서 집어 키트 박스 슬롯에 배치 |
| 🔍 CNN 검사 | 슬롯별 크롭을 빈 슬롯 또는 구성품 12종으로 분류하고 박스 단위로 정상/불량 판정 |
| 🧱 불량 배출 | 불량 박스를 푸셔가 밀어 불량 박스(RejectBin)에 모음 |
| 🏷 팩킹·라벨 | 로봇암 B(UR3)가 뚜껑을 덮고, 뚜껑이 닫히면 키트 라벨 자동 표시 |
| 🚚 하역 | AGV가 컨베이어 끝의 박스를 집어 하역장의 키트별 구역에 내려놓음 |
| 📊 대시보드 | 공정 현황, 주문, 재고, 로그, 배속, 카메라 패널 |

<br/>

## 🖼 시뮬레이션 화면

Unity 셀 전체 모습입니다. 키팅 → 비전 검사 → 불량 분리 → 뚜껑 덮기 → AGV 하역 순으로 컨베이어를 따라 진행합니다.

<div align="center">
  <img src="docs/images/cell_overview.png" alt="MediKit Twin Unity 셀 구성" width="100%" />
</div>

<br/>

```
주문(CSV) → 키팅(로봇암 A) → 컨베이어 → 검사(CNN) ─정상→ 팩킹(로봇암 B, 뚜껑) → 라벨 → 컨베이어 끝 → 하역(AGV) → 하역장
                                              └─불량→ 푸셔 → 불량 박스(RejectBin)
```

| 공정 | 하는 일 | 주요 코드 |
| :-- | :-- | :-- |
| 주문 | CSV를 읽어 키트 단위로 풀고, 재고가 모자라면 주문을 보류 | `Scripts/Orders`, `Scripts/Recipes` |
| 재고 | 자재 선반 12칸의 재고를 셈 | `Scripts/Inventory` |
| 키팅 | 로봇암 A(FR5)가 선반에서 집어 키트 박스 슬롯에 놓음 | `Scripts/ArmA`, `Scripts/Mobile` |
| 컨베이어 | 박스를 검사, 팩킹, 끝 구간으로 이동 | `Scripts/Conveyor` |
| 검사 | 슬롯별 크롭을 CNN으로 분류해 정상/불량 판정(규칙 판정도 지원) | `Scripts/Inspection`, `Scripts/ML` |
| 푸셔 | 불량 박스를 밀어 불량 박스에 넣음 | `Scripts/Pusher` |
| 팩킹 | 로봇암 B(UR3)가 뚜껑을 집어 덮음 | `Scripts/ArmB` |
| 라벨 | 뚜껑에 키트 라벨을 붙임 | `Scripts/Label` |
| 하역 | AGV가 박스를 하역장의 키트별 구역(EM, BL, DX)에 내려놓음 | `Scripts/Unloader` |
| 대시보드 | 공정 현황, 주문, 재고, 로그, 배속, 카메라 패널 | `Scripts/Dashboard`, `Scripts/Demo` |

<br/>

| 키팅 (로봇암 A) | 검사 슬롯 |
| :-: | :-: |
| <img src="docs/images/kitting_arm_a.png" alt="로봇암 A 키팅" width="100%" /> | <img src="docs/images/inspection_slot_crop.png" alt="키트 박스 슬롯 검사 화면" height="260" /> |
| 선반에서 구성품을 집어 지그 위 키트 박스 슬롯에 담음 | 키트 박스 슬롯을 크롭해 CNN으로 정상·불량 판정 |

| 불량 배출 (푸셔) | 하역 (AGV) |
| :-: | :-: |
| <img src="docs/images/pusher_reject_bin.png" alt="푸셔와 불량 박스" width="100%" /> | <img src="docs/images/unloading_agv.png" alt="하역 AGV와 하역장" width="100%" /> |
| 불량 키트를 컨베이어 밖 불량 박스로 밀어 분리 | 컨베이어 끝에서 박스를 집어 경로를 따라 하역장으로 이동 |

<br/>

## 🎬 시연 영상

주문을 받아 키팅, 검사, 팩킹, 하역까지 한 셀을 처리하는 과정입니다. 오른쪽 대시보드에서 공정 현황과 재고를 함께 볼 수 있습니다.

<p align="center">
  <img src="docs/images/demo.gif" alt="MediKit Twin 시연 (미리보기)" width="720" />
</p>

🎬 [시연 영상 전체 보기 (YouTube)](https://www.youtube.com/watch?v=0DOmODBqz7Q)

<br/>

## 🏗 공정 구조

<div align="center">
  <img src="docs/AURIX_%EA%B3%B5%EC%A0%95%ED%9D%90%EB%A6%84%EB%8F%84.png" alt="MediKit Twin 공정 흐름도" width="100%" />
</div>

<br/>

## 🧩 설계 규칙

- **신호로만 소통**: 설비끼리 직접 참조하지 않고 `CellEvents`의 신호(`KitReady`, `AtInspection`, `InspectionResult`, `KitShipped` 등)로만 주고받습니다. 신호 목록은 [INTERFACES.md](docs/INTERFACES.md)에 있습니다.
- **시뮬레이션 시간**: 시간은 모두 `SimClock`(배속 반영)을 따릅니다. 예외는 설비 주기 통지입니다.
- **판정의 단일 출처**: 키트 판정 결과의 출처는 `KitData.Verdict` 하나입니다.
- **재현성**: 같은 시드와 같은 주문 파일은 같은 결과를 냅니다(NFR-06).

<br/>

## 🛠 기술 스택

| 영역 | 스택 |
| :-- | :-- |
| 시뮬레이션 | Unity 6000.6.0f1, URP 17.6, Input System 1.20 |
| 추론 | `com.unity.ai.inference` 2.6.1 (Sentis, ONNX) |
| 로봇 모델 | URDF Importer v0.5.2, FR5 / UR3 |
| CNN 학습 | Python 3.9~3.13, PyTorch, 합성 데이터셋 |
| 시험 | Unity Test Runner (EditMode / PlayMode, 모듈별 어셈블리) |

<br/>

## 📚 문서

| 문서 | 설명 |
| :-- | :-- |
| [docs/README.md](docs/README.md) | 상세 안내 — 요구 환경, 폴더 구조, 에디터 도구, 시험, 협업 규칙 (기존 README) |
| [INTERFACES.md](docs/INTERFACES.md) | 공통 인터페이스(CellEvents, KitData, RecipeSO) |
| [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | 만난 문제와 원인 · 해결 (상세 기록은 `docs/troubleShooting/`) |
| [MLTraining/README.md](MLTraining/README.md) | CNN 학습과 성능 측정 |
| [COMMIT_RULES.md](COMMIT_RULES.md) | 브랜치 전략 · 커밋 메시지 · PR 규칙 |
| [CONVENTION.md](CONVENTION.md) | 협업, 씬, 코딩 컨벤션 |

<br/>

## 🚀 빠른 시작

```bash
git clone https://github.com/postwo/MediKit.git
cd MediKit
git config core.hooksPath .githooks   # 커밋 규칙 자동 검사
```

1. Unity Hub에서 이 폴더를 엽니다(에디터 6000.6.0f1). 처음 열 때 패키지를 받느라 시간이 걸립니다.
2. `Assets/_Project/Scenes/Scene_Cell.unity`를 엽니다. 공정 전체가 이 씬 하나에 있습니다.
3. Play를 누릅니다. 주문은 `Assets/StreamingAssets/Orders/orders.csv`를 읽습니다.

주문 CSV는 첫 줄이 머리글이고 열은 `순번,병원명,키트 코드,수량`입니다.

```csv
순번,병원명,키트 코드,수량
1,A병원,EM,2
2,B병원,DX,3
3,C병원,BL,4
```

시나리오(누락 확률, 설비 실패 키트, 난수 시드, 재고)는 `Assets/_Project/Config/ScenarioConfig_Default.asset`에서 바꿉니다. 폴더 구조, 에디터 도구, 시험 방법은 [docs/README.md](docs/README.md)를 참고하세요.
