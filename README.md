# 무료 앱은 정말 무료인가?
## Google Play App Data Safety 기반 개인정보 수집·공유와 데이터 수익화 신호 분석

> 소프트웨어융합학과 3학년 김현정 (학번: 2024105245)

---

## 개요

무료 앱의 '0원'이라는 가격 뒤에 **데이터 수집이라는 실질적 비용**이 존재함을 데이터로 실증한 연구입니다.  
Google Play Store의 Data Safety 공시 데이터와 OS 레벨 권한 요청을 통합 분석하여, 무료 앱의 데이터 수집·공유 구조와 수익화 연관성을 검증합니다.

---

## 연구 가설

| 가설 | 내용 | 결과 |
|------|------|------|
| **가설 1** | 무료 앱은 유료 앱보다 데이터 수집 범위가 더 넓을 것이다 | 지지 |
| **가설 2** | 무료 앱이 공유하는 데이터는 앱 기능 제공보다 수익화 목적과 결합되는 비율이 더 높을 것이다 | 지지 |
| **가설 3** | Data Safety 기반 수집·공유 특성은 고위험 앱 여부를 유의미하게 구분할 수 있을 것이다 | 지지 |

---

## 주요 결과

### 가설 1 — 무료 vs 유료 데이터 수집 범위
- 무료 앱 평균 수집 카테고리: **11.89개** / 유료 앱: **2.85개** (단순 비교 **4.2배**)
- Mann-Whitney U 검정: p = 1.67e-129 (유의수준 0.001에서 통계적으로 유의)
- Cliff's δ = 0.7717 → 무료 앱이 유료 앱보다 수집 카테고리가 많을 확률 **88.6%** (large effect)
- Negative Binomial Regression (교란변수 통제 후): **IRR = 2.12** (p < 0.001)

### 가설 2 — 수익화 목적 결합 비율
- 무료 앱 공유 데이터 중 수익화 목적 결합 비율: **79.4%**
- 앱 기능 제공 목적 결합 비율: **40.0%** (약 39%p 차이)
- 95% 클러스터 부트스트랩 신뢰구간이 두 목적 간 전혀 겹치지 않음 → 구조적 차이 확인

### 가설 3 — 고위험 앱 분류 모델 성능
| 지표 | 값 |
|------|-----|
| Accuracy | 0.9137 |
| Precision | 0.9048 |
| Recall | 0.8482 |
| F1-Score | 0.8756 |
| **ROC-AUC** | **0.9720** |

- 5-Fold 교차검증: ROC-AUC **0.977 ± 0.007** (과적합 없음)
- 핵심 변수: `unified_data_scope` (전체 중요도의 **62.2%** 기여)

---

## 데이터셋

- **수집 앱 수**: 1,562개 (Google Play Store)
- **수집 도구**: `google-play-scraper` 라이브러리
- **데이터 파일**: `realdata_v2.csv`

### 주요 변수

| 변수명 | 설명 |
|--------|------|
| `collected_data_count` | 수집 데이터 항목 수 |
| `shared_data_count` | 공유 데이터 항목 수 |
| `total_data_scope` | 수집 + 공유 합산 |
| `sensitive_data_count` | 민감 데이터 항목 수 |
| `data_risk_score` | 수익화 목적 결합 항목 수 (가설2·3 핵심 변수) |
| `permission_count` | OS 레벨 권한 요청 카테고리 수 |
| `unified_data_scope` | Data Safety 신고 카테고리 ∪ 권한 기반 카테고리 (가설1 핵심 변수) |

---

## 분석 방법

