# Alkaloid Stream
*I found a weird stream cipher scheme. Can you break this?*
## Desciption
This crypto challenge gives you a python script and an output file of encrypted data. You must decrypt the data to find the flag. 
## Solution
The gen_keystream function is the most important: 
```py
def gen_keystream(key):  
    ln = len(key)  
      
    # Generate some fake values based on the given key...  
  fake = [0] * ln  
    for i in range(ln):  
        for j in range(ln // 3):  
            if i + j + 1 >= ln:  
                break  
  fake[i] ^= key[i + j + 1]  
  
    # Generate the keystream  
  res = []  
    for i in range(ln):  
        t = random.getrandbits(1)  
        if t:  
            res.append((t, [fake[i], key[i]]))  
        else:  
            res.append((t, [key[i], fake[i]]))  
  
    # Shuffle!  
  random.shuffle(res)  
  
    keystream = [v[0] for v in res]  
    public = [v[1] for v in res]  
    return keystream, public
```
It takes one parameter: a previously psuedo-randomly generated array of numbers. The function generates a fake value for each value in the key and puts the real values from the key and fake value together in an array of two elements. The two elements are in a random order - the real value could be the first or second one. Lastly, it puts each array of two numbers into an array and shuffles it. 

The generation of the fake values is the vulnerable part. For each fake value, it xores together the next third of the key. In our case, the key array is 600 long so one third of that is 200. 

As an example, to calculate the fake value for index 250:
Take the key value with index 251, xor that with key value 252, then 253, and so on ending with the key value at index 450. 

To exploit the vulnerability in this, we have to start at the end and go backwards. The very last fake value should be 0, because there are no key values after the very end. 

This can be confirmed by using ctrl+f in the text editor: 
![Image 1](https://raw.githubusercontent.com/GabeG888/writeups/master/pbctf-2021/Alkaloid%20Stream/img/AlkaloidImg1.png)

Remember the array is shuffled so the 0 won't be at the very end.

So the other value in that pair of numbers is the last key value. Because the zero, which we know is the fake value, is in the second location (index 1), we know the last value of the keystream is a 0 because the real value is in index 0. Now we have to find the second to last number. Because each fake value is the next 200 real values xored, the second to last fake value must be the same as the last real value. The third to last fake value is the last two real values xored. We can continue this pattern all the way to the beginning, always using the next 200 numbers. 

Now, unless you want to do tens of thousands of xor operations by hand, we need to implement this in python. I just used the same file we were given and wrote my code at the bottom.

The encrypted flag is encrypted in hex so I used CyberChef to turn it into binary. 

![Image 2](https://raw.githubusercontent.com/GabeG888/writeups/master/pbctf-2021/Alkaloid%20Stream/img/AlkaloidImg2.png)
I then manually edited it to make it an array like [0, 1, 1] and set the variable enc to it at the top of my python file. I also copy and pasted the rest of the output file into the python file and put it in the variable public. 
![Image 3](https://raw.githubusercontent.com/GabeG888/writeups/master/pbctf-2021/Alkaloid%20Stream/img/AlkaloidImg3.png)

Here is the final code, excluding enc, public, and the code given:
```py
real = []
keystream = [-1] * 600
found = False
while True:
	correct = 0
	for i in real:
		correct ^= i
	for i in range(len(public)):
		if public[i][0] == correct:
			keystream[i] = 1
			real.append(public[i][1])
			if len(real) > 200: real = real[1:]
			found = True
			public[i] = [[],[]]
		elif public[i][1] == correct:
			keystream[i] = 0
			real.append(public[i][0])
			if len(real) > 200: real = real[1:]
			found = True
			public[i] = [[],[]]
	if not found: break
	found = False
print(bits_to_bytes(xor(keystream, enc)))
```
How the code works:
* Real contains an array of all the values from the key.
* Correct is the value that is being searched for. At the beginning of each iteration, correct is set to all the values in real xored together.
* After setting correct, it iterates through public, containing the array of arrays of two numbers. When it finds correct it:
	* Sets the value for keystream at that index based on the index of the real value
	* Adds the real value of that pair to the real array
	* If real is more than 200 long, it removes the first element
	* Sets found to true, if found is false after searching the entire array we know it is done so it exits
	* Removes that pair from public. This is because when looking for the second to last pair, there is only one real value after that, making it search for the last real value xored with nothing, so it will find the last pair again. This results in a zero being added to the reals array and it will mess up the keystream and make everything off by one when removing values to stay at 200. This is most easily fixed by just removing each pair after it's found.

* After it has finished, it prints out the keystream xored with enc, which produces the key:```pbctf{super_duper_easy_brute_forcing_actually_this_one_was_made_by_mistake}```
