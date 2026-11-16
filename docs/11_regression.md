# 11주차 팀 과제: DAG와 복합표본 선형회귀분석

## C-1. 인과그래프(DAG) 및 보정집합 설정

### 1. 인과그래프 (Mermaid DAG)

~~~mermaid
flowchart LR
    classDef exposure fill:#ffcccc,stroke:#333,stroke-width:2px;
    classDef outcome fill:#cce5ff,stroke:#333,stroke-width:2px;
    classDef confounder fill:#e6ccff,stroke:#333,stroke-width:1px;
    classDef mediator fill:#ffffcc,stroke:#333,stroke-width:1px;

    Genertn["1인 가구 여부<br>(genertn_rc)"]:::exposure --> KHEI["식생활평가지수 총점<br>(HE_khei)"]:::outcome
    
    Age["연령 (age)"]:::confounder --> Genertn
    Age --> KHEI
    
    Sex["성별 (sex)"]:::confounder --> Genertn
    Sex --> KHEI
    
    Incm["가구소득 (incm)"]:::confounder --> Genertn
    Incm --> KHEI
    
    Edu["교육수준 (edu)"]:::confounder --> Genertn
    Edu --> KHEI

    Genertn --> Diet["식사 형태<br>(외식·결식 빈도)"]:::mediator
    Diet --> KHEI
~~~

* **화살표 근거**:
  * **연령·성별 → 가구 형태 및 KHEI**: 인구학적 특성은 가구 구성(고령층의 1인 가구 증가 등)과 식습관 형성에 시간적으로 선행함.
  * **교육수준·가구소득 → 가구 형태 및 KHEI**: 사회경제적 지위(SES)는 독립 주거 선택 가능성과 건강한 식재료 구매력에 모두 영향을 미치는 선행 요인임.
  * **1인 가구 → 식사 형태 → KHEI**: 1인 가구화가 혼밥, 외식 증가, 결식 등의 식사 행태 변화를 유발하고, 이것이 다시 식생활평가지수 저하로 이어지는 매개 경로를 형성함.

---

### 2. 공변량 역할 정의표 및 최소 보정집합

| 변수명 | 변수 내용 | 역할 분류 | 근거 및 모형 반영 여부 |
| :--- | :--- | :---: | :--- |
| `genertn_rc` | 1인 가구 여부 (1: 1인, 0: 다인) | **노출 (Exposure)** | 연구의 주요 관심 독립변수 (모형 1, 2 포함) |
| `HE_khei` | 식생활평가지수 총점 (0–100점) | **결과 (Outcome)** | 연구의 주요 관심 종속변수 (모형 1, 2 포함) |
| `age` | 연령 (만 나이) | **교란 (Confounder)** | 고령층일수록 1인 가구 비율이 높고 식생활 점수도 달라지므로 보정집합에 포함 |
| `sex` | 성별 (남성/여성) | **교란 (Confounder)** | 성별에 따른 가구 구성 차이와 식생활 관리 수준 차이가 존재하므로 보정집합에 포함 |
| `incm` | 가구소득 (4분위) | **교란 (Confounder)** | 소득 수준이 주거 형태와 식품 구매력 모두에 선행 영향을 주므로 보정집합에 포함 |
| `edu` | 교육수준 (4범주) | **교란 (Confounder)** | 교육수준이 가구 구성 및 영양 지식·식습관에 선행 영향을 주므로 보정집합에 포함 |
| 식사 형태 | 결식·외식 빈도 등 | **매개 (Mediator)** | 1인 가구와 KHEI 사이의 중간 경로이므로 **총효과(Total Effect)** 추정을 위해 모형 2에서 제외 |

* **추정 대상 (Estimand)**: 1인 가구 여부가 식생활평가지수(KHEI) 총점에 미치는 **총효과 (Total Effect)**
* **최소 보정집합 (Minimum Adjustment Set)**: `{연령(age), 성별(sex), 가구소득(incm), 교육수준(edu)}`

---

### 3. 완전 사례(Complete Case) 분석 대상 확인

* **Table 1 기본 분석 대상자 (`subpop_t1 == 1`)**: **5,816명**
* **11주차 회귀분석 완전 사례 대상자 (`subpop_reg == 1`)**: **5,652명**
* **인원 차이**: **164명** 감소 (최소 보정집합 변수 중 가구소득·교육수준 등 공변량 결측 제외)

---
