### sw-01
This challenge is fairly easy. We need to detect the architecture of thecelf file.

The ```file``` command is what we need.

```bash
file sw-01
```

Output:

> sw-01: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, BuildID[sha1]=0073012c38af01374a53569a0d79290259d34d8d, not stripped

ARM BABY :)

### sw-02
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

### sw-03

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