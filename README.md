# Privacy[baseline_mapping_for_openended.md](https://github.com/user-attachments/files/26148859/baseline_mapping_for_openended.md)
# Draft Paper → Open-ended 논문 기준 매핑 가이드

## 1. Draft Paper 주요 발견 (가져올 내용)

오픈엔드 응답 기반 논문에서 **비교 기준(baseline)**으로 활용할 핵심 발견:

### 1.1 프라이버시 우려 패턴 (Figure 1)
- **US 정부** < 미국 기업 < 외국 정부 ≈ 외국 기업 순으로 우려 수준 증가
- 미국인 38%만 US 정부에 우려, 56%는 미국 기업에 우려, 65–67%는 외국 행위자에 우려

### 1.2 인구통계학적 예측요인 (Table 2)
| 변수 | Draft Paper 발견 | 오픈엔드 논문에서 검증할 점 |
|------|------------------|---------------------------|
| **Cohort (Millennial)** | 젊은층 → 더 적은 행위자에 대해 우려 (US 정부 제외 시 더 두드러짐) | 오픈엔드에서 젊은층이 어떤 주제/어휘를 더 많이 사용하는가? |
| **Education** | 고학력 → 모든 행위자에 대한 우려 증가 | 고학력자의 오픈엔드 응답이 어떤 프라이버시 이슈를 더 구체적으로 언급하는가? |
| **Income** | 고소득 → US 정부에 대한 신뢰 증가 | 소득과 오픈엔드 토픽 분포 관계 |
| **Gender (Woman)** | 여성 → 일부 카테고리에서 우려 증가 (worried about all vs foreign actors) | 여성이 오픈엔드에서 어떤 토픽을 더 많이 언급하는가? |
| **Latinx** | Latinx → 특정 패턴에서 차이 (not worried US gov vs all) | 인종/히스패닉과 오픈엔드 표현 차이 |
| **Terms of Service (지식)** | 지식 높음 → 우려 수준 상승, 그러나 동시에 데이터 공유 의향도 높음 | 지식과 오픈엔드 응답의 구체성/복잡성 관계 |

### 1.3 프라이버시 역설 (Table 3)
- **선호(willingness)**와 **가치평가(valuation)** 분리: 선호는 행동과 무관, 가치평가는 우려와 양의 상관
- 오픈엔드에서: “말로 표현하는 우려”와 “실제 행동/가치평가”가 어떻게 다른 맥락에서 나타나는지 탐색

---

## 2. 변수 코딩 기준 매핑

### 2.1 Draft Paper vs Notebook/R 코드 변수 대응

| Draft Paper 변수 | Notebook/ALP 원변수 | R (STM) 변수 | 코딩 (PrivacyDataAnalysis2 기준) |
|------------------|---------------------|--------------|----------------------------------|
| Woman | gender | gender | **1: Male, 2: Female** |
| Born in US | borninus | borninus | **1: Yes, 2: No** |
| Latinx | ethnicity + hispaniclatino | hispanic | ethnicity: 1=White,2=Black,3=NatAm,4=API,5=Other / hispaniclatino: 1=Yes,2=No |
| Age/Cohort (Millennial, Senior) | birthyear, calcage | age (=calcage) | calcage 연속변수 / birthyear로 cohort 생성 가능 |
| Education | highesteducation | educ | **1–16** (9=HS, 10=Some college, 11–13=Assoc/Bach, 14–16=Grad) |
| Income | familyincome | (현재 R에 없음) | $35k–$75k, ≥$75k 등 |
| Employed | doyouwork | (현재 R에 없음) | |
| Recruitment type | recruitment_type | r_type | |
| Living situation | currentlivingsituation | living_situ | |

### 2.2 Education Binning (Draft Paper와 비교용)

Draft Paper: HS 기준, Some college, College degree, Graduate degree

**PrivacyDataAnalysis2 ed_bins:**
```
highesteducation 1–8  → ed_bins 1 (less than HS)
highesteducation 9    → ed_bins 2 (HS grad)
highesteducation 10   → ed_bins 3 (some college)
highesteducation 11–13→ ed_bins 4 (undergrad degree)
highesteducation 14–16→ ed_bins 5 (advanced degree)
```

**R에서 사용 중:** `educ` = highesteducation **1–16 연속** (binning 없음)

### 2.3 Age/Cohort (Draft Paper)

- Millennial: birthyear 기준 (보통 1981–1996)
- Senior: 65세 이상 등
- R: `age` = calcage **연속변수** → cohort 더미 생성 가능

---

## 3. R 코드에서 기준(reference) 설정

### 3.1 현재 stm_privacy.R 변수 처리

```r
# stm_privacy.R 86행
data[vars] <- lapply(data[vars], function(x) as.numeric(as.factor(x)))
```

