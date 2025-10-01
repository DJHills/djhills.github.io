---
layout: post
title: "Python venv and the zsh prompt"
date: 2025-09-29 
categories: zsh python
---
Until recently, I've always used Python's [`virtualenv`](https://pypi.org/project/virtualenv/) and [`virtualenvwrapper`](https://pypi.org/project/virtualenvwrapper/) modules to manage Python virtualenvs.
Now that the built-in [`venv`](https://docs.python.org/3/library/venv.html) module is the preferred approach, last week I made the switch.
Here's an example usage:
```zsh
nuc% mkdir sample-python-project
nuc% python3 -m venv sample-python-project/.venv
nuc% source sample-python-project/.venv/bin/activate
(.venv) nuc%
```

The first thing that struck me about this was that I expect I'll always name my virtualenv `.venv` and place it within the project dir.
This differs from when working with `virtualenvwrapper`, where I would keep virtualenvs in a central location and give each a unique name.
As a result, following the switch to `venv`, the default inclusion of the virtualenv name in the shell prompt was no longer very helpful for me.

My solution was to replace the default behaviour with a shell function to show a warning if the active virtualenv is not as expected for the current directory.
Not only does this keep the prompt more succinct, it also makes it much more obvious when the activated virtualenv is not as expected.  

Behaviour is as follows:
1. If `.venv` subdir is present in current dir or any parent dir:
    1. Append `(venv not activated)` to prompt if no virtualenv is active.
    1. Append `(unexpected venv is activated)` to prompt if content of `VIRTUAL_ENV` variable does not match current dir.
1. If `.venv` subdir is not present in current dir or any parent dir, check that no virtualenv is active.  Append `(venv not deactivated)` to prompt if not.

The above was achieved with the below additions to my `.zshrc`:

```zsh
function venv_warning()
{
  venv_dir=$(pwd)
  while [[ "${venv_dir}" != "" && ! -e "${venv_dir}/.venv" ]]; do
    venv_dir=${venv_dir%/*}
  done

  if [[ "${venv_dir}" != "" && ! -v "VIRTUAL_ENV" ]]; then
    echo "(venv not activated)"
  elif [[ "${venv_dir}" == "" && -v "VIRTUAL_ENV" ]]; then
    echo "(venv not deactivated)"
  elif [[ "${venv_dir}" != "" ]] && [[ "${venv_dir}/.venv" != "${VIRTUAL_ENV}" ]]; then
    echo "(unexpected venv is activated)"
  fi
}

VIRTUAL_ENV_DISABLE_PROMPT=1  # Disable default venv prompt prefix
setopt prompt_subst
PROMPT="... \$(venv_warning) ..."
```

*Note that I've only shown the relevant portion of `PROMPT` above.*

Here it is in action:
```zsh
david@nuc [~/dev/python]
[16:44]: cd sample-python-project
david@nuc [~/dev/python/sample-python-project] (venv not activated)
[16:44]: source .venv/bin/activate
david@nuc [~/dev/python/sample-python-project]
[16:45]: source ../other-python-project/.venv/bin/activate
david@nuc [~/dev/python/sample-python-project] (unexpected venv is activated)
[16:45]: deactivate
david@nuc [~/dev/python/sample-python-project] (venv not activated)
[16:45]: source .venv/bin/activate
david@nuc [~/dev/python/sample-python-project]
[16:45]: cd ..
david@nuc [~/dev/python] (venv not deactivated)
[16:45]: deactivate
david@nuc [~/dev]
[16:45]:
```

At this point I coupled the above with another function to auto-activate and auto-deactivate virtualenv as I switched directory, in order to automate the invocation of `activate` and `deactivate`, as detailed in [this post]({% post_url 2025-10-01-auto-activating-python-venv %}).

