# CSS
CSS : Cluster-based Comparison System for Color Scheme Similarity

<br>

## 📄 개요 (Overview)

이미지 간 유사도를 비교하는 일반적인 방법 중 하나는 분류(Classification) 모델을 통해 각 이미지의 특징(feature)을 추출하고, 이를 벡터 형태로 표현한 뒤 유사도를 계산하는 방식입니다. 이 방법은 정량적인 비교에 유리하다는 장점이 있지만, 스타일, 분위기, 느낌처럼 주관적이고 애매한 개념에 기반한 비교에는 한계가 존재합니다.

특히 명확한 라벨이 존재하지 않거나, 기준이 추상적이어서 일관된 학습이 어렵고, 대량의 데이터를 확보하기도 쉽지 않은 경우가 많습니다. 이처럼 분류 기준이 정의되기 어려운 상황에서는 기존 방식으로 유사도를 측정하기가 까다롭습니다.

본 프로젝트에서는 이러한 한계를 극복하기 위해, 색상 팔레트와 클러스터링 기반의 접근을 제안합니다. 별도의 모델 학습 없이도 색상 분포와 구성만으로 유사도를 정량화할 수 있도록 설계되어 있으며, 특히 라벨링이 어렵고 주관성이 큰 영역에서도 직관적인 비교가 가능한 시스템을 제공합니다.

➡️ 본 프로젝트에서는 이러한 방식의 실험 대상으로 **인테리어 이미지**를 선정하여 진행하였습니다.

<br>

## 📚 참고 연구 (References)

두 논문은 각각 색상 팔레트 추출 방식과 팔레트 간 유사도 계산 방식을 제시하고 있으며, 해당 내용을 기반으로 본 시스템의 주요 알고리즘이 구성되었습니다.

*Color Scheme Extraction Based on Image Segmentation and Saliency Map / [링크](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE10536766)*  
- Deep ConvNet, PSPNet 등 CNN 기반 분할 모델을 사용하여 이미지에서 전경과 배경을 분리
- Saliency Map을 활용해 중요 영역을 판단, K-means로 전경/배경의 색상 팔레트를 추출

 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbjkV4O%2FbtsLOtHMGXI%2FajjoeEKhTzIxwMcuLWrLSk%2Fimg.png)
  
*Color Palettes Comparison Using Cluster-based Hausdorff Distance / [링크](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE10618312)*
- 기존 색상 팔레트 유사도 비교 방식의 한계를 보완하고자 클러스터 기반 Hausdorff Distance(CHD) 제안
- 색상 팔레트를 클러스터링한 후 클러스터 간의 거리 기반으로 구조적인 유사도 측정

 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F22jlU%2FbtsLMHuhOmK%2FO53vYo38R8YzCrtjnwJNK0%2Fimg.png)

<br>

## 🙏 감사의 말 (Acknowledgements)

본 프로젝트는 위 연구들을 참고하여 발전시킨 것으로, 해당 연구들의 주요 저자이자 조언을 주신 **김수지님**께 깊은 감사를 드립니다.

<br>

## 📐 유사도 계산 방식 (Similarity Calculation Steps)

본 시스템은 이미지의 색상 유사도를 보다 정밀하게 측정하기 위해 전경, 배경, 그리고 개별 개체들을 분리하고, 각 영역에 대해 다른 방식의 가중 유사도 계산을 적용합니다.

### 1️⃣ 전경 · 배경 유사도

- 전경과 배경을 분리한 뒤, 각 영역의 색상 정보를 K-means 클러스터링을 통해 추출합니다.
- 클러스터링된 색상에 대해 **각 색상이 이미지에서 차지하는 면적 비율을 가중치로 적용**하여 유사도를 계산합니다.
- 색상 간 비교는 Cluster-based Hausdorff Distance(CHD)를 기반으로 수행됩니다.

**💡 개선 포인트**  
→ 단순히 색상만 비교하는 것이 아니라, **이미지에서의 비중(면적)** 을 반영하여 보다 정확한 비교가 가능해졌습니다.

 🖼️  전배경 유사도 비교 예시 (좌측(선택 이미지), 우측(유사 이미지))
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FLb187%2FbtsL4Pw1q8h%2Fxgx6UUlPkQmnvBKEtp2IDK%2Fimg.png)