`as.numeric(as.factor(x))`는 **factor 수준의 알파벳 순서**로 숫자를 부여합니다.  
→ **reference = 숫자 1에 해당하는 수준** (STM의 estimateEffect에서)

### 3.2 변수별 Reference 확인

| R 변수 | 원본 코딩 | as.factor 후 1번 = Reference | Draft Paper와의 정렬 |
|--------|-----------|-----------------------------|---------------------|
| gender | 1=Male, 2=Female | 1=Male (숫자 작음) | ✅ Male 기준 (Woman 효과 해석) |
| borninus | 1=Yes, 2=No | 1=Yes (Born in US) | ✅ Born in US 기준 |
| hispanic | hispaniclatino 1=Yes, 2=No | 1=Yes, 2=No | ⚠️ 확인 필요: 1=Hispanic? |
| educ | 1–16 | 1=최저 학력 | 연속변수로 해석 (1단위 증가 시 효과) |
| r_type | recruitment_type | 알파벳/숫자 순 | |
| living_situ | currentlivingsituation | 알파벳/숫자 순 | |

### 3.3 R에서 Reference를 명시적으로 맞추는 방법

Draft Paper와 동일한 reference를 쓰려면, **factor level 순서**를 조정해야 합니다:

```r
# 예: gender - Male을 reference로 (1번)
data$gender <- factor(data$gender, levels = c(1, 2), labels = c("Male", "Female"))

# 예: borninus - Born in US를 reference로
data$borninus <- factor(data$borninus, levels = c(1, 2), labels = c("BornInUS", "NotBornInUS"))

# 예: hispanic - Non-Hispanic을 reference로 (Draft는 White/Latinx)
# hispaniclatino: 1=Yes(Hispanic), 2=No → 2를 reference로
data$hispanic <- factor(data$hispaniclatino, levels = c(2, 1), labels = c("NonHispanic", "Hispanic"))

# educ: 연속변수 유지 시 reference 개념 없음 (선형 효과)
# binning 시: HS(9)를 reference로
```

STM의 `estimateEffect`는 **수치형 변수**를 그대로 사용하므로,  
**factor로 바꾸지 않고** 현재처럼 numeric을 쓰면:
- `gender`: 1=Male, 2=Female → **Male이 baseline** (2 vs 1의 차이 = Female 효과)
- `borninus`: 1=Yes, 2=No → **Born in US가 baseline**

### 3.4 STM estimateEffect 해석

```r
# STM_gender.r 49행
estimateEffect(1:8 ~ gender + age + hispanic + educ + r_type + living_situ + borninus, ...)
```

- **gender**: cov.value1=1 (Male), cov.value2=2 (Female) → **Male - Female 차이** (음수면 Female이 해당 토픽 더 높음)
- **borninus**: cov.value1=1 (Yes), cov.value2=2 (No) → **Born in US - Not Born in US**
- **age, educ**: 연속변수 → 1단위 증가 시 토픽 비율 변화

---

## 4. 오픈엔드 논문에서 강조할 비교 포인트

1. **Closed-ended (Draft)**: 4점 척도, 기관별 우려 수준 → “얼마나 걱정하는가”
2. **Open-ended (당신 논문)**: 자유 응답 → “무엇을, 어떤 맥락으로 걱정하는가”

비교 질문 예:
- Draft: 젊은층이 덜 걱정한다 → 오픈엔드: 젊은층이 **어떤 종류의** 프라이버시 이슈를 더/덜 언급하는가?
- Draft: 여성이 더 걱정한다 → 오픈엔드: 여성이 **어떤 주제/어휘**를 더 많이 사용하는가?
- Draft: 고학력이 더 걱정한다 → 오픈엔드: 고학력자의 응답이 **어떤 수준의 구체성/추상성**을 보이는가?

---

## 5. 체크리스트: R 코드 기준 정리

- [ ] `gender`: 1=Male, 2=Female 확인 (plot에서 cov.value1=1, cov.value2=2 사용 중)
- [ ] `borninus`: 1=Yes(Born in US), 2=No 확인
- [ ] `hispanic`: hispaniclatino 1=Yes, 2=No 매핑 확인 (ethnicity와 별도)
- [ ] `educ`: 1–16 연속 사용 시, Draft의 education binning과 결과 해석 시 대응
- [ ] `age`: 연속 사용 시, Millennial/Senior 더미가 필요하면 birthyear로 생성
- [ ] `r_type`, `living_situ`: 수준별 빈도 확인 후 reference 해석

---

## 6. 참고: stm_data.xlsx → stm_prep.RData 흐름

- `stm_privacy.R`: stm_data.xlsx 로드 → corpus 생성 → meta2_nltk, meta2_sklearn
- `STM_gender.r`: stm_prep.RData 로드 (이미 전처리된 meta 사용)
- **stm_data.xlsx** 생성 시점에서 gender, borninus, hispanic, educ 등이 **PrivacyDataAnalysis2 코딩**과 일치하는지 확인 필요
