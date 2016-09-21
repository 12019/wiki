
## Runnig unit tests

For debugging a test inside a debugger, use the --gtest_filter=<your_test_name> flag along with --single-process-tests.

```
$ out/Release_Wayland/ozone_unittests  --single-process-tests --gtest_filter=WaylandConnectionTest.Output
```
