# git_presentation

This README will guide you on installation setup to run locally this presentation.

## Installation

### python

Install pyenv for python local version management

#### Unix

```sh
curl -fsSL https://pyenv.run | bash
```

Then add those lines in ```~/.bashrc```

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc
```

Some additional build parameters may need to be installed:
(see https://github.com/pyenv/pyenv/wiki#suggested-build-environment)
```
sudo apt update; sudo apt install make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
> Additionally, you should install libstd-dev if you are building Python 3.14 or newer but note that Ubuntu 20.04 does not include a sufficiently new version of this package to build the compression.zstd module.


#### Windows

Use pyenv-win instead

```sh
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

Then using pyenv

```sh
pyenv install 3.13.7
pyenv local 3.13.7
```

### Notebook

#### Basic installation
```sh
python -m pip install notebook
python -m jupyter notebook
```

#### Additional libraries for slides

```sh
python -m pip install RISE
python -m pip install jupyterlab_rise
```

## Render 

```sh
jupyter nbconvert presentation.ipynb --to slides --post serve # for html output

jupyter nbconvert presentation.ipynb --to pdf # for pdf output
```

#### Other

For signed commits, you can follow this link: https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key.

------------------------------------------------------------------------