### Example project to replicate the issue with ignored folder when building.

When using `balena push` to deploy the code, some services fail to build due to the fact
that they contain a subservice folder which itself contains a `.gitignore` file to exclude the build outputs of `make`.
Because the name of the output file matches the name of the sub-folder, the sub-folder gets excluded from the build context.

If a version before 8.0.0 is used, the build succeeds.


If instead of `balena push` we use `git push`, the build fails for service1 (which has a submodule),
with the following error: `make: *** No targets specified and no makefile found. Stop.`. This is due the
fact the submodules are not initialized on remote servers, which is an expected behaviour.
The image for service2 (which has no submodule, but only a `.gitignore` in a sub-folder) is built successfully.


### Run the example

Steps:
  1. Clone the repo and change the directory
  2. Run `git submodule init && git submodule update`
  3. Run `balena push <app_name>`


### Expected results (with later versions of the CLI):
  - The files `services/service1/subservice1/*.o`, `services/service1/subservice1/subservice1` should be excluded
    for `test_submodule` docker service
  - The files `services/service2/subservice2/*.o`, `services/service2/subservice2/subservice2` should be excluded
    for `test_service2` docker service
  - The paths in `.gitignore` files (perhaps even .dockerignore, didn't test though) should be relative to the
    folder the file is located at
  - The `services/service3/subservice2/.dockerignore` should exclude the `*.o` file from build context
    for `test_service3` docker service (maybe?)

  - The `.gitignore` files are not having effect cross-services


### Actual results (with later versions of the CLI):
  - The folder `services/service1/subservice1` is excluded for `test_submodule` docker service (bad)
  - The folder `services/service2/subservice2` is excluded for `test_service2` docker service (bad)
  - The files `services/service3/subservice2/*.o` are included in the built context for `test_service3`
    docker service (bad)

  - The `.gitignore` files are not having effect cross-services (good)
