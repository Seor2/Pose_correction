# 🏋️ Home Training Pose Correction AI

유튜브 영상 링크만으로 **어떤 운동이든** 기준 자세를 자동 추출하고, 사용자 자세와 비교해 한국어 피드백을 제공하는 자세 교정 AI입니다.

> 개인 프로젝트 | 세종대학교 딥러닝시스템 수업 | 2024

---

## 📌 Problem

혼자 홈트레이닝을 할 때 내 자세가 맞는지 알기 어렵고, 잘못된 자세는 부상으로 이어집니다.  
트레이너 없이도 자세를 교정해줄 수 있는 AI 시스템을 만들고자 했습니다.

---

## 🔧 Pipeline

### 학습 파이프라인
```
COCO pretrained KeypointRCNN (ResNet-50) 로드
→ Roboflow Exercise Pose 데이터셋에 Pseudo-labeling 적용
   (keypoint annotation 없는 이미지에 자동 생성 후 COCO JSON 형식으로 저장)
→ Keypoint Head 레이어만 Fine-tuning
→ PCK@0.2로 성능 평가
```

### 추론 파이프라인
```
유튜브 URL 입력 (yt-dlp 다운로드)
→ 0.5초 단위 프레임 추출 (OpenCV VideoCapture)
→ 프레임별 관절 각도 벡터 추출 (11개 각도)
→ KMeans 클러스터링 (k=2) → 동작 구간 분리
→ 고점 프레임 선택 (무릎 각도 최솟값 기준)
→ 동작별 기준 각도 확정
→ 사용자 영상과 관절 각도 비교 (차이 > 15° → 문제 부위 판정)
→ Groq LLaMA3 API로 한국어 자연어 피드백 생성
→ 관절선 시각화 + 피드백 텍스트 출력
```

---

## 📊 Results

| Metric | Score |
|--------|-------|
| PCK@0.2 | **95.8%** |
| Fine-tuning Loss (epoch 1) | 1.5374 |
| Fine-tuning Loss (epoch 10) | 1.0064 (수렴) |

- 지원 운동: Squat, Pushup, Pullup, Lunge 등 (유튜브 링크 기반으로 확장 가능)
- 문제 부위는 **빨간 관절선**으로 시각화, 정상 부위는 초록색으로 표시
- 동작별로 기준 자세 이미지와 사용자 자세 이미지를 나란히 출력

---

## 🛠️ Tech Stack

- **Model**: KeypointRCNN (ResNet-50 backbone, FPN neck)
- **Dataset**: COCO 2017 Keypoint (pretrain) + Roboflow Exercise Pose 1,402장 (fine-tune)
- **Pseudo-labeling**: COCO pretrained 모델로 keypoint 자동 생성
- **Clustering**: KMeans (k=2) on joint angle vectors
- **LLM Feedback**: Groq API (LLaMA3, 한국어)
- **Libraries**: PyTorch, OpenCV, yt-dlp, scikit-learn

---

## 📁 Structure

```
├── pose_correction.ipynb   # 전체 파이프라인 (학습 + 추론 통합, Kaggle 환경)
└── README.md
```

> **실행 환경**: Kaggle Notebooks (GPU T4, Internet ON 필수)  
> Kaggle Secrets에 `ROBOFLOW_API_KEY`, `GROQ_API_KEY` 등록 필요
