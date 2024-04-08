# Documentation 

## **Possible Candidates Explored**

- **Bookstack**
	- About: CMS
	- [Bookstack Home Page](https://www.bookstackapp.com/): 
	- [GitHub Repo](https://github.com/BookStackApp/BookStack)
	- About: More like wiki media in Markdown.
	- Tech Stack: MySQL/PHP/Laravel-based
	- Pros: Open source, self-hosted, robust, markdown editor
	- Cons: **no git sync integration**, 

- **Grav**
	- About: CMS
	- [Grav Home Page](https://getgrav.org/) 
	- [GitHub Repo](https://github.com/getgrav/grav) 
	- Tech stack: PHP-based
	- Pros: Open source, self-hosted, flat-file based, markdown editor, **git sync might be possible with efforts**
 
- **GitBook**
	- About: CMS
	- [Gitbook Homep Page](https://www.gitbook.com/) 
	- Pros: Git Sync, Markdown supported
	- Cons: Commercial,  not self-hosted, subscription-based

- **Notable**
	- About: Desktop note-taking app
	- [Notable Home Page](https://notable.app/)
	- [GitHub Repo](https://github.com/notable) 
	- Pros: Desktop app, self git sync, Markdown-based, self-hosted
	- Cons: Earlier versions open source, web support in dev

- **Sphinx**
	- About: Documentation generator
	- [Sphinx Home Page](https://www.sphinx-doc.org)
	- [Github Repo](https://github.com/sphinx-doc/sphinx) 
	- Tech stack: Python-based
	- Pros: Documentation generator, open-source, self git sync, supports Markdown, popular among Python and ReadTheDoc community, can be installed as python package. 

**Summary:** For now go with Sphinx with self-hosting. For the future, move to more robust CMS like Bookstack.


### How to generate documentation using Sphinx

- If not already installed, install `sphinx` using Python package installer `pip` using the following command:
```
pip install -U sphinx
```
- If not already installed, install `sphinx-autobuild`
```
pip install sphinx sphinx-autobuild
```
- We are going to use ReadTheDocs theme for the html file. Install the theme using the following command. Read [here](https://sphinx-rtd-theme.readthedocs.io/en/stable/) for details.
```
pip install sphinx_rtd_theme
```
- We are going to use `myst-parser` to generate html documentation from Markdown. See [here](https://github.com/executablebooks/MyST-Parser) for details. Install it using:
```
pip install myst-parser
```

- Initialize the documentation configuration from the top level directory using `sphinx-autobuild`.

- Add these at the end of the existing conf.py file to connect to the Markdown parser and ReadTheDocs theme installed:
```
extensions = ["myst_parser", "sphinx_rtd_theme"]
source_suffix = ['.rst', '.md']
html_theme = "sphinx_rtd_theme"
```

- Generate the html documentation: `make html`.
