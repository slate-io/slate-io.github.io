---
layout:     post
title:      Adding Swagger Specifications Validation to Your Travis CI Build
date:       2017-01-16 12:30:00
summary:    Tutorial on how to implement Swagger specification validation for Travis CI.
categories: slate Swagger CI
---

Swagger specifications are arguably the most compelling aspect of Swagger. It provides a clear, standard definition format on how to describe your RESTful APIs. If you haven't heard of Swagger, then we highly suggest heading over to [Swagger Specifications](http://swagger.io/specification/) to learn more.

For Slate, it's a powerful mechanism to define the behavior of any networking operating system. This provides a standardized format to allow users to define commands that can be executed remotely without modifying any of the core code in Slate; which ultimately bridges the gap between developers and network engineers.

However, this results in a multitude of Swagger specifications, which needs to be validated and managed, which can be time-consuming and error-prone. As a solution, the Slate team created the `swagger-spec-validator` which can be utilized by any continuous integration tools, such as Travis or Jenkins.

In this post, we will be demonstrating how to quickly configure a repository of Swagger specifications to be tested against Travis. Please note that this can be used with other continuous integrations tools, such as Jenkins.

First off, for `swagger-spec-validator` to locate Swagger specification files, your files must match the filename format *.swagger.json. Thus, your repository structure should resemble a similar structure below:

```
.
├── .travis.yml
└── specifications
    └── cats
        └── persian.swagger.json
    └── dogs
        └── pug.swagger.json
├── LICENSE
├── README.md
```

Afterward, we want to pass an environment variable in Travis to reference the application directory, which we’ll explain a little later.

```
env:
  - SPEC_PATH=$(pwd)
```

Next, we’ll need to inform your Travis build to access the [Docker](https://www.docker.com/) service.

```
services:
  - docker
```

Once Travis can utilize Docker, we set up Travis to pull the [slate/swagger-spec-validator]( https://hub.docker.com/r/slate/swagger-spec-validator/) from the hub. Now if you recall the environment variable before, we will pass this to our Docker instance to mount your project to be tested against.

```
before_install:
  - docker pull slate/swagger-spec-validator
  - docker run -it -e "SPEC_PATH=$SPEC_PATH" -v $SPEC_PATH:/data --name spec-validator slate/swagger-spec-validator:latest
```

Ideally, we want to run the `docker logs` command to inspect the validation of our specifications to provide additional details of our validation test.

```
script:
  - docker logs $(docker ps -lq)
```

This should yield: `The command "docker logs $(docker ps -lq)" exited with 0.`, and if your swagger specifications are valid this will cause your CI to continue to buzz along.

To piece it all together, your `.travis.yml` should be similar to the example below:

```
language: node_js
cache: yarn
sudo: required
env:
  - SPEC_PATH=$(pwd)
services:
  - docker
before_install:
  - docker pull slate/swagger-spec-validator
  - docker run -it -e "SPEC_PATH=$SPEC_PATH" -v $SPEC_PATH:/data --name spec-validator slate/swagger-spec-validator:latest
script: docker logs $(docker ps -lq)
```

We hope you find this repository ande post useful for any projects using Swagger. If you would like to see a working example of this, head to [Slate’s Swagger specifications](https://github.com/slate-io/specifications).
