## What is Multi Stage Dockerfile?
Multi-stage builds are a Docker feature that allows you to use multiple FROM statements in a single Dockerfile to create smaller, cleaner, and more secure images. 

__Without multi-stage builds:__

* You often end up with large images containing build tools, dependencies, or temporary files that are no longer needed after your app is built.

__With multi-stage builds:__

* You can use one stage to build your application, and another stage to copy only the necessary files into a clean, minimal runtime image.

## Benefits:
* __Smaller image size__ – because build tools aren’t copied to the final image.

* __Better security__ – fewer attack surfaces in your runtime image. (More packages = more risk)

* __Cleaner structure__ – clear separation of build and runtime steps.


## Why GoLang?
The main purpose of choosing a golang based applciation to demostrate this example is golang is a statically-typed programming language that does not require a runtime in the traditional sense. Unlike dynamically-typed languages like Python, Ruby, and JavaScript, which rely on a runtime environment to execute their code, Go compiles directly to machine code, which can then be executed directly by the operating system.

So the real advantage of multi stage docker build and distro less images can be understand with a drastic decrease in the Image size.

## Project Structure
The application used is a simple calculator in golang.

There are two dockerfiles here for comparison. One as a regualar dockerfile and the other as a Multi Stage dockerfile.

Both of them will be built seperately to create two images. Then the image size will be compared.

## 1. dockerfile-without-multistage
```
FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

ENTRYPOINT ["/app"]
```
> Uses the official Ubuntu image as the base.

> Updates the package list and installs Go compiler.

> Sets an environment variable to disable Go modules (legacy builds).

> Copies all files from the build context (local directory) into the container's working directory.

> Builds the Go app, disables C bindings (CGO_ENABLED=0) for portability, and outputs the binary as /app.

> Sets the compiled binary as the entry point. When the container runs, it executes /app.
## Result
The final image contains all build tools (like Golang and Ubuntu libraries), making it larger and potentially insecure.
## 2. dockerfile-multistage
```
###########################################
# BASE IMAGE - STAGE 1
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

############################################
# STAGE 2
############################################

FROM scratch

# Copy the compiled binary from the build stage
COPY --from=build /app /app

# Set the entrypoint for the container to run the binary
ENTRYPOINT ["/app"]
```
> Stage 1 is same as the previous one. This stage contains executions only until building the binary.
>
> Stage 2 starts by using the scratch image: completely empty (no shell, no OS tools). It's the smallest possible distroless image. (scratch doesnt include go runtime hence why its so small. If this is for a python or java app, you'd need to use a python/ openjdk distroless image)

> Then copies just the compiled binary from the build stage to this new minimal image.

> Set the compiled Go app as the entry point for this tiny runtime image.

Create the docker images by naming them seperately.

![di](https://github.com/user-attachments/assets/a76bd21a-5744-4173-95e2-2120ffa7c6d8)


__calculator__ image size = 641MB

__multistage-calculator__ image size = 1.96MB

The result is a 99.7% reduction in image size — from 641 MB → 1.96 MB.

## Final Verdict
Using multi-stage builds:

* Improves efficiency

* Reduces storage and bandwidth costs

* Enhances security and speed

This is a best practice, especially for compiled languages like Go or Rust.

