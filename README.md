# git_presentation


## Installation

### python

Install pyenv for python local version management

#### Unix

```sh
curl -fsSL https://pyenv.run | bash
```

#### Windows

Use pyenv-win instead

```sh
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

Then using pyenv

```sh
pyenv install 3.10.7
pyenv local 3.10.7 # TO BE FIXED
```



```sh
python -m pip install notebook==6.5.5 # Issues with slide with newer versions.
python -m pip uninstall traitlets
python -m pip install traitlets==5.9.0

jupyter notebook
```

## Render 

```sh
jupyter nbconvert presentation.ipynb --to slides --post serve # for html output

jupyter nbconvert presentation.ipynb --to slides --post serve --TemplateExporter.exclude_input=True # Slides without python "pure" cell code.

jupyter nbconvert presentation.ipynb --to pdf # for pdf output
```