## Fd

| Level | Points |
| ----- | ------ |
| Easy | 1 |

> Mommy! what is a file descriptor in Linux?

#### Solution

We need to send `LETMEWIN` to stdin_input which mean `fd == 0` and `argv[1] == 0x1234`.

We see
```
fd@ubuntu:~$ perl -e 'print 0x1234;'
4660
```
Then can get flag with this
```
fd@ubuntu:~$ echo 'LETMEWIN' | ./fd 4660
```
