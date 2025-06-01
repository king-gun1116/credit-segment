< 신용카드 세그먼트 분류 프로젝트 >

1.이진 분류 기반 모델링 (Baseline Version)

- 분석 목표 : Segment 컬럼을 기반으로 신용카드 고객을 다음과 같이 이진 분류
  • E (고위험 고객)
  • Non-E (A/B/C/D 그룹)

- 주요 변수 (금융 정보 & 연체/잔액 관련)
  • 이용 한도, 이자율, 리볼빙 비율 등
  • 잔액 정보 (일시불, 할부, 카드론 등 평잔/변동률)
  • 연체일수/금액, 연체원금, 회차
  • 선결제/정기결제(RP) 정보 등

 - 모델 구성
  • 1단계 이진 분류 (E vs Non-E)
  • 모델: RandomForestClassifier(class_weight='balanced')
  • 성능: Accuracy = 0.87, F1-score (E 클래스) = 0.92
  • 2단계 다중 분류 (A/B/C/D → AB/C/D 재구성)
  • SMOTE 적용 / 클래스 가중치 조정
  • 사용 모델: XGBoost, CatBoost, LightGBM
  • 최종 성능: Accuracy ≈ 0.80, F1-score (weighted avg) ≈ 0.79

2. 군집화 결합 기반 모델링 (Cluster Version)

 - 분석 목표 : 고객의 금융 정보 및 연체/잔액 패턴을 기반으로 자연스럽게 분류되는 군집을 사전 학습하고, 이를 분류 모델의 입력 피처로 활용하여 성능을 고도화
 - 주요 변수
   • 약 140여 개의 파생 금융 변수 (표준화 및 결측치 처리 완료)
   • 리볼빙/한도 사용률, 연체 정보, 결제 패턴 등

 - 군집화 적용 과정
  1) KMeans(n_clusters=3) 군집화 적용
    • Segment E는 제외한 Non-E 고객 대상
    • 각 고객에 대해 Cluster 레이블(0/1/2) 부여
  2) Cluster 라벨을 신규 변수(Cluster_KMeans)로 추가
    • SMOTE 적용 후 3-class 분류 모델 (AB / C / D)에 추가 입력
    • 실루엣 점수 기준 최적의 클러스터 수 도출 및 변수 중요도 분석 수행
  3) 모델 학습 및 평가
    • SMOTE 기반 모델에 Cluster_KMeans를 추가한 후 성능 비교
