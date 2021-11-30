
== Test   alarmtest: test2 ==
alarmtest: test2: FAIL
...
test2 start
...alarm!
alarm!
test2 failed: alarm handler called more than once
$ qemu-system-riscv64: terminating on signal 15 from pid 24830 (<unknown process>)
MISSING '^\.?test2 passed$'




== Test   alarmtest: test1 ==
alarmtest: test1: FAIL
...
test1 failed: too few calls to the handler
test2 start
.....alarm!
test2 passed
$ qemu-system-riscv64: terminating on signal 15 from pid 25655 (<unknown process>)
MISSING '^\.?test1 passed$'
