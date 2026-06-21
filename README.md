# GPT 2.0 Assignment — TinyGPT on *Pride and Prejudice*

## 1. 프로젝트 개요

이 프로젝트는 수업에서 제공된 Notebook 1–6의 구현 흐름을 바탕으로, **character-level decoder-only Transformer(TinyGPT)**를 구현하고 학습한 과제입니다.

기존 실습 데이터인 Tiny Shakespeare 대신 Jane Austen의 소설 **_Pride and Prejudice_**를 사용했습니다. 이 모델은 OpenAI의 공식 GPT-2 전체 모델을 재현한 것이 아니라, GPT의 핵심 구성 요소를 작은 규모로 구현한 교육용 모델입니다.

실행 코드와 전체 출력은 [`GPT_2_0_Pride_and_Prejudice.ipynb`](GPT_2_0_Pride_and_Prejudice.ipynb)에서 확인할 수 있습니다.

## 2. 데이터셋

- 작품명: *Pride and Prejudice*
- 저자: Jane Austen
- 출처: [Project Gutenberg eBook #1342](https://www.gutenberg.org/ebooks/1342)
- 형식: UTF-8 plain text
- 학습에 사용한 문자 수: **694,452**
- Vocabulary size: **86**

노트북 실행 시 Project Gutenberg의 텍스트 파일을 `input.txt`로 내려받습니다. 이후 다음 문장부터 Project Gutenberg 종료 표시 전까지를 사용했습니다.

```text
It is a truth universally acknowledged ...
```

## 3. 모델 구조

모델은 다음 구성 요소를 포함합니다.

- Character-level token embedding
- Positional embedding
- Causal masked self-attention
- Multi-head attention
- Feed-forward network
- Residual connection
- Layer normalization
- Dropout
- 4개의 Transformer block
- 다음 문자를 예측하는 language-model head

## 4. 학습 설정

| 항목 | 설정값 |
|---|---:|
| Context length (`block_size`) | 64 |
| Batch size | 64 |
| Embedding dimension | 128 |
| Attention heads | 4 |
| Transformer layers | 4 |
| Dropout | 0.1 |
| Optimizer | AdamW |
| Learning rate | 0.0003 |
| Training rounds | 100 |
| Maximum steps per round | 300 |
| Random seed | 1337 |

## 5. 학습 결과

| 지표 | 결과 |
|---|---:|
| Epoch 0 train loss | **2.6625** |
| Epoch 99 train loss | **1.1058** |
| Loss 감소량 | **1.5567** |

Train loss가 2.6625에서 1.1058로 전반적으로 감소했습니다. 이는 모델이 데이터셋의 문자 배열, 영어 단어 형태, 띄어쓰기, 구두점 및 대화문 패턴을 학습했음을 보여줍니다.

이 과제에서는 수업 노트북의 학습 방식을 유지하여 **train loss**를 결과로 제시했습니다. 별도의 validation loss는 계산하지 않았습니다.

## 6. 생성 결과

- Prompt: `Elizabeth`
- 생성 길이: 500 characters

생성 결과 일부:

```text
Elizabeth and /*.

[Illustration:

“This was now concerned as you?”

“I am much for obliged to him. His praises manners on the other,” said her sister,
“But connected with greater which had from the aracter a hundress ...
```

전체 생성 결과는 [`generated_sample.txt`](generated_sample.txt)와 노트북의 마지막 출력에서 확인할 수 있습니다.

생성 문장은 영어 단어와 대화문·구두점 패턴을 보이지만, 긴 문맥의 문법적·의미적 일관성은 완전하지 않습니다. 또한 `[Illustration]`, 저작권 관련 문구가 나타났는데, 이는 원본 Project Gutenberg 텍스트에 포함된 편집·삽화 표기를 모델이 함께 학습했기 때문입니다.

## 7. 실행 방법

Google Colab에서 다음 순서로 실행합니다.

1. `GPT_2_0_Pride_and_Prejudice.ipynb`를 Colab에서 엽니다.
2. `런타임 → 런타임 유형 변경 → GPU`를 선택합니다.
3. `런타임 → 모두 실행`을 선택합니다.
4. 학습이 끝나면 epoch별 train loss와 생성 텍스트를 확인합니다.

노트북 안의 다음 코드가 데이터셋을 자동으로 내려받습니다.

```python
!wget -q -O input.txt https://www.gutenberg.org/cache/epub/1342/pg1342.txt
```

## 8. 한계

- 문자 단위 모델이므로 BPE나 subword tokenizer보다 비효율적입니다.
- 한 권의 소설만 학습했기 때문에 일반적인 영어 생성 능력이 제한적입니다.
- Context length가 64자로 짧아 긴 문맥을 안정적으로 유지하기 어렵습니다.
- Train loss만 측정했으므로 학습하지 않은 데이터에 대한 일반화 성능은 별도로 검증하지 않았습니다.
- 생성 결과는 그럴듯한 문장 형식을 보일 수 있지만 사실성이나 의미적 일관성을 보장하지 않습니다.

## 9. 참고 및 출처

- 수업에서 제공된 Notebook 1–6
- Andrej Karpathy, [Neural Networks: Zero to Hero](https://github.com/karpathy/nn-zero-to-hero)
- Jane Austen, [*Pride and Prejudice*, Project Gutenberg eBook #1342](https://www.gutenberg.org/ebooks/1342)

## 10. 본 과제에서 변경한 내용

- Tiny Shakespeare를 *Pride and Prejudice*로 교체
- Project Gutenberg 본문 구간 추출
- 데이터셋에 맞게 character vocabulary 재생성
- `Elizabeth`를 prompt로 사용해 500자 생성
- 100회 학습 결과 및 생성 결과 기록
- 데이터셋, 모델 구조, 학습 설정, 결과와 한계 설명 추가
