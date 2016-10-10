## Flag

| Level | Points |
| ----- | ------ |
| Easy | 7 |

> Papa brought me a packed present! let's open it.

#### Solution

First, run `./flag`

```
./flag
I will malloc() and strcpy the flag there. take it.
```

It's nothing, but we can open it with `gdb`

```
(gdb) info functions
All defined functions:
```

It's nothing too, maybe something wrong. Let's try this with `hexdump`

```
00000000  7f 45 4c 46 02 01 01 03  00 00 00 00 00 00 00 00  |.ELF............|
00000010  02 00 3e 00 01 00 00 00  f0 a4 44 00 00 00 00 00  |..>.......D.....|
00000020  40 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |@...............|
00000030  00 00 00 00 40 00 38 00  02 00 40 00 00 00 00 00  |....@.8...@.....|
00000040  01 00 00 00 05 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 40 00 00 00 00 00  00 00 40 00 00 00 00 00  |..@.......@.....|
00000060  04 ad 04 00 00 00 00 00  04 ad 04 00 00 00 00 00  |................|
00000070  00 00 20 00 00 00 00 00  01 00 00 00 06 00 00 00  |.. .............|
00000080  d8 62 0c 00 00 00 00 00  d8 62 6c 00 00 00 00 00  |.b.......bl.....|
00000090  d8 62 6c 00 00 00 00 00  00 00 00 00 00 00 00 00  |.bl.............|
000000a0  00 00 00 00 00 00 00 00  00 00 20 00 00 00 00 00  |.......... .....|
000000b0  fc ac e0 a1 55 50 58 21  1c 08 0d 16 00 00 00 00  |....UPX!........|
000000c0  21 7c 0d 00 21 7c 0d 00  90 01 00 00 92 00 00 00  |!|..!|..........|
000000d0  08 00 00 00 f7 fb 93 ff  7f 45 4c 46 02 01 01 03  |.........ELF....|
```

In the header of file, can see `UPX!` so this file is packed with `UPX`. Decompress it

```
upx -d flag -o flag.out
```

Open `./flag.out` with `gdb`

```
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   %rbp
   0x0000000000401165 <+1>:	mov    %rsp,%rbp
   0x0000000000401168 <+4>:	sub    $0x10,%rsp
   0x000000000040116c <+8>:	mov    $0x496658,%edi
   0x0000000000401171 <+13>:	callq  0x402080 <puts>
   0x0000000000401176 <+18>:	mov    $0x64,%edi
   0x000000000040117b <+23>:	callq  0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    %rax,-0x8(%rbp)
   0x0000000000401184 <+32>:	mov    0x2c0ee5(%rip),%rdx        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    -0x8(%rbp),%rax
   0x000000000040118f <+43>:	mov    %rdx,%rsi
   0x0000000000401192 <+46>:	mov    %rax,%rdi
   0x0000000000401195 <+49>:	callq  0x400320
   0x000000000040119a <+54>:	mov    $0x0,%eax
   0x000000000040119f <+59>:	leaveq 
   0x00000000004011a0 <+60>:	retq   
End of assembler dump.
```

Inline `main+32` we seen the comment `#0x6c2070 <flag>`, maybe `flag` is stored in `$rdx`. Let's try

```
(gdb) b *main+32
Breakpoint 1 at 0x401184
```
```
(gdb) b *main+39
Breakpoint 2 at 0x40118b
```
```
(gdb) r
Starting program: /home/traioi/flag.out
I will malloc() and strcpy the flag there. take it.

Breakpoint 1, 0x0000000000401184 in main ()
```
```
(gdb) c
Continuing.

Breakpoint 2, 0x000000000040118b in main ()
```
```
(gdb) x/s $rdx
0x496628:	"<flag>"
```