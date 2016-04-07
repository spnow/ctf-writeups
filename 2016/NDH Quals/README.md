## Exploit Me 200 (Secure File Reader)

```
$ file pwn
pwn: ELF 32-bit LSB  executable, Intel 80386, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.24, BuildID[sha1]=acc530c91c4841537384866623e6dc50074105c8, not stripped
```
checksec:
```
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

size check bypass => stack buffer overflow => ROP