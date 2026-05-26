# personal-website

MKDocs based personal website with СV

## Prepare environment

```
python3 -m venv ./venv
git remote -v
. ./venv/bin/activate
mkdocs build --strict --verbose
pip3 install -r ./requirements.txt
mkdocs build --strict --verbose
mkdocs serve
```
