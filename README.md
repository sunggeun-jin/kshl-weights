# KSHL 가중치

한국수어 양방향 번역 앱 **[KSHL](https://github.com/sunggeun-jin/kshl)** 의 수어 인식 모델 가중치를 담은 저장소입니다.

기반 모델 없이 처음부터 전체 학습시킨 키포인트 시퀀스 분류기이며, 학습·추론 코드는 본 저장소가 아니라 [sunggeun-jin/kshl](https://github.com/sunggeun-jin/kshl) 에 있습니다.

## 파일 구성

| 경로 | 크기 | 설명 |
| --- | --- | --- |
| `kshl-weights/ksl_mp_model/best.safetensors` | 16,968,160 B (약 16.2 MiB) | 전체 가중치 |
| `kshl-weights/ksl_mp_model/config.json` | 332 B | 모델 구조와 학습 설정 |
| `kshl-weights/CHECKSUMS.txt` | — | `best.safetensors` 의 SHA-256 |

승인 절차 없이 누구나 내려받을 수 있습니다.

## 모델 사양

`config.json` 에 기록된 값입니다.

| 항목 | 값 |
| --- | --- |
| 구조 | Transformer |
| 임베딩 차원 (`dim`) | 256 |
| 층 수 (`depth`) | 4 |
| 헤드 수 (`heads`) | 4 |
| 출력 클래스 (`n_class`) | 3,000 |
| 학습 epoch | 24 |

## 입력 형식

`55 키포인트 × 3 채널 × 64 프레임` 텐서를 받습니다.

- **키포인트 55점** — 상반신 13 + 왼손 21 + 오른손 21. MediaPipe Holistic 과 OpenPose BODY_25 가 실제로 측정하는 것의 교집합으로 정의한 표준 키포인트입니다.
- **정규화** — 목을 원점으로 옮기고 어깨 너비를 1로 맞춰 인물 크기와 화면 위치의 영향을 제거합니다.
- **길이** — 클립 길이가 제각각이므로 64 프레임으로 리샘플합니다.

좌표만 사용하며 원본 영상과 얼굴 이미지는 모델에 넣지 않습니다.

## 학습

Apple Silicon 에서 [MLX](https://github.com/ml-explore/mlx) 로 학습했습니다. AI-Hub 한국수어 영상 데이터셋을 MediaPipe Holistic 으로 재추출해 3,000 단어의 키포인트 시퀀스로 캐시한 것을 사용했습니다.

증강 설정(`config.json` 의 `aug`)은 손 소실 0.32, 좌우 반전 0.5, 회전 13°, 크기 0.2, 이동 0.1, 잡음 0.01, 프레임 드롭 0.05 입니다.

## 성능

성격이 다른 두 숫자가 있으니 함께 보시기 바랍니다.

| 기준 | top-1 | top-5 |
| --- | --- | --- |
| `config.json` 에 기록된 학습 시 평가값 | 0.9578 | 0.9975 |
| 화자 독립 평가 (자체 녹화 100 클립, 후보 3,000 개) | 0.610 | 0.760 |

아래 줄이 실제 사용 조건에 가까운 값입니다. 학습에 쓰지 않은 화자·장소·카메라로 찍은 클립을 3,000 개 후보 중에서 맞히게 한 결과이며, 무작위 기대치는 0.0003 입니다. 후보를 실제 녹화한 100 단어로 제한하면 top-1 은 0.770 입니다.

손이 신호의 전부입니다. 손 키포인트를 제거하면 top-1 이 0.000 으로 떨어지므로, 손이 잘 보이지 않는 영상에서는 결과를 신뢰하기 어렵습니다.

## 무결성 확인

```bash
cd kshl-weights && shasum -a 256 -c CHECKSUMS.txt
```

## 사용

[sunggeun-jin/kshl](https://github.com/sunggeun-jin/kshl) 의 서버 코드가 이 가중치를 불러 씁니다. 평가를 재현하려면:

```bash
python eval_own.py --model ksl_mp_model
```

## 라이선스

[Apache License 2.0](LICENSE). Copyright 2026 sunggeun-jin.

키포인트 추출에 사용한 MediaPipe 는 Apache License 2.0 이며, 학습 데이터의 출처는 AI-Hub 한국수어 영상 데이터셋입니다.
