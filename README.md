# OpenReview MCP server

[![Build Status](https://img.shields.io/github/actions/workflow/status/anyakors/openreview-mcp-server/ci.yml?branch=main)](https://github.com/anyakors/openreview-mcp-server/actions)
[![License](https://img.shields.io/github/license/anyakors/openreview-mcp-server.svg)](LICENSE)
[![Python Version](https://img.shields.io/badge/python-3.12%2B-blue.svg)](https://www.python.org/downloads/release/python-3120/)

A Model Context Protocol (MCP) server that provides access to OpenReview data for research and analysis. This server allows you to search for users, fetch papers, and export research data from major ML conferences (ICML, ICLR, NeurIPS).

## Features

- **User search**: Find OpenReview profiles by email address
- **Paper retrieval**: Fetch all papers by a specific author
- **Conference papers**: Get papers from specific venues (ICLR, NeurIPS, ICML) and years
- **Keyword search**: Search papers by keywords across multiple conferences
- **JSON&PDF export**: Export search results to PDF and JSON files for convenient reading or further analysis and coding assistant usage

## Installation

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/openreview-mcp-server.git
cd openreview-mcp-server
```

### 2. Create and activate virtual environment
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install the package
```bash
pip install -e .
```

## Configuration for Cursor

### Step 1: Get your OpenReview credentials
You'll need your OpenReview account email and password.

### Step 2: Find your Cursor MCP configuration file

Either run cmd+shift+P to open the Command Palette and find MCP settings that will lead you to the mcp.json, or look for:

**Cursor:** `~/.cursor/mcp.json`  

### Step 3: Add the OpenReview MCP server

Open the MCP configuration file and add the `openreview` server to the `mcpServers` section:

```json
{
  "mcpServers": {
    "openreview": {
      "command": "/ABSOLUTE/PATH/TO/openreview-mcp-server/venv/bin/python",
      "args": ["-m", "openreview_mcp_server"],
      "cwd": "/ABSOLUTE/PATH/TO/openreview-mcp-server",
      "env": {
        "OPENREVIEW_USERNAME": "your_email@domain.com",
        "OPENREVIEW_PASSWORD": "your_password",
        "OPENREVIEW_BASE_URL": "https://api2.openreview.net",
        "OPENREVIEW_DEFAULT_EXPORT_DIR": "./openreview_exports"
      }
    }
  }
}
```

**Important:** 
- Replace `/ABSOLUTE/PATH/TO/openreview-mcp-server` with the actual path (e.g., `/Users/yourname/workspace/openreview-mcp-server`)
- Replace `your_email@domain.com` and `your_password` with your OpenReview credentials
- Use the full path to the venv Python interpreter (ending in `/venv/bin/python`)

**Example configuration:**
```json
{
  "mcpServers": {
    "openreview": {
      "command": "/Users/john/workspace/openreview-mcp-server/venv/bin/python",
      "args": ["-m", "openreview_mcp_server"],
      "cwd": "/Users/john/workspace/openreview-mcp-server",
      "env": {
        "OPENREVIEW_USERNAME": "john@university.edu",
        "OPENREVIEW_PASSWORD": "mySecurePassword123",
        "OPENREVIEW_BASE_URL": "https://api2.openreview.net",
        "OPENREVIEW_DEFAULT_EXPORT_DIR": "./openreview_exports"
      }
    }
  }
}
```

### Step 4: Restart Cursor

Completely quit and reopen Cursor for the MCP server to load.

## Usage

Once configured and Cursor is restarted, you can use natural language to interact with the OpenReview MCP server:

### Example queries:

**Search for papers:**
```
Search OpenReview for papers about "multimodal tokenization" from ICML 2025, ICLR 2025 and NeurIPS 2025
```

**Get your own papers:**
```
Get my papers from OpenReview using email researcher@university.edu
```

**Export papers with PDFs:**
```
Export papers about "multimodal tokenization" from ICLR 2024, download PDFs and extract text
```

**Get conference papers:**
```
Show me all papers from NeurIPS 2024
```

The server will automatically:
- Fetch papers from OpenReview
- Search across titles, abstracts, and authors
- Download and extract text from PDFs
- Export results to JSON for further analysis

Exported files are saved to `./openreview_exports/` by default (or your custom directory).

## Example output

![Example Output](public/output.jpg)

## Available tools

### search_user
Find a user profile by email address.

```python
search_user(email="researcher@university.edu", include_publications=true)
```

### get_user_papers
Fetch all papers published by a specific user.

Input schema:

| Field    | Type   | Description                                              | Required | Default   | Allowed Values         |
|----------|--------|----------------------------------------------------------|----------|-----------|-----------------------|
| `email`  | string | Email address of the user whose papers to fetch          | Yes      | —         | —                     |
| `format` | string | Format of the response: summary or detailed              | No       | summary   | `summary`, `detailed` |

```python
get_user_papers(email="researcher@university.edu", format="detailed")
```

### get_conference_papers
Get papers from a specific conference and year.

Input schema:

| Field    | Type     | Description                                                                 | Required | Default   | Allowed Values                    |
|----------|----------|-----------------------------------------------------------------------------|----------|-----------|-----------------------------------|
| `venue`  | string   | Conference venue (e.g., `"ICLR.cc"`, `"NeurIPS.cc"`, `"ICML.cc"`)           | Yes      | —         | `ICLR.cc`, `NeurIPS.cc`, `ICML.cc`|
| `year`   | string   | Conference year (e.g., `"2024"`, `"2025"`)                                  | Yes      | —         | Four-digit year (e.g., `2024`)    |
| `limit`  | integer  | Maximum number of papers to return                                          | No       | `50`      | 1–1000                            |
| `format` | string   | Format of the response: summary or detailed                                 | No       | `summary` | `summary`, `detailed`             |

```python
get_conference_papers(venue="ICLR.cc", year="2024", limit=50)
```

### search_papers
Search for papers by keywords across multiple conferences.

Search modes:
- **any**: returns papers that match at least one of the keywords in the specified fields. If any keyword is found, the paper is included.
- **all**: returns papers that match all of the keywords in the specified fields. Only papers containing every keyword are included.
- **exact**: returns papers that contain the exact phrase (all keywords together, in order) in the specified fields.

Input schema:

| Field           | Type     | Description                                                                                  | Required | Default                | Allowed Values                |
|-----------------|----------|----------------------------------------------------------------------------------------------|----------|------------------------|-------------------------------|
| `query`         | string   | Keywords or phrase to search for (e.g., `"time series token merging"`, `"neural networks"`)  | Yes      | —                      | —                             |
| `venues`        | array    | List of conference venues and years to search in.<br>Each item:<br>• `venue`: string<br>• `year`: string | Yes      | —                      | —                             |
| `search_fields` | array    | Fields to search in. Options: `"title"`, `"abstract"`, `"authors"`                           | No       | `["title", "abstract"]`| `"title"`, `"abstract"`, `"authors"` |
| `match_mode`    | string   | How keywords are matched:<br>• `"any"`: match any keyword<br>• `"all"`: match all keywords<br>• `"exact"`: match exact phrase | No       | `"all"`                | `"any"`, `"all"`, `"exact"`   |
| `limit`         | integer  | Maximum number of results to return                                                          | No       | `20`                   | 1–100                         |
| `min_score`     | number   | Minimum match score (between 0.0 and 1.0)                                                    | No       | `0.1`                  | 0.0–1.0                       |

```python
search_papers(
  query="time series token merging",
  match_mode="all",
  search_fields=["title", "abstract"],
  venues=[
    {"venue": "ICLR.cc", "year": "2024"},
    {"venue": "NeurIPS.cc", "year": "2024"}
  ],
  limit=20
)
```

### export_papers
Export search results to JSON files for analysis.

Input schema:

| Field             | Type     | Description                                                                                  | Required | Default                  | Allowed Values                |
|-------------------|----------|----------------------------------------------------------------------------------------------|----------|--------------------------|-------------------------------|
| `query`           | string   | Keywords to search for before export                                                          | Yes      | —                        | —                             |
| `venues`          | array    | List of conference venues and years to export from.<br>Each item:<br>• `venue`: string<br>• `year`: string | Yes      | —                        | —                             |
| `export_dir`      | string   | Directory to export JSON files to                                                             | No       | `./openreview_exports`   | —                             |
| `filename`        | string   | Base filename for the export (without extension)                                              | No       | *auto-generated*         | —                             |
| `include_abstracts`| boolean | Whether to include full abstracts in export                                                   | No       | `True`                   | `True`, `False`               |
| `min_score`       | number   | Minimum match score for search results (0.0 to 1.0)                                          | No       | `0.2`                    | 0.0–1.0                       |
| `max_papers`      | integer  | Maximum number of papers to export and download                                               | No       | `3`                      | 1–10                          |
| `download_pdfs`   | boolean  | Whether to download PDFs and extract full text content                                        | No       | `True`                   | `True`, `False`               |

```python
export_papers(
  query="neural networks",
  venues=[
    {"venue": "ICLR.cc", "year": "2024"},
    {"venue": "ICML.cc", "year": "2024"}
  ],
  max_papers=1, 
  download_pdfs=true, 
  include_abstracts=true,
  export_dir="./research_exports"
)
```

## Example workflow

1. Search for papers on a topic of interest:
```python
search_papers(query="time series forecasting", match_mode="all", venues=[{"venue": "ICLR.cc", "year": "2024"}])
```

2. Export relevant papers to JSON:
```python
export_papers(query="time series token merging", venues=[{"venue":"ICML.cc","year":"2025"}], max_papers=1, download_pdfs=true, include_abstracts=true)
```

3. Use the exported JSON files with Claude Code to implement methods inspired by the research.

## Supported conferences

- ICLR (International Conference on Learning Representations)
- NeurIPS (Conference on Neural Information Processing Systems)
- ICML (International Conference on Machine Learning)

## License

MIT License