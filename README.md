test mpscfifo2
===

Test Dmitry Vyukov's [non-intrusive mpsc fifo](http://www.1024cores.net/home/lock-free-algorithms/queues/non-intrusive-mpsc-node-based-queue).

A second implementation of Dimtry Vyukov's [non-intrusive mpsc fifo](http://www.1024cores.net/home/lock-free-algorithms/queues/non-intrusive-mpsc-node-based-queue).
This first implementation I did is [here](https://github.com/winksaville/test-mpscfifo)
this one differs by having a Cell_t and allocating the stub in the fifo
and having another one in the Msg_t. So as with my previous implementation
it actually is intrusive because the infomation is in the Msg_t.

This algorithm currently doesn't completely clean up after itself but
for the time being its OK. The thing I like this algorithm is that I
don't have to manage the Cell_t separately. But it appears there is
a fairly significant performance penalty.

For the simple application I see a slow down of a 13.6ns per msg
compared to 10.2ns for about a 36% slower:
```
wink@wink-desktop:~/prgs/test-mpscfifo2 (master)
$ time ./simple 800000000
test loops=800000000
     1 7f700fbed700  simple:+
     2 7f700fbed700  simple: init cmdFifo=0x7ffdb35089c0
     3 7f700fbed700  simple: remove from empty cmdFifo=0x7ffdb35089c0
     4 7f700fbed700  simple: add a message to empty cmdFifo=0x7ffdb35089c0
     5 7f700fbed700  simple: remove from with one item in cmdFifo=0x7ffdb35089c0
     6 7f700fbed700  simple: add a message to empty cmdFifo=0x7ffdb35089c0
     7 7f700fbed700  simple: add a message to non-empty cmdFifo=0x7ffdb35089c0
     8 7f700fbed700  simple: remove msg1 from cmdFifo=0x7ffdb35089c0
     9 7f700fbed700  simple: remove msg2 from cmdFifo=0x7ffdb35089c0
    10 7f700fbed700  simple: remove from empty cmdFifo=0x7ffdb35089c0
    11 7f700fbed700  simple:-error=0

    12 7f700fbed700  perf:+loops=800000000
    13 7f700fbed700  perf: init cmdFifo=0x7ffdb3508980
    14 7f700fbed700  perf: remove from empty cmdFifo=0x7ffdb3508980
    15 7f700fbed700  perf: add_rmv from empty fifo  processing=10.993s
    16 7f700fbed700  perf: add rmv from empty fifo ops_per_sec=72771314.456
    17 7f700fbed700  perf: add rmv from empty fifo   ns_per_op=13.7ns
    18 7f700fbed700  perf: add_rmv from non-empty fifo  processing=10.861s
    19 7f700fbed700  perf: add rmv from non-empty fifo ops_per_sec=73655539.511
    20 7f700fbed700  perf: add rmv from non-empty fifo   ns_per_op=13.6ns
    21 7f700fbed700  perf:-error=0

Success

real    0m21.856s
user    0m21.853s
sys     0m0.000s


wink@wink-desktop:~/prgs/test-mpscfifo (master)
$ time ./simple 800000000
test loops=800000000
     1 7f8d59d95700  simple:+
     2 7f8d59d95700  simple: init pool=0x7ffec9b9a580
     3 7f8d59d95700  simple: init cmdFifo=0x7ffec9b9a640
     4 7f8d59d95700  simple: remove from empty cmdFifo=0x7ffec9b9a640
     5 7f8d59d95700  simple: add a message to empty cmdFifo=0x7ffec9b9a640
     6 7f8d59d95700  simple: remove from with one item in cmdFifo=0x7ffec9b9a640
     7 7f8d59d95700  simple: add a message to empty cmdFifo=0x7ffec9b9a640
     8 7f8d59d95700  simple: add a message to non-empty cmdFifo=0x7ffec9b9a640
     9 7f8d59d95700  simple: remove msg1 from cmdFifo=0x7ffec9b9a640
    10 7f8d59d95700  simple: remove msg2 from cmdFifo=0x7ffec9b9a640
    11 7f8d59d95700  simple: remove from empty cmdFifo=0x7ffec9b9a640
    12 7f8d59d95700  simple:-error=0

    13 7f8d59d95700  perf:+loops=800000000
    14 7f8d59d95700  perf: add_rmv from empty fifo  processing=8.742s
    15 7f8d59d95700  perf: add rmv from empty fifo ops_per_sec=91514036.628
    16 7f8d59d95700  perf: add rmv from empty fifo   ns_per_op=10.9ns
    17 7f8d59d95700  perf: add_rmv from non-empty fifo  processing=8.194s
    18 7f8d59d95700  perf: add rmv from non-empty fifo ops_per_sec=97633309.223
    19 7f8d59d95700  perf: add rmv from non-empty fifo   ns_per_op=10.2ns
    20 7f8d59d95700  perf:-error=0

Success

real	0m16.937s
user	0m16.937s
sys	0m0.000s
```

Add the hit is greater for the stress full 'test' application with
28.0ns vs 18.6ns or 50% slow down:
```
wink@wink-desktop:~/prgs/test-mpscfifo2 (master)
$ time ./test 12 10000000 0x100000
test client_count=12 loops=10000000 msg_count=1048576
     1 7fee6e12b700  multi_thread_msg:+client_count=12 loops=10000000 msg_count=1048576
     2 7fee6e12b700  multi_thread_msg: cmds_processed=722018502 msgs_processed=1457668660 mt_msgs_sent=62081318 mt_no_msgs=57918682
     3 7fee6e12b700  startup=0.647227
     4 7fee6e12b700  looping=33.658571
     5 7fee6e12b700  disconnecting=6.882786
     6 7fee6e12b700  stopping=0.017140
     7 7fee6e12b700  complete=0.238081
     8 7fee6e12b700  processing=40.797s
     9 7fee6e12b700  cmds_per_sec=17698016.380
    10 7fee6e12b700  ns_per_cmd=56.5ns
    11 7fee6e12b700  msgs_per_sec=35730170.002
    12 7fee6e12b700  ns_per_msg=28.0ns
    13 7fee6e12b700  total=41.444
    14 7fee6e12b700  multi_thread_msg:-error=0

Success

real	0m41.473s
user	6m46.037s
sys	0m38.503s


wink@wink-desktop:~/prgs/test-mpscfifo (master)
$ time ./test 12 10000000 0x100000
test client_count=12 loops=10000000 msg_count=1048576
     1 7f11d5248700  multi_thread_msg:+client_count=12 loops=10000000 msg_count=1048576
     2 7f11d5248700  multi_thread_msg: cmds_processed=1013196688 msgs_processed=2040025045 mt_msgs_sent=87016580 mt_no_msgs=32983420
     3 7f11d5248700  startup=0.440037
     4 7f11d5248700  looping=29.840819
     5 7f11d5248700  disconnecting=7.841160
     6 7f11d5248700  stopping=0.019422
     7 7f11d5248700  complete=0.206526
     8 7f11d5248700  processing=37.908s
     9 7f11d5248700  cmds_per_sec=26727831.902
    10 7f11d5248700  ns_per_cmd=37.4ns
    11 7f11d5248700  msgs_per_sec=53815263.240
    12 7f11d5248700  ns_per_msg=18.6ns
    13 7f11d5248700  total=38.348
    14 7f11d5248700  multi_thread_msg:-error=0

Success

real	0m38.350s
user	6m13.510s
sys	0m39.570s
```
