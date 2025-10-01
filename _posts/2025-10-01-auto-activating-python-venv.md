---
layout: post
title: "Auto-activating Python venv"
date: 2025-10-01
categories: zsh python
---
When I switch into a Python project directory, I invariably want to also activate the associated venv.
As a matter of course, I will have created the virtualenv in a project subdir named `.venv`. 
In order to have that virtualenv activated automatically upon switching to the project directory, I added the following to my .zshrc:
```zsh
# Auto-activate venv if not already active and current dir contains one
precmd() {
  venv_dir=$(pwd)
  while [[ "${venv_dir}" != "" && ! -e "${venv_dir}/.venv" ]]; do
    venv_dir=${venv_dir%/*}
  done

  if [[ "${venv_dir}" == "" && -v "VIRTUAL_ENV" ]]; then
    deactivate
  fi

  if [[ "${venv_dir}" != "" && "${VIRTUAL_ENV}" != "${venv_dir}/.venv" ]]; then
    source ${venv_dir}/.venv/bin/activate
  fi
}
```

Here it is in action:
```zsh
david@nuc [~]
[11:47]: which python3
/usr/bin/python3
david@nuc [~]
[11:47]: cd dev/python/sample-python-project
david@nuc [~/dev/python/sample-python-project]
[11:47]: which python3
/home/david/dev/python/sample-python-project/.venv/bin/python3
david@nuc [~/dev/python/sample-python-project]
[11:47]: cd ~
david@nuc [~]
[11:48]: which python3
/usr/bin/python3
david@nuc [~]
[11:48]: cd dev/python/other-python-project/subdir
david@nuc [~/dev/python/other-python-project/subdir]
[11:48]: which python3
/home/david/dev/python/other-python-project/.venv/bin/python3
david@nuc [~/dev/python/other-python-project/subdir]
[11:48]: cd ~
david@nuc [~]
[11:48]: which python3
/usr/bin/python3
```

As you can see, the appropriate virtualenv gets activated and deactivated as I switch in and out of various project directories.

