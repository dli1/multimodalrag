# Paper–Code Experiment Mapping

Paper: *Context-Enriched Figure Indexing for Scientific Multi-Modal RAG: An Industry Case Study at Scale* (SIGIR-AP 2026)

Scope: Production dataset only (≈113,000 figures from 10,000 ScienceDirect articles).

---

## RQ1 — Caption Generation Quality

### Part A: Impact of Textual Context (Table 2, Part A)

Four context configurations compared using MiniCPM as captioning model.

| Config | Paper name | Context content | Code location |
|--------|-----------|-----------------|---------------|
| 0 | Base Context | Original author caption only | `Caption_Exps.ipynb` → `generate_prompt(row, 0)` |
| 1 | Global Context | Title + abstract + keywords | `Caption_Exps.ipynb` → `generate_prompt(row, 1)` |
| 2 | Local Context | Figure mentions (sentences) + mention section | `Caption_Exps.ipynb` → `generate_prompt(row, 2)` |
| 3 | Hybrid Context (**best**) | Figure mentions + keywords + original caption | `Caption_Exps.ipynb` → `generate_prompt(row, 3)` |

The production-scale captioning script (`experiments/production_dataset/captioning/caption_figures_with_contexts.py`) implements the Hybrid Context (config 3) as the default, used for the full 113k-figure run.

### Part B: Comparison of Captioning Models (Table 2, Part B)

All models use Hybrid Context.

| Model | Code location | Notes |
|-------|--------------|-------|
| MiniCPM-V 2.6 | `caption_figures_with_contexts.py` (production scale) + `Caption_Exps.ipynb` (experiments) | Default model; runs on single A10 GPU |
| GPT-4o | `Caption_Exps.ipynb` → `inference_worker_batch` (Azure OpenAI) | Proprietary baseline |

### Part C: Impact of Prompting Strategy (Table 2, Part C)

MiniCPM + Hybrid Context, two prompt variants.

| Prompt | Code location |
|--------|--------------|
| Standard Prompt | `Caption_Exps.ipynb` → `final_prompt` string; same prompt used in `caption_figures_with_contexts.py` |
| Chain-of-Thought (CoT) Prompt | `Caption_Exps.ipynb` → `base_cot_prompt` string; applied via `try_4o_cot` (GPT-4o) and equivalent MiniCPM CoT run (`df_mini_cot`) |

### Caption Quality Evaluation (Table 2 scores, Table 3 human scores)

| Evaluation type | Rubrics | Code location |
|----------------|---------|---------------|
| LLM-as-judge (GPT-4o) | COR, COM, FOC, CIN, VIN (1–4 scale) | `Caption_Exps.ipynb` → `evaluate_captions_with_vllm` + `evaluation_prompt` + `base_eval_rubrics` |
| Human evaluation | Same 5 rubrics, blind rating | `Caption_Exps.ipynb` → `categorize_images` + `display_image_and_captions` |

---

## RQ2 — Retrieval Performance

### Figure Representations Compared (Table 4)

| Representation | Description | Code location |
|----------------|-------------|---------------|
| VE (OpenCLIP-1) | Direct visual embedding via CLIP ViT-B/32 | `src/question_answering/rag/single_vector_store/retrieval.py` → `ClipRetriever` |
| OC (Original Caption) | Author-written caption, indexed as text | Same retrieval pipeline; caption passed as text |
| EC (GPT-4o Enriched) | GPT-4o generated enriched caption | Same retrieval pipeline; swap text input |
| EC (MiniCPM — Ours) | MiniCPM generated enriched caption | Same retrieval pipeline; output of `caption_figures_with_contexts.py` |

### Retrieval Backbones (Table 4, Figure 5)

| Retriever | Code location |
|-----------|--------------|
| BGE (BAAI/bge-m3) | `retrieval.py` → `SummaryStoreAndRetriever` (embedding_model != 'openai' branch) |
| CLIP / OpenCLIP (visual) | `retrieval.py` → `ClipRetriever` |

---

## RQ3 — Answer Generation Quality

### RAG Pipeline Variants (Tables 5 & 6)

All variants use GPT-4o as answer generator. The figure representation controls what is retrieved and passed alongside text chunks.

| Modality | Representation | Code location |
|----------|---------------|---------------|
| Visual | VE (OpenCLIP-1) | `src/question_answering/rag/run_image_only_rag.py` + `rag_pipeline_clip.py` |
| Textual | OC (Original Caption) | `src/question_answering/rag/run_text_only_rag.py` |
| Textual | EC (GPT-4o) | `run_multimodal_rag.py` → `run_pipeline_with_summaries_dual` (swap caption source) |
| Textual | EC (MiniCPM — Ours) | `run_multimodal_rag.py` → `new_pipe` / `run_pipeline_with_summaries_dual` |

Dual vector store (separate stores for text and image) is the architecture used in production experiments: `src/question_answering/rag/separate_vector_stores/`.

### Answer Quality Evaluation

| Metric | Type | Code location |
|--------|------|--------------|
| A.COR (Answer Correctness) | LLM-as-judge | `src/evaluation/evaluation_module.py` → `_evaluate_answer_correctness` |
| A.REL (Answer Relevancy) | LLM-as-judge | `evaluation_module.py` → `_evaluate_answer_relevancy` |
| T.FAI (Text Faithfulness) | LLM-as-judge | `evaluation_module.py` → `_evaluate_text_faithfulness` |
| I.FAI (Image Faithfulness) | LLM-as-judge | `evaluation_module.py` → `_evaluate_image_faithfulness` |

Evaluation entry point: `src/evaluation/evaluate_rag_pipeline.py` → `evaluate_dataframe`.

---

## Benchmark Dataset

| Item | File | Notes |
|------|------|-------|
| 450 QA pairs (150 text-only, 150 image-only, 150 cross-modal) | `experiments/production_dataset/dataset/thesis_450_set.xlsx` | Generated synthetically using Claude Sonnet 4 |
| Benchmark generation notebook | `experiments/production_dataset/dataset_generation/Benchmark_Generation.ipynb` | SPIQA-style methodology |

