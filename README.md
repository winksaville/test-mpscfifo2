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

But the hit is actually less for the stressful 'test' application with
18.1ns vs 15.2ns or 20% slow down:
```
wink@wink-desktop:~/prgs/test-mpscfifo2 (master)
$ time ./test 12 10000000 0x100000
test client_count=12 loops=10000000 msg_count=1048576
     1 7f8713559700  multi_thread_msg:+client_count=12 loops=10000000 msg_count=1048576
     2 7f8713559700  multi_thread_msg: cmds_processed=897584156 msgs_processed=1808799968 mt_msgs_sent=76870417 mt_no_msgs=43129583
     3 7f8713559700  startup=0.428069
     4 7f8713559700  looping=31.454950
     5 7f8713559700  disconnecting=1.134321
     6 7f8713559700  stopping=0.035123
     7 7f8713559700  complete=0.194830
     8 7f8713559700  processing=32.819s
     9 7f8713559700  cmds_per_sec=27349341.814
    10 7f8713559700  ns_per_cmd=36.6ns
    11 7f8713559700  msgs_per_sec=55114039.467
    12 7f8713559700  ns_per_msg=18.1ns
    13 7f8713559700  total=33.247
    14 7f8713559700  multi_thread_msg:-error=0

Success

real	0m33.251s
user	6m7.940s
sys	0m21.810s

wink@wink-desktop:~/prgs/test-mpscfifo (master)
$ time ./test 12 10000000 0x100000
test client_count=12 loops=10000000 msg_count=1048576
     1 7f8a263d3700  multi_thread_msg:+client_count=12 loops=10000000 msg_count=1048576
     2 7f8a263d3700  multi_thread_msg: cmds_processed=1010090576 msgs_processed=2033812821 mt_msgs_sent=86202983 mt_no_msgs=33797017
     3 7f8a263d3700  startup=0.474970
     4 7f8a263d3700  looping=29.276984
     5 7f8a263d3700  disconnecting=1.358415
     6 7f8a263d3700  stopping=0.063303
     7 7f8a263d3700  complete=0.165955
     8 7f8a263d3700  processing=30.865s
     9 7f8a263d3700  cmds_per_sec=32726447.852
    10 7f8a263d3700  ns_per_cmd=30.6ns
    11 7f8a263d3700  msgs_per_sec=65894555.210
    12 7f8a263d3700  ns_per_msg=15.2ns
    13 7f8a263d3700  total=31.340
    14 7f8a263d3700  multi_thread_msg:-error=0

Success

real	0m31.341s
user	5m46.070s
sys	0m20.500s
```
