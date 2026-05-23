# MultiCTbench

MultiCTbench is a multimodal benchmark for evaluating structured data understanding and generation over charts, tables, and their combinations. It is designed for multimodal large language models (MLLMs) and focuses on realistic multi-image scenarios where multiple charts and/or tables need to be interpreted jointly.

The benchmark is constructed from real-world academic papers across eight arXiv domains. Charts and tables are manually cropped from source PDFs and saved as image inputs. Each image set contains 2 to 4 semantically related charts and/or tables from the same paper, including chart-table, multi-chart, and multi-table settings. In total, MultiCTbench contains 1,018 image sets and 3,380 manually annotated questions.

MultiCTbench evaluates four core capabilities:
- Perception: directly reading values, labels, legends, chart types, and table structures from visual inputs.
- Reasoning: performing numerical calculation, comparison, aggregation, trend analysis, and cross-image inference based on charts and tables.
- Structured data extraction: converting visual chart information into structured table formats with row labels, column labels, and values.
- Visualization generation: generating executable Python code to visualize tabular data as a specified chart type.

In addition to the main benchmark, the project includes supplementary evaluation settings for analyzing chart-table complementarity, textual-context effects, understanding-generation interaction, reasoning guidance, reference specificity, and evidence dependency. These settings examine whether charts and tables provide complementary evidence, whether source-paper text improves performance, and whether understanding and generation can mutually reinforce each other.

## Project Structure

```text
MultiCTbench/
├── chart-table/             # Chart-table setting
├── multicharts/             # Multi-chart setting
├── multitables/             # Multi-table setting
├── table_and_chart/         # Chart-table complementarity experiments
├── und_gen/                     # Understanding-generation interaction experiments
├── exact_questions_300/     # Quick-evaluation subset
├── evaluation_openai*.ipynb # OpenAI-style evaluation notebooks
└── README.md
```

```exact_questions_300/``` contains a balanced quick-evaluation subset sampled from the three main settings: ```chart-table```, ```multicharts```, and ```multitables```. For each setting, it includes 300 perception questions and 300 reasoning questions.




Data directories are usually organized by `task type / subject / arXiv ID / image files`, for example:

```text
chart-table/cs/
├── questions_single.json
├── questions_multi.json
├── questions_text.json
└── 2508.04417v1/
    ├── chart1.jpg
    └── table1.jpg
```

The subject-level directories include eight arXiv domains: ```cs```, ```econ```, ```eess```, ```math```, ```physics```, ```q-bio```, ```q-fin```, and ```stat```.

## JSON Fields

A common sample contains the following fields:

```json
{
  "question_id": 1,
  "items": ["chart1", "table1"],
  "path": {
    "chart1": "2508.04417v1\\chart1.jpg",
    "table1": "2508.04417v1\\table1.jpg"
  },
  "caption": {
    "chart1": "Caption of chart.",
    "table1": "Caption of table."
  },
  "question": "Question text.",
  "question_type": "reasoning",
  "answer": 0.34,
  "answer_type": "numeric"
}
```

Field descriptions:

| Field | Description |
| --- | --- |
| `question_id` | Unique ID of the question. |
| `items` | Input items involved in the question, such as `chart1`, `table1`, or `text`.|
| `path` | Relative image paths for each visual input item. |
| `caption` | Short caption for each chart or table. |
| `text` | Optional textual context, mainly used in  `questions_text.json` . |
| `question` | The final question given to the model. |
| `question_chart` / `question_table` / `question_both` | Questions in `table_and_chart`, corresponding to chart-only, table-only, and chart+table inputs. |
| `question_comp` / `question_gen` |Intermediate understanding questions and generation questions used in `und_gen`. |
| `question_type` | Task type, such as `perceptual`, `reasoning`, `chart to table`, or `table to chart`. |
| `answer` | Ground-truth answer. |
| `answer_type` | Answer format, such as `numeric`, `numeric_list`, `text`, `choice`, `choice_list`, `table_list`, or `code`. |

