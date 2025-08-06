---
title: Using Makefiles, SOPS, and virtualenv Together for Elegant Python Environments
description: Here's a very simple architecture that might become the boilerplate for all your future python projects
tags: ["python", "sops", "makefile"]
categories: ["Engineering and Development"]
date: 2025-08-06
author: "Rick Pfahl"
draft: false
image: "/images/blog/sops.png"
---

I've been managing my secrets with `sops` ever since I looked into the subject last month, and I've been using Makefiles to handle bringing up my docker environments as they provide a nice way to not only shorten the commands I need to type, but they provide a means of self-documenting that portion of the process.

But I realized, even in some of my simpler projects that are just running in a virtualenv, and have a secret or two that shouldn't be stored in the repository can benefit from the use of Makefiles. Especially since it feels like a lot of overhead for simpler projects to implement the entire `sops` secret encryption process. Here's what I'm using and it takes about 2 minutes to setup.

The special trick to this design is the use of the `.ONESHELL:` directive in your *Makefile* as normally sourcing `bin/activate` from within a script does not effect the user environment. The .ONESHELL: directive tells make to execute all lines in a recipe within a single subshell. This allows the virtual environment activation to persist for all commands in that recipe.

Anyway, first thing to do is make sure you provide a `.sops.yaml` file to tell `sops` which pgp key to use for encryption

```yaml
# .sops.yaml
creation_rules:
  - path_regex: ./*
    pgp: '<your key fingerprint here>'
```

Next create your `env.json` with your secrets, the keys will correspond to the names of the environment variables they will be set

```json
// env.json
{
  "A_SECRET": "------[REDACTED VALUE]------",
  "ANOTHER_SECRET": "------[REDACTED VALUE]------",
  "YET_ANOTHER_SECRET": "------[REDACTED VALUE]------"
}
```

And finally create your Makefile

```
.ONESHELL:

VENV_PATH = venv
PREFERRED_SHELL = /usr/bin/zsh

encrypt-secrets:
	@echo "encrypting secrets"
	sops -e env.json > env.json.enc
	mv env.json.enc env.json

run-env:
	@echo "loading the environment"
	source $(VENV_PATH)/bin/activate
	sops exec-env env.json $(PREFERRED_SHELL)	
```

And you can verify everything worked by running


```bash

make encrypt-secrets

make run-env

printenv | grep A_SECRET
# prints your secret

which python
# prints a path inside your virtual environment

exit

printenv | grep A_SECRET 
# doesn't print your secret 

which python
# prints /usr/bin/python or something similar

make run-env

printenv | grep A_SECRET 
#and again, your environment specific settings

which python
#should appear 

```


And there you have it. 

Simple and efficient management of your entire project. 






