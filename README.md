# datamining_teamproject
# 영화 박스오피스 데이터 수집 & 전처리 파이프라인

## 📁 파일 구조

```
프로젝트/
├── 01_collect_kobis.py        # KOBIS 박스오피스 + Top1 상세 수집
├── 02_collect_kma.py          # KMA 날씨 데이터 수집
├── 03_merge_preprocess.py     # 3개 데이터 병합 + 전처리
├── data/
│   ├── kobis_daily.csv              # ← 01번 실행 결과
│   ├── kobis_movie_detail.csv       # ← 01번 실행 결과
│   ├── kma_weather.csv              # ← 02번 실행 결과
│   ├── movie_discount_days_full.csv # ← 이미 만들어둔 할인일 파일 (여기에 복사)
│   └── final_dataset.csv            # ← 03번 실행 최종 결과 ⭐
```

## 🔑 1단계: API 키 발급

### KOBIS API 키
1. https://www.kobis.or.kr/kobisopenapi 접속
2. 회원가입 → 로그인 → 마이페이지 → API 키 발급
3. 발급된 키를 `01_collect_kobis.py`의 `KOBIS_KEY`에 입력

### KMA(공공데이터포털) API 키
1. https://www.data.go.kr 접속
2. "기상청_지상(종관, ASOS) 일자료 조회서비스" 검색
3. 활용신청 → 승인(보통 즉시) → 마이페이지에서 일반 인증키(Decoding) 확인
4. 키를 `02_collect_kma.py`의 `KMA_KEY`에 입력

## 🚀 2단계: 실행 순서

```bash
# data 폴더 생성하고, 미리 만든 할인일 CSV를 복사
mkdir -p data
cp movie_discount_days_full.csv data/

# 1. KOBIS 데이터 수집 (시간 오래 걸림 - 약 1,100일 × API 호출)
python 01_collect_kobis.py

# 2. KMA 날씨 데이터 수집 (월 단위 청크로 빠름)
python 02_collect_kma.py

# 3. 병합 및 전처리
python 03_merge_preprocess.py
```

## ⚠️ 주의사항

1. **KOBIS 일별 박스오피스는 약 1,100번의 API 호출이 필요**합니다.
   - `time.sleep(0.1)` 적용으로 약 2~3분 소요
   - 일일 호출 제한이 있을 수 있으니 필요시 분할 실행
   - 중간에 끊기면 이미 받은 부분은 무시하고 재시작 가능

2. **API 키는 절대 GitHub 등에 올리지 마세요**
   - 환경변수로 분리하는 것을 권장: `os.getenv("KOBIS_KEY")`

3. **공휴일 리스트는 검증 필요**
   - `03_merge_preprocess.py`의 `KOREAN_HOLIDAYS`는 일반적인 일정 기준
   - 2026년 부처님오신날 등 정확한 일자는 추후 검증 필요

## 📊 최종 산출물: `final_dataset.csv`

총 52개 변수가 포함된 일별 데이터셋:

| 카테고리 | 변수 예시 |
|----------|----------|
| 종속변수 | top1_audience, top10_audience |
| 날씨 | temp_avg, temp_max, temp_min, rain, wind_avg, heatwave, coldwave |
| 가격 | is_discount_day, discount_policy |
| 시간 통제 | weekday_num, is_weekend, is_holiday, is_vacation, month |
| 시장 매력도 | top10_share_sum, has_blockbuster, has_mega_hit, hhi_index, top1_dominance, num_new_in_top10, top1_days_since_release |
| 콘텐츠 특성 | top1_major_distributor, top1_runtime, top1_is_korean, genre_*, grade_* (더미) |
