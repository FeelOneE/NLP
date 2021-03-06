# KoBERT-Transformers

`KoBERT` & `DistilKoBERT` on ๐ค Huggingface Transformers ๐ค

KoBERT ๋ชจ๋ธ์ [๊ณต์ ๋ ํฌ](https://github.com/SKTBrain/KoBERT)์ ๊ฒ๊ณผ ๋์ผํฉ๋๋ค. ๋ณธ ๋ ํฌ๋ **Huggingface tokenizer์ ๋ชจ๋  API๋ฅผ ์ง์**ํ๊ธฐ ์ํด์ ์ ์๋์์ต๋๋ค.

## ๐จ ์ค์! ๐จ

### ๐ TL;DR

1. `transformers` ๋ `v3.0` ์ด์์ ๋ฐ๋์ ์ค์น!
2. `tokenizer`๋ ๋ณธ ๋ ํฌ์ `kobert_transformers/tokenization_kobert.py`๋ฅผ ์ฌ์ฉ!

### 1. Tokenizer ํธํ

`Huggingface Transformers`๊ฐ `v2.9.0`๋ถํฐ tokenization ๊ด๋ จ API๊ฐ ์ผ๋ถ ๋ณ๊ฒฝ๋์์ต๋๋ค. ์ด์ ๋ง์ถฐ ๊ธฐ์กด์ `tokenization_kobert.py`๋ฅผ ์์ ๋ฒ์ ์ ๋ง๊ฒ ์์ ํ์์ต๋๋ค.

### 2. Embedding์ padding_idx ์ด์

์ด์ ๋ถํฐ `BertModel`์ `BertEmbeddings`์์ `padding_idx=0`์ผ๋ก **Hard-coding**๋์ด ์์์ต๋๋ค. (์๋ ์ฝ๋ ์ฐธ๊ณ )

```python
class BertEmbeddings(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.word_embeddings = nn.Embedding(config.vocab_size, config.hidden_size, padding_idx=0)
        self.position_embeddings = nn.Embedding(config.max_position_embeddings, config.hidden_size)
        self.token_type_embeddings = nn.Embedding(config.type_vocab_size, config.hidden_size)
```

๊ทธ๋ฌ๋ Sentencepiece์ ๊ฒฝ์ฐ ๊ธฐ๋ณธ๊ฐ์ผ๋ก `pad_token_id=1`, `unk_token_id=0`์ผ๋ก ์ค์ ์ด ๋์ด ์๊ณ  (์ด๋ KoBERT๋ ๋์ผ), ์ด๋ฅผ ๊ทธ๋๋ก ์ฌ์ฉํ๋ BertModel์ ๊ฒฝ์ฐ ์์น ์์ ๊ฒฐ๊ณผ๋ฅผ ๊ฐ์ ธ์ฌ ์ ์์ต๋๋ค.

Huggingface์์๋ ์ต๊ทผ์ ํด๋น ์ด์๋ฅผ ์ธ์งํ์ฌ ์ด๋ฅผ ์์ ํ์ฌ `v2.9.0`์ ๋ฐ์ํ์์ต๋๋ค. ([๊ด๋ จ PR #3793](https://github.com/huggingface/transformers/pull/3793)) config์ `pad_token_id=1` ์ ์ถ๊ฐ ๊ฐ๋ฅํ์ฌ ์ด๋ฅผ ํด๊ฒฐํ  ์ ์๊ฒ ํ์์ต๋๋ค.

```python
class BertEmbeddings(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.word_embeddings = nn.Embedding(config.vocab_size, config.hidden_size, padding_idx=config.pad_token_id)
        self.position_embeddings = nn.Embedding(config.max_position_embeddings, config.hidden_size)
        self.token_type_embeddings = nn.Embedding(config.type_vocab_size, config.hidden_size)
```

๊ทธ๋ฌ๋ `v.2.9.0`์์ `DistilBERT`, `ALBERT` ๋ฑ์๋ ์ด ์ด์๊ฐ ํด๊ฒฐ๋์ง ์์ ์ง์  PR์ ์ฌ๋ ค ์ฒ๋ฆฌํ์๊ณ  ([๊ด๋ จ PR #3965](https://github.com/huggingface/transformers/pull/3965)), **`v2.9.1`์ ์ต์ข์ ์ผ๋ก ๋ฐ์๋์ด ๋ฐฐํฌ๋์์ต๋๋ค.**

์๋๋ ์ด์ ๊ณผ ํ์ฌ ๋ฒ์ ์ ์ฐจ์ด์ ์ ๋ณด์ฌ์ฃผ๋ ์ฝ๋์๋๋ค.

```python
# Transformers v2.7.0
>>> from transformers import BertModel, DistilBertModel
>>> model = BertModel.from_pretrained("monologg/kobert")
>>> model.embeddings.word_embeddings
Embedding(8002, 768, padding_idx=0)
>>> model = DistilBertModel.from_pretrained("monologg/distilkobert")
>>> model.embeddings.word_embeddings
Embedding(8002, 768, padding_idx=0)


### Transformers v2.9.1
>>> from transformers import BertModel, DistilBertModel
>>> model = BertModel.from_pretrained("monologg/kobert")
>>> model.embeddings.word_embeddings
Embedding(8002, 768, padding_idx=1)
>>> model = DistilBertModel.from_pretrained("monologg/distilkobert")
>>> model.embeddings.word_embeddings
Embedding(8002, 768, padding_idx=1)
```

## KoBERT / DistilKoBERT on ๐ค Transformers ๐ค

### Dependencies

- torch>=1.1.0
- transformers>=3,<5

### How to Use

```python
>>> from transformers import BertModel, DistilBertModel
>>> bert_model = BertModel.from_pretrained('monologg/kobert')
>>> distilbert_model = DistilBertModel.from_pretrained('monologg/distilkobert')
```

**Tokenizer๋ฅผ ์ฌ์ฉํ๋ ค๋ฉด, [`kobert_transformers/tokenization_kobert.py`](https://github.com/monologg/KoBERT-Transformers/blob/master/kobert_transformers/tokenization_kobert.py) ํ์ผ์ ๋ณต์ฌํ ํ, `KoBertTokenizer`๋ฅผ ์ํฌํธํ๋ฉด ๋ฉ๋๋ค.**

- KoBERT์ DistilKoBERT ๋ชจ๋ ๋์ผํ ํ ํฌ๋์ด์ ๋ฅผ ์ฌ์ฉํฉ๋๋ค.
- **๊ธฐ์กด KoBERT์ ๊ฒฝ์ฐ Special Token์ด ์ ๋๋ก ๋ถ๋ฆฌ๋์ง ์๋ ์ด์**๊ฐ ์์ด์ ํด๋น ๋ถ๋ถ์ ์์ ํ์ฌ ๋ฐ์ํ์์ต๋๋ค. ([Issue link](https://github.com/SKTBrain/KoBERT/issues/11))

```python
>>> from tokenization_kobert import KoBertTokenizer
>>> tokenizer = KoBertTokenizer.from_pretrained('monologg/kobert') # monologg/distilkobert๋ ๋์ผ
>>> tokenizer.tokenize("[CLS] ํ๊ตญ์ด ๋ชจ๋ธ์ ๊ณต์ ํฉ๋๋ค. [SEP]")
>>> ['[CLS]', 'โํ๊ตญ', '์ด', 'โ๋ชจ๋ธ', '์', 'โ๊ณต์ ', 'ํฉ๋๋ค', '.', '[SEP]']
>>> tokenizer.convert_tokens_to_ids(['[CLS]', 'โํ๊ตญ', '์ด', 'โ๋ชจ๋ธ', '์', 'โ๊ณต์ ', 'ํฉ๋๋ค', '.', '[SEP]'])
>>> [2, 4958, 6855, 2046, 7088, 1050, 7843, 54, 3]
```

## Kobert-Transformers (Pip library)

[![PyPI](https://img.shields.io/pypi/v/kobert-transformers)](https://pypi.org/project/kobert-transformers/)
[![license](https://img.shields.io/badge/license-Apache%202.0-red)](https://github.com/monologg/DistilKoBERT/blob/master/LICENSE)
[![Downloads](https://pepy.tech/badge/kobert-transformers)](https://pepy.tech/project/kobert-transformers)

- `tokenization_kobert.py`๋ฅผ ๋ฉํํ ํ์ด์ฌ ๋ผ์ด๋ธ๋ฌ๋ฆฌ
- KoBERT, DistilKoBERT๋ฅผ Huggingface Transformers ๋ผ์ด๋ธ๋ฌ๋ฆฌ ํํ๋ก ์ ๊ณต
- `v0.5.1`์์๋ `transformers v3.0` ์ด์์ผ๋ก ๊ธฐ๋ณธ ์ค์นํฉ๋๋ค. (`transformers v4.0` ๊น์ง๋ ์ด์ ์์ด ์ฌ์ฉ ๊ฐ๋ฅ)

### Install Kobert-Transformers

```bash
pip3 install kobert-transformers
```

### How to Use

```python
>>> import torch
>>> from kobert_transformers import get_kobert_model, get_distilkobert_model
>>> model = get_kobert_model()
>>> model.eval()
>>> input_ids = torch.LongTensor([[31, 51, 99], [15, 5, 0]])
>>> attention_mask = torch.LongTensor([[1, 1, 1], [1, 1, 0]])
>>> token_type_ids = torch.LongTensor([[0, 0, 1], [0, 1, 0]])
>>> sequence_output, pooled_output = model(input_ids, attention_mask, token_type_ids)
>>> sequence_output[0]
tensor([[-0.2461,  0.2428,  0.2590,  ..., -0.4861, -0.0731,  0.0756],
        [-0.2478,  0.2420,  0.2552,  ..., -0.4877, -0.0727,  0.0754],
        [-0.2472,  0.2420,  0.2561,  ..., -0.4874, -0.0733,  0.0765]],
       grad_fn=<SelectBackward>)
```

```python
>>> from kobert_transformers import get_tokenizer
>>> tokenizer = get_tokenizer()
>>> tokenizer.tokenize("[CLS] ํ๊ตญ์ด ๋ชจ๋ธ์ ๊ณต์ ํฉ๋๋ค. [SEP]")
['[CLS]', 'โํ๊ตญ', '์ด', 'โ๋ชจ๋ธ', '์', 'โ๊ณต์ ', 'ํฉ๋๋ค', '.', '[SEP]']
>>> tokenizer.convert_tokens_to_ids(['[CLS]', 'โํ๊ตญ', '์ด', 'โ๋ชจ๋ธ', '์', 'โ๊ณต์ ', 'ํฉ๋๋ค', '.', '[SEP]'])
[2, 4958, 6855, 2046, 7088, 1050, 7843, 54, 3]
```

## Reference

- [KoBERT](https://github.com/SKTBrain/KoBERT)
- [DistilKoBERT](https://github.com/monologg/DistilKoBERT)
- [Huggingface Transformers](https://github.com/huggingface/transformers)
