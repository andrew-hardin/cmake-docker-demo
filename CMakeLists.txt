project(cmake-docker)
find_program(DOCKER_CMD docker REQUIRED)

# Add a dirt simple executable. This is what will
# be embedded in the docker container.
add_executable(hello-world main.c)

# Setup a custom command to build the image. The
# command sets up a temporary working directory,
# copies the executable into it, then executes
# the docker build command.
#
# We communicate to CMake that the image needs to
# be rebuilt when the hello-world target rebuilds
# via the DEPENDS argument.
#
# One thing that's odd is that the output of the
# docker build command is an image in docker -- not
# a file artifact that CMake can then track. We
# take the hash of the image and store that on-disk.
# That then becomes the OUTPUT of the command that
# can be tracked in later steps.
set(IMAGE_NAME "cmake-docker-example")
set(IMAGE_HASH ${CMAKE_CURRENT_BINARY_DIR}/${IMAGE_NAME}.hash)
set(DOCKERFILE ${CMAKE_CURRENT_LIST_DIR}/Dockerfile)
set(_BUILD_CONTEXT context)
add_custom_command(
    OUTPUT ${IMAGE_HASH}
    COMMENT "Building docker image \"${IMAGE_NAME}\""

    # Create a fresh context with the executable.
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${_BUILD_CONTEXT}
    COMMAND ${CMAKE_COMMAND} -E make_directory ${_BUILD_CONTEXT}
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:hello-world> ${_BUILD_CONTEXT}

    # Launch the docker build command using the build context.
    COMMAND ${DOCKER_CMD} build 
        --tag ${IMAGE_NAME}
        --file ${DOCKERFILE}
        ${_BUILD_CONTEXT}

    # Cleanup the build context. Not strictly required, but it
    # cuts down on some of the clutter in the binary directory.
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${_BUILD_CONTEXT}

    # Capture the image's hash and store it as a tracked output.
    COMMAND ${DOCKER_CMD} images --no-trunc ${IMAGE_NAME} > ${IMAGE_HASH}

    DEPENDS
        ${DOCKERFILE}
        $<TARGET_FILE:hello-world>
)

# Define a target that depends on the image hash.
# Running "make" will trigger the custom
# command if it's out-of-date (e.g. the underlying
# executable has changed or the dockerfile was updated).
add_custom_target(docker_image ALL DEPENDS ${IMAGE_HASH})