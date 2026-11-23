# Market Research Agent

An end-to-end, multi-agent sunglasses campaign pipeline built in a Jupyter notebook. The project combines live market research, an internal product catalog, text and image generation, and multimodal copywriting to produce an executive-ready marketing report.

## What It Does

The pipeline runs four specialized agents:

1. **Market Research Agent**
   - Searches the web with Tavily for current sunglasses trends.
   - Reads the local sunglasses inventory.
   - Matches 2-3 trends to in-stock products and explains why they fit a summer campaign.

2. **Graphic Designer Agent**
   - Uses `openai:gpt-5-mini` through `aisuite` to turn trend insights into an image prompt and short caption.
   - Calls OpenAI `gpt-image-1-mini` to generate `generated_image.png`.

3. **Copywriter Agent**
   - Sends the generated image and the trend summary to a multimodal model.
   - Returns a campaign quote of no more than 12 words and a justification for the quote.

4. **Packaging Agent**
   - Refines the research for an executive audience.
   - Creates a timestamped Markdown report containing trend insights, the campaign visual, the quote, and the rationale.

The full pipeline is available through `run_sunglasses_campaign_pipeline()`.

## Project Structure

```text
Market Research Agent/
|-- Market_reserch_team.ipynb  # Complete notebook and agent definitions
|-- tools.py                   # Tavily and inventory tool implementations
|-- inventory_utils.py         # Inventory DataFrame and product helpers
|-- utils.py                   # Notebook rendering and HTML display helpers
|-- generated_image.png        # Created by the graphic designer step
|-- campaign_summary_*.md      # Timestamped reports created by the pipeline
`-- README.md
```

## Requirements

- Python 3.10 or newer
- Jupyter Notebook or JupyterLab
- An OpenAI API key
- A Tavily API key
- Internet access for Tavily searches and image generation

Install the notebook dependencies:

```powershell
python -m pip install jupyter pandas requests python-dotenv openai aisuite tavily-python pillow
```

This project does not currently include a `requirements.txt` or a Python package manifest.

## Configuration

Create a `.env` file in the project directory:

```dotenv
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key
```

The following optional Tavily setting is also supported:

```dotenv
DLAI_TAVILY_BASE_URL=your_tavily_api_base_url
```

Keep the `.env` file out of source control. Never commit API keys or generated credentials.

## Setup and Run

From the project directory, start Jupyter:

```powershell
jupyter notebook
```

Open `Market_reserch_team.ipynb`, then run the notebook cells in order.

To run the complete campaign flow from Python code after importing the notebook definitions:

```python
pipeline_result = run_sunglasses_campaign_pipeline()

print(pipeline_result["trend_summary"])
print(pipeline_result["visual"]["image_path"])
print(pipeline_result["quote"]["quote"])
print(pipeline_result["markdown_path"])
```

The pipeline writes:

- `generated_image.png` for the generated campaign visual
- A file such as `campaign_summary_2026-11-23_16-44-50.md` for the final report

## Agent Interfaces

### Market Research Agent

```python
trend_summary = market_research_agent()
```

It can call:

- `tools.tavily_search_tool(query, max_results=5, include_images=False)`
- `tools.product_catalog_tool(max_items=10)`

Tool calls are automatically dispatched through `tools.handle_tool_call()`.

### Graphic Designer Agent

```python
visual = graphic_designer_agent(
    trend_insights=trend_summary,
    caption_style="short punchy",
    size="1024x1024",
)
```

Returns:

```python
{
    "image_path": "generated_image.png",
    "prompt": "...",
    "caption": "...",
}
```

### Copywriter Agent

```python
quote = copywriter_agent(
    image_path="generated_image.png",
    trend_summary=trend_summary,
)
```

Returns a quote, its justification, and the source image path.

### Packaging Agent

```python
report_path = packaging_agent(
    trend_summary=trend_summary,
    image_url="generated_image.png",
    quote=quote["quote"],
    justification=quote["justification"],
)
```

The default report name is `campaign_summary.md`. The orchestration wrapper uses a timestamped filename instead.

## Inventory Data

`inventory_utils.create_inventory_dataframe()` defines five sunglasses styles:

| Item ID | Name | Description |
|---|---|---|
| SG001 | Aviator | Teardrop lenses with thin metal frames |
| SG002 | Wayfarer | Thick angular frames with a retro feel |
| SG003 | Mystique | Upward-sweeping cat-eye-inspired frames |
| SG004 | Sport | Wraparound shield with a performance focus |
| SG005 | Round | Circular lenses with a minimalist, creative aesthetic |

The DataFrame includes `quantity_in_stock` and `price` fields. The inventory is generated locally and seeded for reproducible setup.

## Outputs

The generated Markdown report includes:

- Refined, executive-focused trend insights
- Products recommended for the campaign
- A generated campaign image
- A concise marketing quote
- A rationale explaining how the quote connects visual and trend context
- A report generation date

The notebook also provides styled HTML cards and tables for agent titles, tool calls, tool results, final summaries, and generated assets.

## Troubleshooting

### `TAVILY_API_KEY not found in environment variables`

Confirm that `.env` exists in the project directory and contains `TAVILY_API_KEY`. Restart the notebook kernel after changing environment variables.

### OpenAI authentication or model errors

Confirm that `OPENAI_API_KEY` is valid, that the account has access to the configured models, and that billing or quota limits are available.

### Image generation returns no base64 data

The graphic designer depends on the OpenAI Images API. Check API availability, model access, and account quotas. The notebook expects `gpt-image-1-mini` to return `b64_json`.

### A previous image or report is reused

The generated image has a fixed name: `generated_image.png`. Existing reports are protected by timestamped names only when using `run_sunglasses_campaign_pipeline()`. Remove or rename older outputs if you want to distinguish them manually.
