<div dir="rtl" align="right">

# چارچوب تولید و ارزیابی Synthetic Benchmark برای سیستم‌های RAG

## معرفی

این پروژه یک چارچوب کامل برای تولید و ارزیابی benchmarkهای سیستم‌های Retrieval-Augmented Generation (RAG) است.

هدف اصلی سیستم، تولید datasetهای ساختاریافته برای ارزیابی دقیق سه لایه اصلی RAG است:

1. Retrieval Layer
2. Generation Layer
3. End-to-End Layer

این framework تلاش می‌کند ارزیابی RAG را از حالت QA ساده خارج کرده و آن را به سمت ارزیابی capability-driven و cognition-aware هدایت کند.

---

# فلسفه طراحی سیستم

benchmarkهای سنتی معمولاً روی موارد زیر تمرکز دارند:

* memorization
* factual lookup
* retrieval ساده
* single-hop QA

اما در استفاده واقعی، کاربران سؤال‌هایی می‌پرسند که:

* نیازمند reasoning هستند
* چند سند را درگیر می‌کنند
* causal analysis می‌خواهند
* scenario analysis می‌خواهند
* recommendation و evaluation نیاز دارند

بنابراین این framework بر اساس دو محور مستقل طراحی شده است:

1. Cognitive Complexity
2. Retrieval Complexity

---

# معماری کلی سیستم

```text
                    ┌────────────────────┐
                    │   User Corpus      │
                    │ PDFs / Docs / KBs  │
                    └─────────┬──────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │ Corpus Preprocessing   │
                 │ Chunking / Indexing    │
                 └─────────┬──────────────┘
                           │
                           ▼
             ┌──────────────────────────────┐
             │ Taxonomy-Aware Prompt Engine │
             │ Cognitive + Retrieval Types  │
             └──────────┬───────────────────┘
                        │
                        ▼
           ┌─────────────────────────────────┐
           │ Strong LLM Synthetic Generator │
           │ Query + Gold Answer + Evidence │
           └──────────┬──────────────────────┘
                      │
                      ▼
        ┌──────────────────────────────────────┐
        │ Synthetic Benchmark Dataset         │
        │ query / answer / chunk_ids / labels │
        └────────────────┬─────────────────────┘
                         │
                         ▼
            ┌───────────────────────────┐
            │ Target RAG System         │
            │ Retrieval + Generation    │
            └──────────┬────────────────┘
                       │
                       ▼
          ┌────────────────────────────────┐
          │ Evaluation Framework           │
          │ Retrieval / Generation / E2E  │
          └────────────────────────────────┘
```

---

# مراحل Pipeline

## مرحله 1 — انتخاب Corpus

کاربر یک corpus انتخاب می‌کند.

Corpus می‌تواند شامل موارد زیر باشد:

* مقاله
* PDF
* مستندات فنی
* Knowledge Base
* Documentation
* داده‌های تخصصی دامنه‌ای

هدف این مرحله، فراهم کردن منبع دانش برای benchmark generation است.

---

## مرحله 2 — Preprocessing و Chunking

در این مرحله:

* documentها chunk می‌شوند
* embedding ساخته می‌شود
* index ایجاد می‌شود
* metadata ذخیره می‌شود

نمونه metadata:

```json
{
  "doc_id": "economics_doc_1",
  "chunk_id": 14,
  "text": "Central banks increase interest rates to control inflation..."
}
```

---

# محورهای اصلی Taxonomy

سیستم از دو محور مستقل استفاده می‌کند:

---

# محور اول — Cognitive Type

این محور نوع عملیات شناختی موردنیاز برای پاسخ را مشخص می‌کند.

| نوع       | توضیح                                                               |
| ------------ | ------------------------------------------------------------------------ |
| Factual      | پاسخ مستقیماً داخل context وجود دارد             |
| Cause-Effect | نیازمند causal reasoning و inference است                      |
| Hypothetical | نیازمند scenario reasoning و synthesis کنترل‌شده است |

---

# محور دوم — Retrieval Type

این محور پیچیدگی retrieval را مشخص می‌کند.

| نوع     | توضیح                                                                    |
| ---------- | ----------------------------------------------------------------------------- |
| Single-hop | تنها یک chunk برای پاسخ کافی است                         |
| Multi-hop  | نیازمند چند مرحله retrieval و reasoning است                |
| Multi-doc  | نیازمند ترکیب اطلاعات از چند سند مختلف است |

---

# ترکیب دو محور

این دو محور به‌صورت orthogonal با یکدیگر ترکیب می‌شوند.

---

# ماتریس ترکیبی Taxonomy

