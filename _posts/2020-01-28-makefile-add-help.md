---
layout: post
title: The Self-Documenting Makefile
date: 2020-01-28
categories: [reproducibility, makefiles, easydata]
excerpt: How to include a self-documenting `show-help` target in every `Makefile` you create, and make it your default rule.
---

**TL;DR** : _How to include a self-documenting `show-help`
target in every `Makefile` you create, and make it your default rule._

Here's a trick that we borrowed from [cookiecutter-datascience] (who
borrowed it from [Marmelab]). It's such a good one that it has
become a fixture in every `Makefile` we create:

The default rule (the one invoked if you just type `make`) is
`show-help`.  This target scans the `Makefile` itself and extracts
comments prefaced by two hashes. These are assumed to be help messages
for the target that begins on the next line.

Here's the magic in action, on our familiar [bus_number] repo:
~~~
$ make
To get started:
  >>> make create_environment
  >>> conda activate bus_number

Project Variables:
PROJECT_NAME = bus_number

Available rules:
clean               Delete all compiled Python files 
create_environment  Set up python interpreter environment 
requirements        Install or update Python Dependencies 
test_environment    Test python environment is set-up correctly 

~~~

How does it work? For that we need to delve into some command-line
magic involving `sed`.


~~~
.DEFAULT_GOAL := show-help

show-help:
	@echo "$$(tput bold)Available rules:$$(tput sgr0)"
	@sed -n -e "/^## / { \
		h; \
		s/.*//; \
		:doc" \
		-e "H; \
		n; \
		s/^## //; \
		t doc" \
		-e "s/:.*//; \
		G; \
		s/\\n## /---/; \
		s/\\n/ /g; \
		p; \
	}" ${MAKEFILE_LIST} \
	| LC_ALL='C' sort --ignore-case \
	| awk -F '---' \
		-v ncol=$$(tput cols) \
		-v indent=19 \
		-v col_on="$$(tput setaf 6)" \
		-v col_off="$$(tput sgr0)" \
	'{ \
		printf "%s%*s%s ", col_on, -indent, $$1, col_off; \
		n = split($$2, words, " "); \
		line_length = ncol - indent; \
		for (i = 1; i <= n; i++) { \
			line_length -= length(words[i]) + 1; \
			if (line_length <= 0) { \
				line_length = ncol - indent - length(words[i]) - 1; \
				printf "\n%*s ", -indent, " "; \
			} \
			printf "%s ", words[i]; \
		} \
		printf "\n"; \
	}' \
	| more $(shell test $(shell uname) = Darwin && echo '--no-init --raw-control-chars')
~~~

Here's that `sed` script explained

~~~
 /^##/:
 	* save line in hold space
 	* purge line
 	* Loop:
 		* append newline + line to hold space
 		* go to next line
 		* if line starts with doc comment, strip comment character off and loop
 	* remove target prerequisites
 	* append hold space (+ newline) to line
 	* replace newline plus comments by `---`
 	* print line
~~~

In practice, we've added a little more to that message, including some
instructions for creating the environment, and a the list of `Makefile`
variables that you specify in the `HELP_VARS` list.

~~~
.DEFAULT_GOAL := show-help

HELP_VARS := PROJECT_NAME

print-%  : ; @echo $* = $($*)

help-prefix:
	@echo "To get started:"
	@echo "  >>> $$(tput bold)make create_environment$$(tput sgr0)"
	@echo "  >>> $$(tput bold)conda activate $(PROJECT_NAME)$$(tput sgr0)"
	@echo
	@echo "$$(tput bold)Project Variables:$$(tput sgr0)"

show-help: help-prefix $(addprefix print-, $(HELP_VARS))
	@echo
	@echo "$$(tput bold)Available rules:$$(tput sgr0)"
	...
~~~

That's the magic. Cut and paste this snippet. Use it everywhere you
can. Or even better, check out [cookiecutter-easydata], and just use
that for all your reproducible data science needs.

### More Reading
Want some other takes on using makefiles effectively? We don't necessary agree, but we're thinking about what the authors have to say:
* [all-purpose]
* [opinionated-makefiles]

[all-purpose] https://blog.mindlessness.life/2019/11/17/the-language-agnostic-all-purpose-incredible-makefile.html "The Language Agnostic, All Purpose, Incredible Makefile"
[opinionated-makefiles]: https://tech.davis-hansson.com/p/make/ "An opinionated approach to (GNU) make"
[marmelab]: http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
[cookiecutter-datascience]: https://drivendata.github.io/cookiecutter-data-science/
[cookiecutter-easydata]: https://github.com/hackalog/cookiecutter-easydata

