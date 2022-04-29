# Gambler_Baby
## Description
This challenge gives you an ELF file and remote server that ask you to guess random strings to get coins. Getting enough coins gives you the flag. 
## Solution
The random seed is static so the random values will be the same each time. You could reverse engineer the code and recreate it yourself in C but instead I just ran the binary multiple times until I had found all the values. It is easier, though, to use the same script as `Gambler_Overflow` because that solution applies to `Gambler_Baby` as well.
