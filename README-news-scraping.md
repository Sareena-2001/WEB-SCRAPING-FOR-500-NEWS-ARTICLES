# Web Scraping and Text Preprocessing for 500 News Articles

A Python project that collects news metadata from five publishers through their RSS feeds and stores the results as CSV files. The repository also includes a text-preprocessing and exploratory-data-analysis workflow using pandas, NLTK, TF-IDF, and visualization libraries.

## Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [News Sources](#news-sources)
- [Repository Structure](#repository-structure)
- [Data Format](#data-format)
- [Technologies](#technologies)
- [Installation](#installation)
- [Run the News Scraper](#run-the-news-scraper)
- [Run Preprocessing and Visualization](#run-preprocessing-and-visualization)
- [How the Code Works](#how-the-code-works)
- [Important Caveats](#important-caveats)
- [Limitations and Future Improvements](#limitations-and-future-improvements)
- [Responsible Use](#responsible-use)

## Project Overview

The project has two main parts:

1. **RSS news collection:** Requests RSS feeds from five news publishers, parses feed entries, and saves article metadata to CSV files.
2. **Text preprocessing and exploration:** Loads the CSV files, removes duplicate rows, drops rows with missing descriptions, preprocesses description text, vectorizes it using TF-IDF, and generates exploratory visualizations.

The repository includes CSV datasets for each publisher, Python scripts, and Jupyter notebooks. The included CSV files each contain 500 data rows, for **2,500 rows in total across the five files**. This is the number of rows in the supplied files; it does not establish that all rows are unique articles or that the live scraper will always collect 500 articles per publisher.

## Features

- Collects RSS feed entries from five news publishers.
- Uses Python threads to process the publisher feeds concurrently.
- Stops each feed-collection loop after 500 entries or a 60-second time limit, or when no entries are returned.
- Captures article title, link, publication date when available, and description.
- Exports the collected metadata to publisher-specific CSV files.
- Removes duplicate rows and excludes records with missing descriptions.
- Cleans description text by lowercasing, removing punctuation and English stop words, and applying Porter stemming.
- Converts text to TF-IDF features.
- Generates class-distribution plots and word clouds as exploratory visualizations.

## News Sources

The scraper defines RSS feed URLs for these publishers:

| Publisher | Output CSV |
|---|---|
| BBC | `BBC_articles.csv` |
| CNN | `CNN_articles.csv` |
| The Guardian | `The Guardian_articles.csv` |
| The New York Times | `New York Times_articles.csv` |
| Al Jazeera | `Al Jazeera_articles.csv` |

RSS feed contents, availability, and publisher feed formats can change over time. The number of entries collected by a future run may therefore differ from the supplied CSV files.

## Repository Structure

```text
.
├── Al Jazeera_articles.csv
├── BBC_articles.csv
├── CNN_articles.csv
├── New York Times_articles.csv
├── The Guardian_articles.csv
├── DATA_PREPROCESSING_AND_VISUALIZATION.ipynb
├── data_preprocessing_and_visualization.py
├── web_scraping_python_script.ipynb
└── web_scraping_python_script.py
```

- `web_scraping_python_script.py`: RSS collection script and the main runnable scraper.
- `web_scraping_python_script.ipynb`: Notebook version of the RSS collection code.
- `data_preprocessing_and_visualization.py`: Text cleaning, TF-IDF, resampling, and exploratory visualization functions.
- `DATA_PREPROCESSING_AND_VISUALIZATION.ipynb`: Notebook version of the preprocessing and visualization workflow.
- `*_articles.csv`: Included article metadata for each publisher.

## Data Format

Each CSV file uses the following columns:

| Column | Description |
|---|---|
| `title` | Article title from the RSS entry |
| `link` | Article URL from the RSS entry |
| `published` | Publication date string when available; otherwise a fallback value |
| `description` | Description or summary text from the RSS entry; may use a fallback when unavailable |

The scraper saves feed metadata. It does **not** fetch and parse the full text of each linked news article.

## Technologies

The repository imports or uses:

- **Python** for scripting.
- **requests** for HTTP requests to RSS feeds.
- **feedparser** for parsing RSS responses.
- **pandas** for tabular data and CSV export.
- **threading** and **time** for concurrent collection and time limits.
- **NumPy** for generating placeholder labels in the preprocessing workflow.
- **NLTK** for English stop words and Porter stemming.
- **scikit-learn** for TF-IDF vectorization, train/test split utilities, and related text workflows.
- **imbalanced-learn** for SMOTE resampling.
- **Matplotlib** and **Seaborn** for plots.
- **WordCloud** for word-cloud visualization.
- **Jupyter Notebook** for interactive execution.

The repository does not include a `requirements.txt` or a pinned environment file, so the exact dependency versions used originally are not specified.

## Installation

### 1. Download the repository

Clone the repository or download it as a ZIP from GitHub, then open a terminal in the project directory.

### 2. Create a virtual environment (recommended)

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Activate it on macOS or Linux:

```bash
source .venv/bin/activate
```

### 3. Install the dependencies

For the scraper:

```bash
python -m pip install --upgrade pip
python -m pip install requests feedparser pandas
```

For preprocessing and visualization:

```bash
python -m pip install numpy matplotlib seaborn nltk scikit-learn imbalanced-learn wordcloud jupyter
```

These commands install the libraries imported by the code. They are suggested setup commands, not a tested, version-pinned dependency specification.

## Run the News Scraper

Run the main script from the repository directory:

```bash
python web_scraping_python_script.py
```

The script defines RSS feeds for all five publishers and starts a separate thread for each source. Each thread:

1. Requests its configured RSS URL.
2. Parses the response with `feedparser`.
3. extracts title, link, publication date, and description from available entries.
4. Continues until it reaches 500 entries, the 60-second limit, or a response contains no entries.
5. Saves the collected data to a publisher-specific CSV file in the current working directory.

The output filenames are defined in the script using the publisher name, followed by `_articles.csv`.

### Configuration

The following values are defined near the top of `web_scraping_python_script.py` or inside `fetch_articles`:

```python
TIME_LIMIT = 60
max_articles = 500
```

- `TIME_LIMIT` is the maximum time, in seconds, allowed for each source's collection loop.
- `max_articles` is the maximum number of feed entries the loop attempts to collect per source.

The script includes a one-second delay between feed requests. It does not define environment variables or require API keys in the current implementation.

### Example output

The script prints progress messages, such as the source being fetched, the number of entries collected, and the CSV filename written. Actual counts and messages depend on the current RSS responses and network conditions.

## Run Preprocessing and Visualization

The preprocessing script is:

```bash
python data_preprocessing_and_visualization.py
```

**Before running it:** Update the `file_paths` list at the bottom of the script. The current paths point to `/content/sample_data/`, a Google Colab location. For local execution, replace them with the actual CSV paths, for example:

```python
file_paths = [
    "Al Jazeera_articles.csv",
    "BBC_articles.csv",
    "CNN_articles.csv",
    "New York Times_articles.csv",
    "The Guardian_articles.csv",
]
```

The script calls `nltk.download('stopwords')` to obtain the English stop-word corpus. This step requires internet access unless the corpus is already available locally.

The preprocessing workflow performs the following operations for each CSV:

1. Load the CSV into a pandas DataFrame.
2. Remove duplicate rows.
3. Drop rows where `description` is missing.
4. Add a randomly generated placeholder label (`Category 1` or `Category 2`).
5. Lowercase the descriptions, remove punctuation, remove English stop words, and stem tokens using Porter Stemmer.
6. Transform the descriptions into TF-IDF features.
7. Apply SMOTE to the vectorized features and generated labels.
8. Display a label-count plot and a word cloud.
9. Print a message indicating that preprocessing is complete.

The current script is an exploratory preprocessing workflow. It does not train or evaluate a news-classification model.

## How the Code Works

### RSS collection

The scraper uses `requests.get()` to retrieve a feed and `feedparser.parse()` to read its entries. It saves a small set of metadata fields rather than downloading the article pages themselves. Threads allow the five publisher collection tasks to run concurrently.

### Text cleaning

The `preprocess_text()` function:

- converts text to lowercase;
- replaces punctuation with spaces;
- removes English stop words;
- applies Porter stemming.

The `clean_data()` function removes exact duplicate rows and drops rows with missing descriptions before preprocessing.

### TF-IDF and SMOTE

The script converts processed descriptions to TF-IDF vectors. It then calls SMOTE to resample those vectors using the generated labels. Because the labels are random placeholders rather than genuine news topics, the resampling output should not be interpreted as a meaningful balanced news-classification dataset.

### Exploratory visualizations

`perform_eda()` prints summary statistics and creates a count plot for the generated labels and a word cloud from the processed descriptions. These plots help inspect the data pipeline, but the random labels do not reveal real publisher categories or article topics.

## Important Caveats

- **No actual topic labels:** The preprocessing code generates labels randomly. It does not classify articles into real news categories.
- **No trained classifier:** Although `train_test_split` is imported, the current script does not use it to train or evaluate a predictive model.
- **SMOTE is not meaningful with random labels:** Resampling artificial labels does not create reliable topic categories and should not be presented as a model result.
- **Scraping is RSS-based:** The scraper reads RSS entries and does not extract full article bodies from publisher webpages.
- **The 500-entry target is not guaranteed:** The script stops at the time limit if a source is slow or repeatedly returns limited entries.
- **Repeated feed entries may occur:** The scraper does not deduplicate entries during collection. Duplicate removal happens later in the preprocessing function, and only exact duplicate rows are removed.
- **Network robustness could improve:** The scraper does not set explicit request timeouts or implement retries and backoff.
- **Potentially missing fields:** The scraper accesses `article.title` and `article.link` directly, so entries missing those fields may trigger an exception. The exception handler records an error, then saves whatever has been collected.
- **Thread interruption:** The KeyboardInterrupt handler waits for threads to finish; it does not actively cancel in-progress requests.
- **Reproducibility:** Random labels are not seeded, so the generated labels change across runs.
- **No dependency versions:** No pinned environment or requirements file is included.
- **Publisher terms and access:** RSS availability and permitted uses vary by publisher. Check the relevant terms before collecting or redistributing data.

## Limitations and Future Improvements

1. Add a `requirements.txt` with tested package versions.
2. Add request timeouts, retries, backoff, and clearer error reporting.
3. Deduplicate feed entries using article URLs during collection.
4. Normalize timestamps and record the source publisher as a dedicated column.
5. Validate required RSS fields and handle missing titles, links, and descriptions safely.
6. Make the RSS URLs, output directory, article limit, and time limit configurable.
7. Replace random labels with genuine labels from a documented taxonomy if news classification is a project goal.
8. Split the data into training, validation, and test sets before fitting any text vectorizer or resampling method.
9. Fit TF-IDF only on training data and apply resampling to training data only, to avoid evaluation leakage.
10. Add meaningful model training, baseline comparisons, and evaluation metrics if predictive classification is intended.
11. Add tests for parsing, preprocessing, missing values, duplicates, and CSV output.
12. Respect publisher terms, request rates, and robots or feed usage guidance where applicable.

## Responsible Use

This repository is an educational data-collection and text-preprocessing project. Before using the collected content in a public dataset or commercial application, review each publisher's terms, copyright conditions, and restrictions on redistribution. The included article metadata should not be assumed to grant rights to republish full articles.

## License

No license file was included in the inspected repository. Unless a license is added, permissions to reuse, modify, and redistribute the code may be restricted.
