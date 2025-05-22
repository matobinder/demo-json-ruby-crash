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

### Further debugging. 
So I also tried to grab the current release branch of JSON, build that up, and point my Gemfile at that. Get some better chances to look at the code and run under gdb to get a good stack trace and maybe some more info... looks like malloc is failing.

Here's my output from gdb.
```
(gdb) run crasher.rb 
Starting program: /home/myusername/.rbenv/versions/3.4.4/bin/ruby crasher.rb
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7fffdcfff640 (LWP 1863102)]

Thread 1 "ruby" received signal SIGSEGV, Segmentation fault.
_int_malloc (av=av@entry=0x7ffff761ac80 <main_arena>, bytes=bytes@entry=1671355) at ./malloc/malloc.c:4193
4193	./malloc/malloc.c: No such file or directory.
(gdb) where
#0  _int_malloc (av=av@entry=0x7ffff761ac80 <main_arena>, bytes=bytes@entry=1671355) at ./malloc/malloc.c:4193
#1  0x00007ffff74a5139 in __GI___libc_malloc (bytes=bytes@entry=1671355) at ./malloc/malloc.c:3329
#2  0x00007ffff796a21c in rb_gc_impl_malloc (objspace_ptr=0x55555555ccb0, size=<optimized out>) at gc/default/default.c:8195
#3  0x00007ffff796a45d in ruby_xmalloc_body (size=1671355) at gc.c:4582
#4  ruby_xmalloc (size=1671355) at gc.c:4572
#5  0x00007ffff796a647 in rb_xmalloc_mul_add_mul (x=x@entry=1, y=y@entry=1671354, z=z@entry=1, w=w@entry=1) at gc.c:4722
#6  0x00007ffff7aac25b in str_enc_new (klass=<optimized out>, 
    ptr=0x555556011110 "{\"columns\":[{\"namename\":\"BlROP3F2trcHst36OB1LPh\",\"column\":0,\"datatype\":\"S\"},{\"namename\":\"bzSH0NoYxTKVNLBKPnm928\",\"column\":1,\"datatype\":\"S\"},{\"namename\":\"htWHJf1bcLdut9cFpESMKT\",\"column\":2,\"datatype\":\""..., len=1671354, enc=0x5555555634b0) at string.c:1032
#7  0x00007ffff763581d in fbuffer_finalize (fb=0x7fffffffd880) at ../../../../../../ext/json/ext/generator/../fbuffer/fbuffer.h:230
#8  cState_partial_generate (self=140736869567560, obj=140737344306960, func=0x7ffff7636900 <generate_json_object>, io=0)
    at ../../../../../../ext/json/ext/generator/generator.c:1533
#9  0x00007ffff7b23305 in vm_call_cfunc_with_frame_ (ec=0x555555561d30, reg_cfp=0x7ffff65fefa0, calling=<optimized out>, argc=0, argv=<optimized out>, 
    stack_bottom=<optimized out>) at /home/myusername/.rbenv/sources/3.4.4/ruby-3.4.4/vm_insnhelper.c:3794
#10 0x00007ffff7b37cd4 in vm_sendish (method_explorer=<optimized out>, block_handler=<optimized out>, cd=<optimized out>, reg_cfp=<optimized out>, 
    ec=<optimized out>) at /home/myusername/.rbenv/sources/3.4.4/ruby-3.4.4/vm_callinfo.h:415
#11 vm_exec_core (ec=0x555555561d30) at /home/myusername/.rbenv/sources/3.4.4/ruby-3.4.4/insns.def:898
#12 0x00007ffff7b3e82a in rb_vm_exec (ec=0x555555561d30) at vm.c:2595
#13 0x00007ffff7b518fb in rb_iseq_eval_main (iseq=iseq@entry=0x7ffff7379048) at vm.c:2861
#14 0x00007ffff7943e85 in rb_ec_exec_node (ec=ec@entry=0x555555561d30, n=n@entry=0x7ffff7379048) at eval.c:281
#15 0x00007ffff79479eb in ruby_run_node (n=0x7ffff7379048) at eval.c:319
#16 0x0000555555555187 in rb_main (argv=0x7fffffffe028, argc=2) at ./main.c:43
#17 main (argc=<optimized out>, argv=<optimized out>) at ./main.c:68
```
