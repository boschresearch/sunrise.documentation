# Guide to SUNRISE
[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

This repository provides the official documentation and specifications for **SUNRISE**, the _**S**calable **Un**ified **R**ESTful **I**nfrastructure for **S**ystem **E**valuation_.

[**:link: Hosted on GitHub Pages**](https://boschresearch.github.io/sunrise.documentation/)


## How to Use
The documentation is built using [MkDocs](https://www.mkdocs.org).
- The content is written in Markdown files, which are stored in the [docs/](docs/) directory.
- The structure and settings are configured in the [`mkdocs.yml`](mkdocs.yml) file.

### Manual Testing
For editing and local preview, **MkDocs** can be installed as a Python `pip` package.
All commands must be called from the root of this repository.

**Installation:**
```sh
# (Optional) Create and activate a Python Virtual Environment
python -m venv venv
source venv/bin/activate

# Install the required Python packages
pip install -r requirements.txt
```

**Build and Preview:**
```sh
# Start a local web-server to preview the docs
mkdocs serve

# Or: Just generate the static HTML files into the 'site/' directory
mkdocs build
```

### Automated Deployment and Hosting
The included [deployment workflow](.github/workflows/deploy.yml) automates the process of building and publishing the documentation as a web page.
The workflow automatically generates html from the `master` branch and pushes it to the branch `gh-pages` which is used to host the web-page with GitHub Pages.


## Contributing
Questions and requests can be raised via **GitHub issues**.
Please see the [contribution guide](CONTRIBUTING.md) for further information on how to get involved.


## License
```
SUNRISE Documentation © 2025 by Robert Bosch GmbH is licensed under CC BY-SA 4.0.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/4.0/
```


## Citing
Cite this work as defined in the included [citation file](CITATION.cff).


## Acknowledgments
This work was initiated as a research project by [Robert Bosch GmbH](https://www.bosch.com/research/).
