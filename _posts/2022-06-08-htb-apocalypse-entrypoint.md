---
title: Hack The Box Cyber Apocalypse CTF 2022!
date: 2022-06-08
categories: [CTFs, HTB, HTB Apocalypse 2022, test]
tags: [htb, pwn, stack, format string]     # TAG names should always be lowercase
toc: true

# Math: uncomment line below as needed
# math: true       

# Images syntax:
#img_path: /home/kali/Desktop/kyzen-gmg.github.io/_posts/img/htb/     #set this so no need to use absolute paths for images
# ![The flower](flower.png)
# ![img-description](/path/to/image)
# _Image Caption_
# ![Desktop View](/assets/img/sample/mockup.png){: w="700" h="400" }    #size
# ![Desktop View](/assets/img/sample/mockup.png){: .normal } #.left or .right
---
![htb](/img/htb/cyber_apocalypse_2022.jpg)

Space Pirate: Entrypoint
- Format String attack on x64 ELF file to ovewrite the stack

```asm
└─# file sp_entrypoint 
sp_entrypoint: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./glibc/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=b40519cf907fdc2c181539b3714463758f7387d9, not stripped

└─# checksec sp_entrypoint 
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
    RUNPATH:  b'./glibc/'

└─# objdump -d ./sp_entrypoint -M intel | grep 00000
0000000000000838 <_init>:
0000000000000850 <.plt>:
0000000000000860 <strncmp@plt>:
0000000000000870 <puts@plt>:
0000000000000880 <__stack_chk_fail@plt>:
0000000000000890 <system@plt>:
00000000000008a0 <printf@plt>:
00000000000008b0 <alarm@plt>:
00000000000008c0 <read@plt>:
00000000000008d0 <srand@plt>:
00000000000008e0 <time@plt>:
00000000000008f0 <setvbuf@plt>:
0000000000000900 <strtoul@plt>:
0000000000000910 <exit@plt>:
0000000000000920 <rand@plt>:

0000000000000940 <_start>:
0000000000000970 <deregister_tm_clones>:
00000000000009b0 <register_tm_clones>:
0000000000000a00 <__do_global_dtors_aux>:
0000000000000a40 <frame_dummy>:
0000000000000a4a <read_num>:
0000000000000ac3 <banner>:
0000000000000b83 <open_door>:
0000000000000bd5 <setup>:
0000000000000c49 <check_pass>:
0000000000000cf6 <main>:
0000000000000e20 <__libc_csu_init>:
0000000000000e90 <__libc_csu_fini>:
0000000000000e94 <_fini>:
```

```
gdb-peda$ disass main
Dump of assembler code for function main:
   0x0000000000000cf6 <+0>:     push   rbp
   0x0000000000000cf7 <+1>:     mov    rbp,rsp
   0x0000000000000cfa <+4>:     sub    rsp,0x40
   0x0000000000000cfe <+8>:     mov    rax,QWORD PTR fs:0x28
   0x0000000000000d07 <+17>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000000d0b <+21>:    xor    eax,eax
   0x0000000000000d0d <+23>:    call   0xbd5 <setup>
   0x0000000000000d12 <+28>:    call   0xac3 <banner>
   0x0000000000000d17 <+33>:    mov    eax,0xdeadbeef
   0x0000000000000d1c <+38>:    mov    QWORD PTR [rbp-0x40],rax
   0x0000000000000d20 <+42>:    lea    rax,[rbp-0x40]
   0x0000000000000d24 <+46>:    mov    QWORD PTR [rbp-0x38],rax
   0x0000000000000d28 <+50>:    lea    rdi,[rip+0x18b1]        # 0x25e0
   0x0000000000000d2f <+57>:    mov    eax,0x0
   0x0000000000000d34 <+62>:    call   0x8a0 <printf@plt>
   0x0000000000000d39 <+67>:    mov    eax,0x0
   0x0000000000000d3e <+72>:    call   0xa4a <read_num>
   0x0000000000000d43 <+77>:    cmp    rax,0x1
   0x0000000000000d47 <+81>:    je     0xd51 <main+91>
   0x0000000000000d49 <+83>:    cmp    rax,0x2
   0x0000000000000d4d <+87>:    je     0xdaa <main+180>
   0x0000000000000d4f <+89>:    jmp    0xdb4 <main+190>
   0x0000000000000d51 <+91>:    lea    rdi,[rip+0x18b8]        # 0x2610
   0x0000000000000d58 <+98>:    mov    eax,0x0
   0x0000000000000d5d <+103>:   call   0x8a0 <printf@plt>
   0x0000000000000d62 <+108>:   lea    rax,[rbp-0x30]
   0x0000000000000d66 <+112>:   mov    edx,0x1f
   0x0000000000000d6b <+117>:   mov    rsi,rax
   0x0000000000000d6e <+120>:   mov    edi,0x0
   0x0000000000000d73 <+125>:   call   0x8c0 <read@plt>
   0x0000000000000d78 <+130>:   lea    rdi,[rip+0x18d9]        # 0x2658
   0x0000000000000d7f <+137>:   mov    eax,0x0
   0x0000000000000d84 <+142>:   call   0x8a0 <printf@plt>
   0x0000000000000d89 <+147>:   lea    rax,[rbp-0x30]
   0x0000000000000d8d <+151>:   mov    rdi,rax
   0x0000000000000d90 <+154>:   mov    eax,0x0
   0x0000000000000d95 <+159>:   call   0x8a0 <printf@plt>
   0x0000000000000d9a <+164>:   mov    rdx,QWORD PTR [rbp-0x40]
   0x0000000000000d9e <+168>:   mov    eax,0xdead1337
   0x0000000000000da3 <+173>:   cmp    rdx,rax
   0x0000000000000da6 <+176>:   jne    0xdd6 <main+224>
   0x0000000000000da8 <+178>:   jmp    0xdf0 <main+250>
   0x0000000000000daa <+180>:   mov    eax,0x0
   0x0000000000000daf <+185>:   call   0xc49 <check_pass>
   0x0000000000000db4 <+190>:   lea    rsi,[rip+0x17df]        # 0x259a
   0x0000000000000dbb <+197>:   lea    rdi,[rip+0x18a6]        # 0x2668
   0x0000000000000dc2 <+204>:   mov    eax,0x0
   0x0000000000000dc7 <+209>:   call   0x8a0 <printf@plt>
   0x0000000000000dcc <+214>:   mov    edi,0x1b39
   0x0000000000000dd1 <+219>:   call   0x910 <exit@plt>
   0x0000000000000dd6 <+224>:   lea    rsi,[rip+0x17bd]        # 0x259a
   0x0000000000000ddd <+231>:   lea    rdi,[rip+0x18bc]        # 0x26a0
   0x0000000000000de4 <+238>:   mov    eax,0x0
   0x0000000000000de9 <+243>:   call   0x8a0 <printf@plt>
   0x0000000000000dee <+248>:   jmp    0xdfa <main+260>
   0x0000000000000df0 <+250>:   mov    eax,0x0
   0x0000000000000df5 <+255>:   call   0xb83 <open_door>
   0x0000000000000dfa <+260>:   mov    eax,0x0
   0x0000000000000dff <+265>:   mov    rcx,QWORD PTR [rbp-0x8]
   0x0000000000000e03 <+269>:   xor    rcx,QWORD PTR fs:0x28
   0x0000000000000e0c <+278>:   je     0xe13 <main+285>
   0x0000000000000e0e <+280>:   call   0x880 <__stack_chk_fail@plt>
   0x0000000000000e13 <+285>:   leave  
   0x0000000000000e14 <+286>:   ret    
End of assembler dump.
```