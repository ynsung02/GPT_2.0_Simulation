# GPT 2.0 Assignment — TinyGPT on *Pride and Prejudice*

## 1. 개요

이 프로젝트는 수업에서 제공된 Notebook 1–6의 구현 흐름을 바탕으로, **TinyGPT**를 구현하고 학습한 과제입니다.

기존 실습 데이터인 Tiny Shakespeare 대신 Jane Austen의 소설 **_Pride and Prejudice_**를 사용했습니다. 

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

```python
start_phrase = "It is a truth universally acknowledged"
end_marker = "*** END OF THE PROJECT GUTENBERG EBOOK PRIDE AND PREJUDICE ***"
text = raw_text[start_idx:end_idx].strip()
```

소설 본문 안에 포함된 `[Illustration]` 등의 편집 표시는 별도로 제거하지 않았습니다.

---


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

모형의 입력은 연속된 64개 문자이고, 각 위치에서 바로 다음 문자를 예측하도록 cross-entropy loss로 학습합니다.

모형 동작 확인 결과는 다음과 같습니다.

```text
logits.shape: torch.Size([64, 64, 86])
```

이는 한 batch에 64개 문장 조각이 들어가고, 각 조각의 64개 위치마다 86개 문자에 대한 예측값을 출력한다는 의미입니다.

---

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
| Loss function | Cross-entropy loss |

## 5. 학습 결과

| 지표 | 결과 |
|---|---:|
| Epoch 0 train loss | **2.6625** |
| Epoch 99 train loss | **1.1061** |
| Loss 감소량 | **1.5564** |

Train loss가 2.6625에서 1.1061로 전반적으로 감소했습니다. 

```text
epoch  0 | train loss 2.6625
epoch  1 | train loss 2.2340
epoch  2 | train loss 1.9666
...
epoch 97 | train loss 1.1109
epoch 98 | train loss 1.1082
epoch 99 | train loss 1.1061
```

이 결과는 모형이 학습 데이터에서 다음 문자를 예측하는 능력을 점차 개선했음을 보여줍니다.

Notebook 06의 학습 방식을 유지하여 **train loss**를 결과로 제시했습니다. 별도의 validation loss 평가는 수행하지 않았습니다.

## 6. 생성 결과

- Prompt: `Elizabeth`
- 생성 길이: 500 characters

생성 결과 일부:

```text
Elizabeth herself out Jane himself to be sure. The warmth of Elizabeth

[Illustration]

A restraint benefit of the all. The hint ill are to what had longerness in
summer; for, that they were knowledged to conciliation of whom he
can communicate is to Mr. Bingley’s being just can boaster.”

“I had it because he spairs?”

“In what a man treat proper pleasure still more calculativing was slightly awakened very particularly affordship’s consent condescers,” said he, and be perfectly with
the entrance of grap
```

생성 결과에는 `Elizabeth`, `Jane`, `Mr. Bingley`와 같은 인물명, 영어 단어 형태, 따옴표, 대화문 및 소설식 문장 패턴이 나타납니다. 이는 모형이 원문의 문자 배열과 지역적인 문체 패턴을 학습했다는 것을 보여줍니다.

반면 `longerness`, `spairs`, `calculativing`처럼 실제 영어에서 부자연스러운 단어와 의미가 매끄럽게 이어지지 않는 문장도 생성되었습니다. 이는 문자 단위의 작은 모형, 64자의 짧은 context length, 단일 소설 학습이라는 조건에서 예상할 수 있는 한계입니다.

출력에 `[Illustration]`이 나타난 것은 Project Gutenberg 원문 내부의 삽화 표기를 모형이 함께 학습했기 때문입니다. 마지막 문장이 `grap`에서 끝난 것은 오류가 아니라, 생성 길이를 500자로 제한했기 때문입니다.

---

## 7. 한계

- 문자 단위 tokenization을 사용하였기에 BPE나 subword tokenizer보다 비효율적임.
- 한 권의 소설만 학습했기 때문에 일반적인 영어 생성 능력이 제한적임.
- Context length가 64자로 짧아 긴 문맥을 안정적으로 유지하기 어려움.
- Train loss만 측정했으므로 학습하지 않은 데이터에 대한 일반화 성능은 별도로 검증하지 않음.
- 생성 결과는 문장 형태를 흉내낼 수 있지만 문법적 정확성, 일관성 또는 의미적 일관성을 보장하지 않음을 확인함.

## 8. 참고 및 출처

- Learnus 수업 자료로 제공된 Notebook 1–6
- Andrej Karpathy, [Neural Networks: Zero to Hero](https://github.com/karpathy/nn-zero-to-hero)
- Jane Austen, [*Pride and Prejudice*, Project Gutenberg eBook #1342](https://www.gutenberg.org/ebooks/1342)

## 9. 변경한 내용

- Tiny Shakespeare를 *Pride and Prejudice*로 교체
- Project Gutenberg 텍스트 자동 다운로드 코드 추가
- Gutenberg 안내문을 제외하고 소설 본문 구간 추출
- 데이터셋에 맞게 character vocabulary 재생성
- 생성 시작 문구를 'ROMEO:'에서 `Elizabeth`로 변경, prompt로 사용해 500자 생성
- 100회 학습 결과 및 500자 생성 결과 기록
