## Collision

| Level | Points |
| ----- | ------ |
| Easy | 3 |

> Daddy told me about cool MD5 hash collision today.

> I wanna do something like that too!

#### Solution

In function `check_password`, we can see we need to add 5 `int* ip` whose sum is `0x21DD09EC` and length is `20 bytes`.

We see `0x21DD09EC = 0x03EA2CDD*4 + 0x12345678` then we can get flag with this
```
./col $(perl -e 'print "\xdd\x2c\xea\x03"x4 . "\x78\x56\x34\x12";')
```