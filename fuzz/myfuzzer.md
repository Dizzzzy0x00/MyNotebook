---
description: 编写一个自己的模糊测试器
---

# MyFuzzer

```
apt-get install bsdmainutils
```

目标：fuzz一个简单的exif程序

````python
```python
import sys
import random
from pexpect import run
from pipes import quote
flip_rate = 0.01
def get_bytes(filename):
    #以字节的形式读取文件
    file = open(filename,"rb").read()
    return bytearray(file)

def bit_flip(data):
    #实现一个简单的随机按位翻转
    num_of_flips = int ((len(data) - 4 )*flip_rate)
    indexes = range(4,(len(data) - 4 ))
    chosen_index = []
    counter = 0
    while counter < num_of_flips:
        chosen_index.append(random.choice(indexes))
        counter += 1
    for x in chosen_index:
        current = data[x]
        #将 current 转换成二进制的字符串形式，并移除 "0b"（二进制的前缀）
        current = (bin(current).replace("0b",""))
        current = "0" * (8-len(current))+current
        indexes = range(0,8)

        picked_index = random.choice(indexes)

        new_number = []

        for i in current:
            new_number.append(i)
        #flip
        if new_number[picked_index] =="1":
            new_number[picked_index] = "0"
        else:
            new_number[picked_index] = "1"
        
        current = ''
        for i in new_number:
            current += i
        
        current = int (current,2)
        data[x] = current

    return data


def magic(data):

	magic_vals = [
	(1, 255),
	(1, 255),
	(1, 127),
	(1, 0),
	(2, 255),
	(2, 0),
	(4, 255),
	(4, 0),
	(4, 128),
	(4, 64),
	(4, 127)
	]

	picked_magic = random.choice(magic_vals)

	length = len(data) - 8
	index = range(0, length)
	picked_index = random.choice(index)

	# here we are hardcoding all the byte overwrites for all of the tuples that begin (1, )
	if picked_magic[0] == 1:
		if picked_magic[1] == 255:			# 0xFF
			data[picked_index] = 255
		elif picked_magic[1] == 127:			# 0x7F
			data[picked_index] = 127
		elif picked_magic[1] == 0:			# 0x00
			data[picked_index] = 0

	# here we are hardcoding all the byte overwrites for all of the tuples that begin (2, )
	elif picked_magic[0] == 2:
		if picked_magic[1] == 255:			# 0xFFFF
			data[picked_index] = 255
			data[picked_index + 1] = 255
		elif picked_magic[1] == 0:			# 0x0000
			data[picked_index] = 0
			data[picked_index + 1] = 0

	# here we are hardcoding all of the byte overwrites for all of the tuples that being (4, )
	elif picked_magic[0] == 4:
		if picked_magic[1] == 255:			# 0xFFFFFFFF
			data[picked_index] = 255
			data[picked_index + 1] = 255
			data[picked_index + 2] = 255
			data[picked_index + 3] = 255
		elif picked_magic[1] == 0:			# 0x00000000
			data[picked_index] = 0
			data[picked_index + 1] = 0
			data[picked_index + 2] = 0
			data[picked_index + 3] = 0
		elif picked_magic[1] == 128:			# 0x80000000
			data[picked_index] = 128
			data[picked_index + 1] = 0
			data[picked_index + 2] = 0
			data[picked_index + 3] = 0
		elif picked_magic[1] == 64:			# 0x40000000
			data[picked_index] = 64
			data[picked_index + 1] = 0
			data[picked_index + 2] = 0
			data[picked_index + 3] = 0
		elif picked_magic[1] == 127:			# 0x7FFFFFFF
			data[picked_index] = 127
			data[picked_index + 1] = 255
			data[picked_index + 2] = 255
			data[picked_index + 3] = 255
		
	return data


def create_new(data):

	f = open("mutated.jpg", "wb+")
	f.write(data)
	f.close()

def exif(counter,data):

    command = "./MyFuzz/exif/exif mutated.jpg -verbose"

    out, returncode = run("sh -c " + quote(command), withexitstatus=1)

    if b"Segmentation" in out:
        f = open("crashes/crash.{}.jpg".format(str(counter)), "ab+")
        f.write(data)

    if counter % 100 == 0:
    	print(counter, end="\r")

if len(sys.argv) < 2:
	print("Usage: JPEGfuzz.py <valid_jpg>")

else:
	filename = sys.argv[1]
	counter = 0
	while counter < 1000:
		data = get_bytes(filename)
		functions = [0, 1]
		picked_function = random.choice(functions)
		if picked_function == 0:
			mutated = magic(data)
			create_new(mutated)
			exif(counter,mutated)
		else:
			mutated = bit_flip(data)
			create_new(mutated)
			exif(counter,mutated)
		counter += 1
```
````

ASan工具：

* 内存布局：ASan通过修改程序的内存布局，为每块分配的内存增加“红色区域”或者“毒药区域”，当程序访问这些区域时，ASan会立即报告错误
* 使用一个称为“影子内存”的结构来追踪每个内存字节的状态，这样它可以知道哪些内存是有效的，哪些已经被释放，哪些是缓冲区外的
