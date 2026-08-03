# Context-Enriched Figure Indexing for Scientific Multi-Modal RAG: An Industry Case Study at Scale


## Main Features

- **Enhanced Figure Captioning**: Based on existing textual contexts, generate detailed and retrieval-optimized figure captions.
- **Multimodal RAG**: Combines text and image modalities, allowing the model to generate responses using both textual and visual information. Alternatively, text-only and image-only are also options.
- **Evaluation with traditional metrics**: Calculate MRR, recall@k for retrieval as well as BLEU and ROUGE scores for answer generation.
- **Evaluation with LLM as a Judge**: Uses an LLM-based evaluation framework to assess generated responses across multiple metrics such as:
  - **Answer Correctness**
  - **Answer Relevancy**
  - **Text Context Relevancy**
  - **Image Context Relevancy**
  - **Text Faithfulness**
  - **Image Faithfulness**



## Paper–Code Mapping

See [EXPERIMENTS.md](EXPERIMENTS.md) for a detailed mapping between the paper's experiments and the corresponding code locations.

## Usage

To use the system, follow these steps:

0.**Install Dependencies**:  
   Use the `requirements.txt` file and set up the correct environment.

1. **Define Models and Paths**:  
   Have a look at the `rag_env.py` file and define your desired models and paths.

2. **Caption Figure Images**:
   
   Use the `caption_figures_with_contexts.py` file to generate enhanced captions for figures using textual context and save into the correct data structure. This requires access to the figure images and necessary textual context.
   
3. **Run the Desired RAG Experiment**:  
   See the `question_answering` folder and choose the appropriate script. Same for Evaluation see `evaluation`.


