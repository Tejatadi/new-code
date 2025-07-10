final deliverables:

Python Package (get_papers) with:

Typed, modular code

Paper-fetching and filtering logic

Command-line tool (get-papers-list) via poetry

CSV output (with optional console print)

Proper CLI flags: --help, --debug, --file

README.md

Pyproject with Poetry

TestPyPI-ready publishing setup (bonus)


get-papers/
├── get_papers/
│   ├── __init__.py
│   ├── fetch.py          # Handles API requests
│   ├── filter.py         # Filters non-academic authors
│   ├── utils.py          # Helper functions
│   └── cli.py            # CLI entry point
├── tests/
│   └── test_basic.py     # Sample test cases (optional)
├── pyproject.toml
├── README.md
└── .gitignore


__init__.py

fetch.py: Fetching PubMed data

filter.py: Identifying non-academic authors

utils.py: CSV writing and helpers

cli.py: Command-line entrypoint

pyproject.toml: Poetry config

README.md: Documentation
