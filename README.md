TODO

Dockerize tracking ui

Where can we set the artifact storage ? We can explicitly set the tracking backend but what about the artifacts
Docker compose
Several dockerfiles with different Python envs => use inherited yaml files

Explore HTTP Server (REST API that allow not to depend on the package)
Explore MLProject / entrypoints
Explore MLModels / pyfunc / deployment/serving options / example where no dedicater flavor/wrapper(?), cf. prophet example

Dependency management: req.yaml, if absent only generated and logged when a model is logged ? Usefulness depends on 
how we want to rerun: rerun must include an env setup. Ok if managed through MLflow (?) 

Artifacts: log charts 

Code: best pratices (ex: use start_run context manager)/ code example: set tracking uri/list env variables that could be
relocated in a .bashrc. Concepts: name vs id/description/lifecycle.

Questions
What about tracking the training data ?
Test error estimation / CV refit