## API Configuration

The evaluation notebooks use the `openai.OpenAI` client. Before running the notebooks, set the API key, model name, and optionally the base URL in the corresponding notebook.

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    # base_url="YOUR_BASE_URL",  # Optional: only needed for OpenAI-compatible third-party services
)

model = "gpt-5.5"
```

For the official OpenAI API, keep `base_url` commented out. For OpenAI-compatible third-party services, uncomment `base_url` and replace both `base_url` and `model` with the corresponding service configuration.

## Evaluation Entry Points

The repository provides several notebooks for different evaluation settings:

| Notebook | Usage |
|---|---|
| `evaluation_openai.ipynb` | General evaluation entry point with evaluation functions for different answer types. |
| `evaluation_openai_comp_ct2tb.ipynb` | Evaluates chart-to-table generation with an intermediate understanding step. |
| `evaluation_openai_comp_ct2tb_wo.ipynb` | Direct chart-to-table generation without the intermediate understanding step. |
| `evaluation_openai_comp_tb2ct.ipynb` | Evaluates table-to-chart generation with an intermediate understanding step. |
| `evaluation_openai_comp_tb2ct_wo.ipynb` | Direct table-to-chart generation without the intermediate understanding step. |
| `evaluation_openai_ct2tb_comp.ipynb` | Evaluates understanding tasks after converting charts into tables as intermediate representations. |
| `evaluation_openai_ct2tb_comp_wo.ipynb` | Direct understanding evaluation without chart-to-table intermediate generation. |
| `evaluation_openai_tb2ct_comp.ipynb` | Evaluates understanding tasks after converting tables into charts as intermediate visual inputs. |
| `evaluation_openai_tb2ct_comp_wo.ipynb` | Direct understanding evaluation without table-to-chart intermediate generation. |
| `evaluation_openai_chart.ipynb` | Evaluation with chart-only inputs. |
| `evaluation_openai_table.ipynb` | Evaluation with table-only inputs. |
| `evaluation_openai_chart_table.ipynb` | Evaluation with both chart and table inputs. |
| `evaluation_openai_text.ipynb` | Evaluation with chart/table images and textual context from the source paper. |
| `evaluation_openai_text_wo.ipynb` | Evaluation without textual context. |
| `evaluation_openai_reason.ipynb` | Evaluation with explicit reasoning guidance. |
| `evaluation_openai_noreference.ipynb` | Evaluation after replacing explicit references such as `Chart 1` or `Table 1` with generic references. |
| `evaluation_openai_noimage.ipynb` | Evaluation without chart/table image inputs. |

Recommended quick start:

1. Open `evaluation_openai.ipynb` or the notebook corresponding to the target evaluation setting.
2. Fill in `base_url`, `api_key`, and the model name.
3. Set the data root, such as `ROOT_PATH = "exact_questions_300"` or `ROOT_PATH = "chart-table"`.
4. Select the dataset, subject, and JSON question file to evaluate.
5. Run the notebook cells sequentially and check the average scores for each question type.

## Evaluation Logic

The notebooks implement automatic evaluation for common answer types:

- `numeric`: Extracts a single number and evaluates it using a relative tolerance of `1%` and an absolute tolerance of `1e-6`.
- `numeric_list`: Extracts a list of numbers and checks whether the list length and each value match the tolerance.
- `text`: Normalizes case, spaces, and punctuation, then computes edit-distance similarity with a default threshold of `0.9`.
- `choice`: Extracts a single uppercase option letter and performs exact matching.
- `choice_list`: Extracts a list of option letters and evaluates it by set-level matching.
- `table_list`: Parses a 2D table and computes F1 based on cell position and value similarity.
- `code`: Checks code executability, chart type, data values, required operations, and readability.

Raw model outputs are first normalized by an answer-extraction LLM into the `answer: ...` format, and then passed to the corresponding evaluation function.




