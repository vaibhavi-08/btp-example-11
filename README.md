# sample_repo3

A simple Python calculator module.

## Structure

```
sample_repo3/
├── src/
│   └── calculator.py
├── tests/
│   ├── __init__.py
│   └── test_calculator.py
├── requirements.txt
├── setup.py
├── Jenkinsfile
└── .black
```

## Install

```bash
pip install -e .
```

## Run tests

```bash
python -m unittest discover -s tests -p "test*.py" -v
```
