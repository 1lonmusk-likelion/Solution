# Nandoc Solution (난독화 복원 프로젝트)

이 프로젝트는 난독화된 한국어 텍스트(야민정음, 오타 등)를 원문으로 복원하는 AI 솔루션입니다.
최종 솔루션은 두 가지 아키텍처(**Double**, **Single**)로 제공됩니다.

##  파일 구성 (Project Structure)

| 파일명 | 설명 |
|--------|------|
| **P01_Nandoc_solution_double.ipynb** | **Double Architecture** 최종 솔루션 코드.<br>Classifier와 두 개의 특화 모델을 연동하여 복원 수행. |
| **P01_Nandoc_solution_single.ipynb** | **Single Architecture** 최종 솔루션 코드.<br>Classifier와 태그 기반 통합 모델을 연동하여 복원 수행. |
| **dict+jamoB+KoBART_cleanData_yamin.ipynb** | **야민정음 특화 모델** 학습 노트북.<br>Double Architecture의 한 축을 담당하며, 딕셔너리와 자모 분리를 활용해 학습. |
| **FinalModel1_3.ipynb** | **오타/일반 난독화 특화 모델** 학습 노트북.<br>Double Architecture의 다른 한 축을 담당. |
| **specialtag+KoBART_cleanData_Final.ipynb** | **Single(통합) 모델** 학습 노트북.<br>스페셜 태그를 활용해 여러 난독화 기법(야민, 오타 등)을 동시에 학습. |
| **Classifier15k.ipynb** | 난독화 유형 분류기 (Signal Classifier). |

---

##  아키텍처 설명

### 1. Double Architecture (P01_Nandoc_solution_double)
**"분류 후 전문 모델 라우팅" 방식**
난독화 유형을 먼저 파악하고, 해당 유형을 가장 잘 처리하는 전문가 모델에게 작업을 맡깁니다.

- **작동 원리**:
  1. **Classifier**: 입력 데이터의 난독화 패턴(오타 vs 야민정음)을 분석합니다.
  2. **Routing**: 분석 결과에 따라 데이터를 갈라서 각각 동시에 구동 중인 전용 모델로 보냅니다.
     -  **야민정음 모델**: dict+jamoB... 에서 학습된 모델이 처리.
     -  **오타 모델**: FinalModel1_3... 에서 학습된 모델이 처리.
  3. **Inference**: 각 기법을 집중적으로 학습한 두 모델이 병렬로 존재하며, 배정받은 요청을 해독합니다.

### 2. Single Architecture (P01_Nandoc_solution_single)
**"태그 기반 통합 모델" 방식**
하나의 모델이 다양한 난독화 기법을 모두 이해하도록 학습된 구조입니다. 더 많은 기법을 유연하게 수용할 수 있습니다.

- **작동 원리**:
  1. **Training**: 학습 단계에서 한글로 작성된 **Special Tag**를 데이터에 부착하여 학습했습니다. 모델은 이 태그를 통해 어떤 방식으로 해독해야 할지 문맥을 배웁니다.
  2. **Inference**:
     - **Classifier**가 먼저 데이터에 적절한 태그를 미리 붙여서 모델에게 보냅니다.
     - **통합 모델**은 텍스트에 붙은 태그를 기반으로 문맥을 파악하고 해독을 수행합니다.
- **특징**: 두 개의 아키텍처를 정리하여 하나의 통합된 모델로 구현했습니다. 추후 난독화 기법이 늘어나더라도 태그만 추가하면 되므로 확장성이 뛰어납니다.

##  주요 기술
- **KoBART**: 한국어 텍스트 생성 모델을 베이스로 fine-tuning 하였습니다.
- **Jamo Decomposition (자모 분리)**: 한글의 특성을 고려하여 텍스트를 자모 단위(초성, 중성, 종성)로 분해하여 처리합니다. 이는 야민정음과 같은 시각적/구조적 난독화를 처리하는 데 핵심적인 역할을 합니다.
