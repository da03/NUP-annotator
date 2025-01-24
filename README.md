# Conversation Project

A tool for annotating conversations using Label Studio. This project uses the open source version of [Label Studio](https://github.com/heartexlabs/label-studio) - many thanks to the Label Studio team for making this possible!

## Overview

This project facilitates consistent annotation across multiple annotators:
- Each annotator runs their own local instance of Label Studio
- All instances are configured with the same taxonomy structure
- Annotators work independently on their local installations
- Results can be exported and compared across annotators

The setup ensures everyone has identical project configuration while maintaining independent workspaces.

## Prerequisites

- Python 3.x
- `make` command-line tool
  - **Linux/Mac**: Usually pre-installed
  - **Windows**: Install via [chocolatey](https://chocolatey.org/): `choco install make`

## Quick Start

1. Clone the repository:
   ```bash
   git clone https://github.com/cedricwhitney/NUP-annotator.git
   cd NUP-annotator
   ```

2. Set up the environment:
   ```bash
   make setup
   ```

3. Add your data file:
   - Place your annotation tasks file in the `data/` directory
   - Supports both JSON and JSONL formats
   - File will be automatically converted and validated

4. Create a new project:
   ```bash
   make create-project
   ```
   This will:
   - Start Label Studio (if not running)
   - Guide you through getting your API key (first time only)
   - List available data files for you to choose from
   - Validate your chosen file's format
   - Convert JSONL to JSON if needed
   - Create a new project with your data and the correct taxonomy structure
   
   Note: If your file format is invalid, the script will exit with an error message. See "Required JSON Format" below.

5. Start Label Studio for regular use:
   ```bash
   make run
   ```
   - Visit http://localhost:8080
   - Log in to your account
   - Your projects will be available in your workspace

## Project Structure

The repository is organized as follows:

    conversation_project/          # This is a directory structure diagram
    ├── src/
    │   ├── core/                 # Core functionality
    │   │   ├── create_project.py     # Label Studio project setup
    │   │   └── label_studio_integration.py
    │   └── tools/                # Utility tools
    │       ├── csv_to_labelstudio.py     # Convert CSV to Label Studio format
    │       ├── convert_jsonl_to_json.py   # Convert JSONL to Label Studio format
    │       └── validate_labelstudio_json.py  # Validate JSON format
    ├── tests/
    │   └── tools/                # Tests for utility tools
    │       ├── test_csv_converter.py     # CSV conversion tests
    │       ├── test_json_validator.py    # JSON validation tests
    │       └── test_jsonl_converter.py   # JSONL conversion tests
    ├── data/                     # Data directory
    │   └── your_tasks.json       # Your annotation tasks in Label Studio format
    └── Makefile                  # Project automation

## Testing

The project includes tests to ensure data format compatibility with Label Studio.

### Running Tests

```bash
make test
```

### What's Tested

1. CSV Conversion (`test_csv_converter.py`)
   - Converts Turn 0-4 format to Label Studio JSON
   - Validates role assignment (human/llm)
   - Ensures proper JSON structure

2. JSON Validation (`test_json_validator.py`)
   - Validates Label Studio JSON format
   - Catches common format issues:
     - Missing "data" wrapper
     - Incorrect conversation structure
     - Invalid role/text fields

3. JSONL Conversion (`test_jsonl_converter.py`)
   - Converts JSONL to Label Studio JSON format
   - Adds required "data" wrapper if missing
   - Handles empty files correctly

### Example Valid Formats

CSV Input:
```csv
Turn 0,Turn 1,Turn 2,Turn 3,Turn 4
"Hello","Hi there","How are you?","I'm good","Great!"
```

JSON Output:
```json
[
  {
    "data": {
      "conversation": [
        {"role": "human", "text": "Hello"},
        {"role": "llm", "text": "Hi there"}
      ]
    }
  }
]
```

## Core Features
- Project creation and setup
- JSON validation
- Label Studio integration

## Optional Tools
- CSV to Label Studio JSON conversion (with tests in tests/tools/)
- JSON format fixing

## Data Tools

### File Format Support

The project supports both JSON and JSONL formats:
- JSON: Array of tasks in Label Studio format
- JSONL: One task per line in Label Studio format

When running `make create-project`:
1. All JSON/JSONL files in your data directory will be listed
2. You'll be prompted to choose which file to use
3. JSONL files are automatically converted to JSON
4. The file is validated for Label Studio compatibility

Example formats:
```jsonl
{"conversation": [{"role": "human", "text": "Hello"}, {"role": "assistant", "text": "Hi"}]}
{"conversation": [{"role": "human", "text": "How are you"}, {"role": "assistant", "text": "Good"}]}
```

or

```json
[
  {
    "data": {
      "conversation": [
        {"role": "human", "text": "Hello"},
        {"role": "assistant", "text": "Hi"}
      ]
    }
  }
]
```

### JSON Validation

Before importing tasks into Label Studio, you can validate and fix your JSON format:

```bash
# Just validate
make validate-json

# Validate and fix
python src/tools/validate_labelstudio_json.py input.json fixed_output.json
```

The validator will:
- Check JSON structure
- Attempt to fix common issues
- Provide detailed error messages
- Optionally save a fixed version

### CSV Conversion

For creating new datasets, you can convert CSV files to Label Studio format:

1. Edit the file paths in `src/tools/csv_to_labelstudio.py`:
   ```python
   # Input and output file paths - edit these to change your files
   csv_file = "data/your_input.csv"    # Your input CSV file
   json_file = "data/your_output.json" # Where to save the JSON
   ```

2. Run the conversion:
   ```bash
   make convert-csv
   ```

The script expects a CSV with columns "Turn 0" through "Turn 4":
```csv
Turn 0,Turn 1,Turn 2,Turn 3,Turn 4
"Hello","Hi there","How are you?","I'm good","Great!"
```

## Required JSON Format

Your JSON file must follow this structure for Label Studio:
```json
[
  {
    "data": {
      "conversation": [
        {
          "text": "Hello, how are you?",
          "role": "human"
        },
        {
          "text": "I'm doing well, thank you!",
          "role": "assistant"
        }
      ]
    }
  }
]
```

## Troubleshooting

If you encounter issues:
1. Ensure Python 3.x is installed: `python3 --version`
2. Check if Label Studio is running: http://localhost:8080
3. Try stopping and restarting Label Studio: `make stop-label-studio && make run`
4. If project is missing: Run `make create-project` to set up the project structure
5. If JSON import fails: Run `make validate-json` to check your data format
6. API key issues:
   - Your API key is specific to your Label Studio installation
   - Get a new key from Account & Settings > Access Token
   - Either enter it when prompted or set it as an environment variable

## Acknowledgments

This project uses [Label Studio](https://github.com/heartexlabs/label-studio), an open source data labeling tool. We're grateful to the Label Studio team for providing this excellent platform for annotation projects.

