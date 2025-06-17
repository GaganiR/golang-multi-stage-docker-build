## What is Multi Stage Dockerfile?
Multi-stage builds are a Docker feature that allows you to use multiple FROM statements in a single Dockerfile to create smaller, cleaner, and more secure images. 
Without multi-stage builds:

* You often end up with large images containing build tools, dependencies, or temporary files that are no longer needed after your app is built.

With multi-stage builds:

* You can use one stage to build your application, and another stage to copy only the necessary files into a clean, minimal runtime image.
## Why GoLang?
The main purpose of choosing a golang based applciation to demostrate this example is golang is a statically-typed programming language that does not require a runtime in the traditional sense. Unlike dynamically-typed languages like Python, Ruby, and JavaScript, which rely on a runtime environment to execute their code, Go compiles directly to machine code, which can then be executed directly by the operating system.

So the real advantage of multi stage docker build and distro less images can be understand with a drastic decrease in the Image size.