### 2️⃣ 개체 유사도 (Object-Level Matching)

#### ● 왜 필요한가?
- 전경에는 조명, 소품, 가구 등 다양한 개체가 존재합니다.
- 이를 전경 전체로 묶어 처리하면, 중요한 개체의 색상이 무시될 수 있습니다.

#### ● 어떻게 개선했나?
- Masked-RCNN을 이용해 이미지 속 개체들을 분리합니다.
- **면적 기준으로 주요 개체 최대 5개를 선정**하여, 너무 작거나 중요도가 낮은 개체는 제외합니다.
- 각 개체의 색상 팔레트를 추출하고, 개체 간 유사도 비교를 수행합니다.
- 개체 수가 다를 경우 **Hungarian Algorithm**을 통해 최적의 매칭을 찾습니다.

**💡 개선 포인트**  
→ 전경 내 개체별 색상을 분리하여 반영함으로써, **전체 분위기를 해치지 않으면서도 세부 유사도 반영**

🖼️ 개체별 색상 팔레트 추출 예시
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F340O8%2FbtsL2mXWMuu%2FwEEo6vlGEo0ccxeZQ72GlK%2Fimg.png)

### 3️⃣ 최종 유사도 산출  
각 이미지의 전경 / 개체 / 배경에서 계산된 유사도는  
해당 영역이 이미지 전체에서 **차지하는 면적 비율**에 따라 최종 유사도 점수에 반영됩니다.
이 방식을 통해 이미지 내 색상의 **비율, 공간적 의미, 객체 단위의 분포까지 종합적으로 고려한 유사도 측정이 가능**합니다.

🖼️ 데이터 전처리 예시 (전경, 배경, 개체별 색상 비율)
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FczuM0I%2FbtsL4NTxrZD%2FbNyQw0tklwnkB8DxBOPGy0%2Fimg.png)
🖼️ 유사도 계산 결과 예시
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSm4K5%2FbtsL3SBuf1W%2FnFWYwlEK3wqbqzD8thWxU1%2Fimg.png)

<br>

## 🧑‍💻 실행 방법 (How to Run)
본 프로젝트는 색상별 가중치가 저장된 **전처리 데이터(csv)** 를 기반으로 실행됩니다.  
전처리된 csv 파일과 이미지 데이터는 이미 포함되어 있습니다.

### ▶️ 실행 순서
1. **`images/`** 폴더에서 비교하고 싶은 **이미지 번호(ID)**를 확인합니다.  
2. **`CSS.ipynb`** 실행 후 이미지 번호를 입력하면 'images/'폴더 내 유사 이미지가 출력됩니다.

### 📘 추가 사항 
**`CSS_process.ipynb`** 는 아래와 같은 분석 과정을 포함하고 있으니,  
유사도 계산의 내부 과정을 자세히 확인하고 싶은 경우 활용하시면 됩니다:
- 전경 / 배경 색상 팔레트 확인  
- 개체별 색상 팔레트 확인  
- 전경 / 배경 / 개체 각 기준별 유사 이미지 확인

<br>

> 💡 구체적인 계산 방식과 배경 개념이 궁금하다면 아래 블로그 글들을 참고해 주세요:

- [🔗 #1 기존 방식 (Feature Extraction 기반)](https://rokart.tistory.com/entry/%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%9C%A0%EC%82%AC%EB%8F%84-%EB%B9%84%EA%B5%90-1-Feature-extraction)  
  → CNN 기반 이미지 임베딩과 벡터 비교 방식 설명

- [🔗 #2 참고연구 정리 (Clustering 기반 접근)](https://rokart.tistory.com/entry/%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%9C%A0%EC%82%AC%EB%8F%84-%EB%B9%84%EA%B5%90-2-Color-Scheme)  
  → 색상 팔레트 추출 및 CHD 기반 유사도 비교 방식 정리

- [🔗 #3 본 시스템 개발 과정 정리](https://rokart.tistory.com/entry/%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%9C%A0%EC%82%AC%EB%8F%84-%EB%B9%84%EA%B5%90-3-Color-Scheme)  
  → 가중치 적용, 개체 분리, 최종 유사도 계산 방식까지 전체 구현 흐름 소개

