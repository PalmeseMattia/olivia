## Olicyber software security
### Disclaimer
This repository is intended for educational purposes only. The challenges and content provided here are not intended to facilitate cheating or unethical behavior in CTF competitions or any other assessments. Users are encouraged to use this resource responsibly and ethically, respecting the rules and integrity of any competitions or learning environments they participate in.
<!-- SW-01 -->
<details>
<summary>sw-01</summary>

This challenge is fairly easy. We need to detect the architecture of thecelf file.

The ```file``` command is what we need.

```bash
file sw-01
```

Output:

> sw-01: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, BuildID[sha1]=0073012c38af01374a53569a0d79290259d34d8d, not stripped

ARM BABY :)
</details>

<!-- SW-02 -->
<details>
<summary>sw-02</summary>

For this challenge, we need to explore the shared dependencies needed for our ELF to run.

the ``` ldd ``` command will work. Ultimately, this is just a wrapper of the dynamic linker and will output the list of dynamic libraries that this program needs.

BEWARE

This comes from the man page of ldd:

>  Be  aware  that  in  some  circumstances  (e.g., where the program specifies an ELF interpreter other than ld-linux.so), some versions of ldd may attempt to obtain the dependency information by attempting to directly execute  the  program,  which  may lead to the execution of whatever code is defined in the program's ELF interpreter, and perhaps to execution of the program itself.
(In glibc versions before 2.27, the upstream ldd  implementation did this for example, although most distributions provided a modified version that did not.)
Thus,  you should never employ ldd on an untrusted executable, since this may result in the execution of arbitrary code.

Another alternative is to use ```objdump```.

```bash
objdump -p sw-02 | grep NEEDED
```
Output:

>   NEEDED               F
  NEEDED               L
  NEEDED               A
  NEEDED               G
  NEEDED               {
  NEEDED               1
  NEEDED               d
  NEEDED               8
  NEEDED               d
  NEEDED               b
  NEEDED               5
  NEEDED               5
  NEEDED               9
  NEEDED               }

I don't think i need to specify where is the flag :)
</details>

<!-- SW-03 -->
<details>
<summary>sw-03</summary>

In this challenge we need to analyze an ELF file in order to find a secret section.

We can use our amazing ```objdump``` to get a list of all the sections.

```bash
objdump -h sw-03
```

Output:
>  24 .comment      0000001b  0000000000000000  0000000000000000  00003028  2\*\*0
                  CONTENTS, READONLY
 25 .super-secret-section 0000001c  0000000000000000  0000000000000000  00003043  2\*\*0
                  CONTENTS, READONLY
 26 .debug_aranges 000000f0  0000000000000000  0000000000000000  00003060  2\*\*4
                  CONTENTS, READONLY, DEBUGGING, OCTETS

Should i say which one is the section that contains our flag? :)

We can now print the content of this super secret section

```bash
objdump -s -j .super-secret-section sw-03
```

Output:

> sw-03:     file format elf64-x86-64
Contents of section .super-secret-section:
 0000 46004c00 41004700 7b006400 30003300  F.L.A.G.{.d.0.3.
 0010 6c007600 6e003400 69007d00           l.v.n.4.i.}.

 I think that our job here is done.

By the way, this is a good opportunity to learn about __\_\_attributes\_\___ in C.
</details>

<!-- SW-04 -->
<details>
<summary>sw-04</summary>
In this challenge we just need to find the flag between the strings of the program.
Olicyber suggest to use the ```strings``` command.

```bash
strings sw-04
```

Output:

> GLIBC_2.4
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
\_\_gmon_start\_\_
_ITM_registerTMCloneTable
u/UH
[]A\A]A^A_
flag{0cca06f6}
 Qual'
 la flag? :
 Sbagliato! Prova ancora
 Giusto!
;*3$"
GCC: (GNU) 10.2.1 20201203
../sysdeps/x86_64
</details>

<!-- SW-05 -->
<details>
<summary>sw-05</summary>
This challenge is similar to the previous one, except that this time is recommended to use <a href="https://ghidra-sre.org">Ghidra</a>.
Objdump can go a long way, but this time i felt the need of something more suitable for the job.
In ghidra i began to rename some variables auto-generated and this was the code that i was left with:

```c
undefined8 main(void)

{
  int strcmp_res;
  long in_FS_OFFSET;
  ulong i;
  undefined8 len;
  char input [256];
  char password [264];
  long offset;
  
  offset = *(long *)(in_FS_OFFSET + 0x28);
  memset(input,0,0x100);
  memset(password,0,0x100);
  do {
    printf(&DAT_0010202c);
    fgets(input,0x100,stdin);
    len = strlen(input);
    if (len != 0) {
      if (input[len - 1] == '\n') {
        input[len - 1] = '\0';
      }
      for (i = 0; i < 0xe; i = i + 1) {
        password[i] = flag[i * 2];
      }
      strcmp_res = strcmp(input,password);
      if (strcmp_res == 0) {
        puts(&DAT_00102061);
        if (offset != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
          __stack_chk_fail();
        }
        return 0;
      }
    }
    puts(&DAT_00102045);
  } while( true );
}
```

We can see that the password is generated from the flag, by just picking the even elements from the buffer.
In the .rodata section we can find the effective value of the flag, but we can let ghidra guide us by clicking on the flag.

```asm
                    flag                              XREF[3]: Entry Point(*), 
                                                               main:001012ca(*), 
                                                               main:001012d1(*)  
   00102010 66 00      undef
            6c 00 
            61 00 
     00102010 66         undef  66h              [0]                     XREF[3]: Entry Point(*), 
                                                                                  main:001012ca(*), 
                                                                                  main:001012d1(*)  
     00102011 00         undef  00h              [1]
     00102012 6c         undef  6Ch              [2]
     00102013 00         undef  00h              [3]
     00102014 61         undef  61h              [4]
     00102015 00         undef  00h              [5]
     00102016 67         undef  67h              [6]
     00102017 00         undef  00h              [7]
     00102018 7b         undef  7Bh              [8]
     00102019 00         undef  00h              [9]
     0010201a 38         undef  38h              [10]
     0010201b 00         undef  00h              [11]
     0010201c 31         undef  31h              [12]
     0010201d 00         undef  00h              [13]
     0010201e 37         undef  37h              [14]
     0010201f 00         undef  00h              [15]
     00102020 35         undef  35h              [16]
     00102021 00         undef  00h              [17]
     00102022 30         undef  30h              [18]
     00102023 00         undef  00h              [19]
     00102024 65         undef  65h              [20]
     00102025 00         undef  00h              [21]
     00102026 36         undef  36h              [22]
     00102027 00         undef  00h              [23]
     00102028 33         undef  33h              [24]
     00102029 00         undef  00h              [25]
     0010202a 7d         undef  7Dh              [26]
     0010202b 00         undef  00h              [27]

```
We can finally start to pick the even elements to generate our flag :)
</details>
