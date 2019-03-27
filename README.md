### Example project to replicate the issue with ignored folder when building.

#### Folder structure:
  - `services/service1` - has a git submodule which has `.gitignore` file
  - `services/service2` - has a normal sub-folder which has `.gitignore` file
  - `services/service3` - has a normal sub-folder which has `.dockerignore` file


When using `balena push` to deploy the code, some services fail to build due to the fact
that they contain a subservice folder which itself contains a `.gitignore` file to exclude the build outputs of `make`.
Because the name of the output file matches the name of the sub-folder, the sub-folder gets excluded from the build context.

If a version before 8.0.0 is used, the build succeeds.


### Run the example

Steps:
  1. Clone the repo and change the directory
  2. Run `git submodule init && git submodule update`
  3. cd into each subservice folder, and run `make`
  4. Run `balena push <app_name>` or `git add . && git commit -m "test" && git push balena master`


#### Expected results (with later versions of the CLI):
  - The files `services/service1/subservice1/*.o`, `services/service1/subservice1/subservice1` should be excluded
    for `test_submodule` docker service
  - The files `services/service2/subservice2/*.o`, `services/service2/subservice2/subservice2` should be excluded
    for `test_service2` docker service
  - The paths in `.gitignore` files (perhaps even .dockerignore, didn't test though) should be relative to the
    folder the file is located at
  - The `services/service3/subservice2/.dockerignore` should exclude the `*.o` file from build context
    for `test_service3` docker service (maybe?)

  - The `.gitignore` files are not having effect cross-services


#### Actual results (with later versions of the CLI):
  - The folder `services/service1/subservice1` is excluded for `test_submodule` docker service (bad)
  - The folder `services/service2/subservice2` is excluded for `test_service2` docker service (bad)
  - The files `services/service3/subservice2/*.o` are included in the built context for `test_service3`
    docker service, and `make` prints "make: Nothing to be done for 'all'." (bad)

  - The `.gitignore` files are not having effect cross-services (good)


### Using `git push`

If instead of `balena push` we use `git push`, the build fails for `test_submodule` service,
which has a submodule, with the following error: `make: *** No targets specified and no makefile found. Stop.`.
This is due the fact the submodules are not initialized on remote servers, which is an expected behaviour.
The image for `test_service2` service , which has no submodule, but only a `.gitignore` in a sub-folder,
is built successfully.
The image for `test_service3` service , which has just a `.dockerignore` gets build, the output files `*.o`
are not sent with the build.

#### Expected results (with `git push`):
  - The build fails for `test_submodule` service with the following error: `make: *** No targets specified and no makefile found. Stop.`.
  - Build of `test_service2` succeeds
  - The `services/service3/subservice2/.dockerignore` should exclude the `*.o` file from build context
    for `test_service3` docker service (maybe?)


#### Actual results (with `git push`):
  - The build fails for `test_submodule` service with the following error: `make: *** No targets specified and no makefile found. Stop.`.
  - Build of `test_service2` succeeds
  - The files `services/service3/subservice2/*.o` are included in the built context for `test_service3`
    docker service, and `make` prints "make: Nothing to be done for 'all'." (bad)