| Cognitive Type | Retrieval Type | توضیح                                                                              |
| -------------- | -------------- | --------------------------------------------------------------------------------------- |
| Factual        | Single-hop     | پاسخ مستقیماً در یک chunk وجود دارد                             |
| Factual        | Multi-hop      | پاسخ factual است اما نیازمند چند مرحله retrieval است        |
| Factual        | Multi-doc      | پاسخ factual نیازمند ترکیب اطلاعات از چند document است   |
| Cause-Effect   | Single-hop     | رابطه علّی در یک chunk قابل استخراج است                      |
| Cause-Effect   | Multi-hop      | causal reasoning نیازمند چند evidence chain است                            |
| Cause-Effect   | Multi-doc      | تحلیل علّی نیازمند synthesis بین چند document است              |
| Hypothetical   | Single-hop     | تحلیل سناریو با evidence محدود انجام می‌شود                |
| Hypothetical   | Multi-hop      | سناریو نیازمند چند مرحله reasoning و retrieval است             |
| Hypothetical   | Multi-doc      | تحلیل فرضی نیازمند synthesis گسترده بین چند document است |

---

# مثال‌های هر دسته

## Factual + Single-hop

### مثال

> «پایتخت ژاپن چیست؟»

### Challenge اصلی

* retrieval accuracy
* grounding

---

## Factual + Multi-hop

### مثال

> «کشوری که مقر اتحادیه اروپا در آن قرار دارد چه واحد پولی دارد؟»

### Challenge اصلی

* evidence chaining
* intermediate inference

---

## Factual + Multi-doc

### مثال

> «مجموع ظرفیت تولید برق کشورهای عضو G7 چقدر است؟»

### Challenge اصلی

* multi-document aggregation
* evidence coverage

---

## Cause-Effect + Single-hop

### مثال

> «چرا افزایش نرخ بهره باعث کاهش تورم می‌شود؟»

### Challenge اصلی

* causal extraction
* grounded reasoning

---

## Cause-Effect + Multi-hop

### مثال

> «چگونه افزایش نرخ بهره می‌تواند بر بازار مسکن اثر بگذارد؟»

### Challenge اصلی

* multi-step reasoning
* causal chaining

---

## Cause-Effect + Multi-doc

### مثال

> «تحریم‌های اقتصادی چگونه بر زنجیره تأمین جهانی اثر می‌گذارند؟»

### Challenge اصلی

* distributed reasoning
* evidence synthesis

---

## Hypothetical + Single-hop

### مثال

> «اگر نرخ مالیات ۵٪ افزایش یابد چه اتفاقی می‌افتد؟»

### Challenge اصلی

* controlled generation
* hallucination prevention

---

## Hypothetical + Multi-hop

### مثال

> «اگر بانک مرکزی نرخ بهره را افزایش دهد، چه اثری بر سرمایه‌گذاری startupها خواهد داشت؟»

### Challenge اصلی

* multi-step scenario reasoning
* grounded extrapolation

---

## Hypothetical + Multi-doc

### مثال

> «اگر انرژی هسته‌ای در اروپا حذف شود، اقتصاد و بازار انرژی چه تغییری خواهند کرد؟»

### Challenge اصلی

* multi-document synthesis
* scenario modeling
* hallucination resistance
* end-to-end reasoning

---

# مرحله 3 — تولید Promptهای Synthetic

در این مرحله، سیستم با استفاده از taxonomy بالا promptهای ساختاریافته تولید می‌کند.

نمونه:

```text
Generate a Cause-Effect Multi-hop question based on the provided corpus.
The question must require causal reasoning across multiple evidence chunks.
Also generate:
- gold answer
- supporting chunk ids
- supporting document ids
```

---

# مرحله 4 — تولید Synthetic Dataset

یک LLM قدرتمند:

* corpus
* taxonomy prompts

را دریافت کرده و dataset تولید می‌کند.

هر sample شامل موارد زیر است:

| فیلد       | توضیح                |
| -------------- | ------------------------- |
| query          | سؤال                  |
| gold_answer    | پاسخ مرجع         |
| chunk_ids      | chunkهای مرتبط    |
| doc_ids        | documentهای مرتبط |
| cognitive_type | نوع شناختی       |
| retrieval_type | نوع retrieval          |
| difficulty     | سطح سختی           |

---

# ساختار نمونه Dataset

