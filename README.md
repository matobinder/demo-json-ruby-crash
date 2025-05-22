# json_crash
Quick repo made to demo a crash problem with JSON gem.

Something in this Hash causes Ruby itself to crash when calling `.to_json` on it
It works fine in version of json `< 2.12.0`. But with `2.12.0` it crashes.

I was originally using Ruby 3.3.7, but since it was crashing. I went to lastest version 3.4.4.

Assuming you have 3.4.4 installed, you should be able to reproduce this problem with this repo as follows

```bash
bundle install
./crasher.rb
```

NOTE: My run was on Ubuntu 22.04 (patched to lastest), with Ruby 3.4.4


When I run `crasher.rb` the output of this shows the following, it hangs, I have to kill -9 the process
```
./crasher.rb:5: [BUG] Segmentation fault at 0x0000000000000000
ruby 3.4.4 (2025-05-14 revision a38531fd3f) +PRISM [x86_64-linux]

-- Control frame information -----------------------------------------------
c:0003 p:---- s:0011 e:000010 CFUNC  :to_json
c:0002 p:16438 s:0007 E:0002b8 EVAL   ./crasher.rb:5 [FINISH]
c:0001 p:0000 s:0003 E:001130 DUMMY  [FINISH]

-- Ruby level backtrace information ----------------------------------------
./crasher.rb:5:in '<main>'
./crasher.rb:5:in 'to_json'

-- Threading information ---------------------------------------------------
Total ractor count: 1
Ruby thread count for this ractor: 1

-- Machine register context ------------------------------------------------
 RIP: 0x00007ec83b8a4790 RBP: 0x00000000001980bb RSP: 0x00007fff626ea2c0
 RAX: 0x0000640d3d2b1f20 RBX: 0x00007ec83ba1ac80 RCX: 0x00007ec83ba1b4b0
 RDX: 0x00007ec83ba1b4c0 RDI: 0x0000000000000002 RSI: 0x0000000000100000
  R8: 0x0000000000000000  R9: 0x0000000000000000 R10: 0x00007ec83ba1b410
 R11: 0x00007ec83ba1ace0 R12: 0xffffffffffffff30 R13: 0x00000000001980d0
 R14: 0x3633303030303030 R15: 0x000000000001980b EFL: 0x0000000000010202

-- C level backtrace information -------------------------------------------
```

When I was running on Ruby 3.3.7 on Ubuntu 22.04, the out put was as follows
```
corrupted double-linked list
Aborted (core dumped)
```

For fun, I tried a different OS via a Docker image which I had, which was a AL2023 with Ruby 3.3.7. Still crashed, but output was slightly different
``` 
corrupted size vs. prev_size
Aborted (core dumped)
```

