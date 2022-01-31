add_executable(test0 test.cc)
KyDepsPackageTarget(test0)

add_test(NAME test0 COMMAND test0)