```
1. 데이터 수집 및 전처리
   ├── google-play-scraper로 Data Safety, 권한, 수익화 변수 수집
   ├── 결측값 처리 (핵심 변수 결측 5건 제거, score/reviews/ratings 결측 50건 유지)
   └── unified_data_scope = Data Safety 카테고리 ∪ OS 권한 카테고리 (합집합)

2. 가설 1 검증
   ├── 기술통계 비교 (평균, 중앙값)
   ├── Shapiro-Wilk 정규성 검정
   ├── Mann-Whitney U 검정 (비모수)
   ├── Cliff's δ 효과 크기
   └── Negative Binomial Regression (교란변수: 카테고리·설치수·광고·인앱구매)

3. 가설 2 검증
   ├── 분석 단위: 앱 단위 → 데이터 항목 단위
   ├── 수익화 목적: Advertising/marketing, Analytics, Personalization
   └── 95% 클러스터 부트스트랩 신뢰구간

4. 가설 3 검증
   ├── KMeans 군집화 (k=2, Elbow Method + Silhouette Score)
   ├── 고위험 임계값: unified_data_scope ≥ 17
   ├── Random Forest 분류 (class_weight='balanced')
   └── 5-Fold 교차검증
```

---

## 파일 구조

```
FreeApp_data_analasys/
├── 분석.ipynb              # 전체 분석 코드 (81개 셀)
├── 데이터분석 보고서.pdf    # 연구 보고서
├── realdata_v2.csv         # 분석 데이터 (1,562개 앱)
└── README.md
```

---

## 실행 환경

```python
# 주요 라이브러리
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import statsmodels.api as sm
from sklearn.ensemble import RandomForestClassifier
from sklearn.cluster import KMeans
from sklearn.model_selection import train_test_split, cross_val_score
```

---

## 실용적 산출물 — Chrome 확장 프로그램

분석 결과를 실용화한 **App Risk Scanner** 크롬 확장 프로그램을 개발하였습니다.

- Google Play Store 앱 페이지에서 Data Safety 정보를 자동 파싱
- `unified_data_scope ≥ 17` 기준으로 고위험 앱 자동 탐지 및 경고 표시
- 사용자가 앱 설치 전 데이터 위험 수준을 직접 확인 가능

> Chrome Web Store: [App Risk Scanner](https://chromewebstore.google.com/detail/app-risk-scanner/hdbgjecbgikgjiidgainklofbeemkhmj?hl=ko)

---

## 연구 한계

1. **Data Safety 자기신고 방식**: 개발자 신고에 의존하므로 공유 범위 과소 측정 가능성 존재 (수집 범위는 OS 권한 데이터로 보완)
2. **수익화 목적 분류**: Analytics·Personalization을 수익화로 분류하였으나, 반드시 수익화와 직결되지 않을 수 있어 가설 2 수치가 다소 과대 추정될 가능성 있음

**향후 연구**: 네트워크 트래픽 분석 및 동적 앱 실행 환경 결합으로 실제 제3자 전송 여부 직접 검증

---

## 핵심 인사이트

> 무료 앱은 유료 앱보다 더 많은 데이터를 수집하고, 그 데이터의 대부분을 수익화 목적으로 제3자에게 공유하며,  
> 이러한 수집·공유 구조는 머신러닝 모델로 사전에 탐지 가능하다.  
> **즉, 무료 앱의 실질적 가격은 '0원'이 아니라 '사용자의 데이터'이다.**

---

## 참고 문헌

1. Statista. "Revenue of mobile apps worldwide from 2014 to 2024." 2024.
2. S. Zuboff. *The Age of Surveillance Capitalism.* PublicAffairs, 2019.
3. 개인정보보호위원회. 「2024 개인정보 이슈 심층 분석 보고서」. 2024.
4. Pew Research Center. "How Americans View Data Privacy." October 2023.
5. ICO and Ofcom. "Adtech Market Research Report." March 2019.
6. Google Play Help. "Provide information for Google Play's Data safety section." 2026.
7. A. Alkinoon et al. "Mobile App Privacy Disclosures on Google Play in the Post-GDPR Context." *Information*, Vol.17, No.4, 2026.
8. SafetyDetectives. "Free Apps, Hidden Costs: Unmasking the Privacy Risks Behind App Permissions." 2024.