```json
{
  "query": "چرا افزایش نرخ بهره باعث کاهش تورم می‌شود؟",
  "gold_answer": "بانک مرکزی با افزایش نرخ بهره میزان تقاضا و هزینه‌کرد را کاهش می‌دهد و این موضوع تورم را کنترل می‌کند.",
  "chunk_ids": [12, 18],
  "doc_ids": ["economics_doc_1"],
  "cognitive_type": "Cause-Effect",
  "retrieval_type": "Multi-hop",
  "difficulty": "Medium"
}
```

---

# مرحله 5 — اجرای سیستم RAG هدف

در این مرحله queryهای benchmark به سیستم RAG هدف داده می‌شوند.

Pipeline داخلی سیستم هدف:

```text
Query
→ Embedding
→ Retrieval
→ Reranking
→ Context Construction
→ Generation
→ Final Answer
```

---

# مرحله 6 — Evaluation Framework

ارزیابی در سه لایه مستقل انجام می‌شود:

1. Retrieval Evaluation
2. Generation Evaluation
3. End-to-End Evaluation

---

# ارزیابی Retrieval Layer

این بخش بررسی می‌کند آیا سیستم توانسته evidence مناسب را retrieve کند یا خیر.

---

## دسته‌های وابسته به Retrieval

| دسته   | وابستگی      |
| ---------- | ------------------- |
| Factual    | بسیار زیاد |
| Single-hop | بسیار زیاد |
| Multi-hop  | زیاد            |
| Multi-doc  | بسیار زیاد |

---

## معیارهای ارزیابی Retrieval

| معیار        | توضیح                                                               |
| ----------------- | ------------------------------------------------------------------------ |
| Recall@K          | آیا evidence صحیح retrieve شده است؟                        |
| Precision@K       | چه درصدی از chunkهای retrieve شده مرتبط هستند؟ |
| Hit Rate          | آیا حداقل یک chunk صحیح پیدا شده؟                  |
| MRR               | کیفیت ranking retrieval                                             |
| Context Relevance | ارتباط semantic context با query                                 |
| Context Coverage  | کامل بودن evidence                                               |

---

# ارزیابی Generation Layer

این بخش کیفیت reasoning و grounded generation را بررسی می‌کند.

---

## دسته‌های وابسته به Generation

| دسته     | وابستگی      |
| ------------ | ------------------- |
| Cause-Effect | بسیار زیاد |
| Hypothetical | بسیار زیاد |
| Multi-hop    | زیاد            |

---

## معیارهای ارزیابی Generation

| معیار          | توضیح                               |
| ------------------- | ---------------------------------------- |
| Faithfulness        | سازگاری پاسخ با context     |
| Correctness         | شباهت معنایی با gold answer |
| Logical Consistency | انسجام reasoning                   |
| Causal Correctness  | صحت روابط علّی               |
| Groundedness        | وابستگی پاسخ به evidence    |
| Hallucination Rate  | میزان اطلاعات unsupported    |
| Answer Relevance    | ارتباط پاسخ با query         |

---

# ارزیابی End-to-End Layer

این بخش کل pipeline سیستم را ارزیابی می‌کند.

---

## دسته‌های وابسته به End-to-End

| دسته     | وابستگی      |
| ------------ | ------------------- |
| Hypothetical | بسیار زیاد |
| Multi-doc    | بسیار زیاد |
| Hard Samples | بسیار زیاد |

---

## معیارهای ارزیابی End-to-End

| معیار                 | توضیح                                        |
| -------------------------- | ------------------------------------------------- |
| End-to-End Accuracy        | صحت پاسخ نهایی                        |
| Task Completion            | میزان پاسخ کامل به سؤال        |
| Robustness                 | پایداری روی دسته‌های مختلف |
| Multi-Document Synthesis   | کیفیت ترکیب اطلاعات              |
| Scenario Reasoning Quality | کیفیت تحلیل سناریو                |
| Overall Utility            | مفید بودن پاسخ                        |
| Reasoning Success Rate     | موفقیت reasoning چندمرحله‌ای     |

---

---

# نتیجه‌گیری

این framework یک سیستم کامل برای:

* تولید synthetic benchmark
* ارزیابی retrieval
* ارزیابی reasoning
* ارزیابی grounded generation
* تحلیل failure mode

در سیستم‌های مدرن RAG فراهم می‌کند.

برخلاف benchmarkهای سنتی QA، این framework روی:

* cognitive reasoning
* retrieval complexity
* synthesis capability
* end-to-end robustness

تمرکز دارد و به استفاده واقعی کاربران از سیستم‌های RAG نزدیک‌تر است.



# References

Hou, Yutao, et al.
   **Compound-QA: A Benchmark for Evaluating LLMs on Compound Questions**
   2024.
   https://arxiv.org/abs/2411.10163

ongoing...


</div>
