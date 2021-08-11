# Create a Docker image via CMake
This project creates a docker image using CMake. The image
holds an executable that was also built by the same CMake
build system. It correctly models various file dependencies
so that only out-of-date items are rebuilt.

Here's the utilitarian user's guide:
```bash
# Clone and build.
mkdir build
cd build
cmake ..
make

# Run the docker image that was just built.
docker run --rm cmake-docker-example

# Run make again. Nothing should be rebuilt.
make

# Touch the main.c file, then run make again.
# Notice how both the executable and the dockerfile
# are rebuilt.
touch ../main.c
make
```

## ~~Best~~ Good practices
This project follows good practices by correctly modeling
the various dependencies between the docker image and
its source files (i.e. the executable target and Dockerfile).

## Bad practices
This project yanks an executable out of the build tree
and dumps it into a Docker image. The executable has the
wrong `RPATH` and may not be setup for the image's operating
system (e.g. building on Ubuntu but creating a CentOS image).

A better option would be to use CPack to create a "shippable"
binary, then inject that package into the image. I've had
good success doing this with TGZs and RPMs. Maybe someday
I'll extend this example to demonstrate that...
