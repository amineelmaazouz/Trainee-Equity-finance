# JPX — Securities to Be Delisted Dashboard

Jupyter-based report listing all securities scheduled to be delisted from the
Japan Exchange Group (JPX), built for a trading assessment exercise.

The pipeline scrapes the third section of the
[JPX supervision page](https://www.jpx.co.jp/english/listing/market-alerts/supervision/index.html),
downloads each security's disclosure PDF, extracts the delisting date from it,
and produces a sortable report with:

1. Security code
2. Designation date
3. Delisting date (as stated in each PDF)

An Excel export (`Securities to be delisted.xlsx`) is generated alongside the
in-notebook dashboard.

## Installation

```bash
pip install -r requirements.txt
```

Or simply open the notebook — the first cell installs the requirements.

## Usage

Open the notebook and run all cells in order. The full pipeline runs in two lines:

```python
pipeline = JpxDelisted()
df = pipeline.evaluate()
```

## Design

The pipeline is a single cohesive class, `JpxDelisted`, orchestrated by
`evaluate()`:

| Step | Method | What it does |
|---|---|---|
| 1 | `fetch_page()` | GET the JPX page, parse HTML with BeautifulSoup |
| 2 | `find_delisted_table()` | Locate the "Securities to Be Delisted" section, extract one record per security |
| 3 | `batch()` | Download all PDFs and extract each delisting date (regex over pdfplumber text) |
| 4 | `dataframe()` | Build the sorted pandas report and export to Excel |

Configuration (URLs, timeout, worker count) is centralised in a `Params` class.

## Technical choices

- **Multithreading, not multiprocessing.** Downloading ~19 PDFs is I/O-bound:
  the program spends its time waiting on the network, not computing. A
  `ThreadPoolExecutor` turns total time from the *sum* of request latencies
  into roughly the *max* per wave of workers. `batch()` accepts
  `multi_threading=False` to reproduce the sequential baseline; the benchmark
  cell compares both.
- **Shared `requests.Session`.** One TCP/TLS connection is opened and reused
  across the page fetch and all PDF downloads, instead of paying the handshake
  cost on every request.
