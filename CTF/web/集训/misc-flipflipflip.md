## [moectf2021]flipflipflip

主要就是base64解码，但是要不断的翻翻翻

脚本

```py
import base64
f= open("task.txt",'r')
flag=f.readline()
for i in range(100):
    try:
        flag=base64.b64decode(flag)
    except Exception as e:
        flag=flag[::-1]
        flag=base64.b64decode(flag)
    try: 
        flag = flag.decode('ascii')
    except Exception as e:
        flag = base64.b64encode(flag)[::-1]
        flag = flag.decode('ascii')

print(flag[::-1])
```





k 107 q 113 E 69 f 102 t 116 u 117 E 69 U 85 E 69 f 102 t 116 q 113 O 79 A 65 D 68 D 68 q 113 o 111 F 70 R 82 x 120 m 109 s 115 O 79 A 65 z 122 s 115 D 68 m 109 F 70 G 71 x 120 m 109 F 70 u 117 A 65 z 122 E 69 



a 97 b 98 c 99 d 100 e 101 f 102 g 103 h 104 i 105 j 106 k 107 l 108 m 109 n 110 o 111 p 112 q 113 r 114 s 115 t 116 u 117 v 118 w 119 x 120 y 121 z 122 A 65 B 66 C 67 D 68 E 69 F 70 G 71 H 72 I 73 J 74 K 75 L 76 M 77 N 78 O 79 P 80 Q 81 R 82 S 83 T 84 U 85 V 86 W 87 X 88 Y 89 Z 90 { 123 } 125 _ 95 

2  -2 32(0) 3 0 15 







moectf{attacking_the_vigenere_cipher_is_interesting}

moectf{moectf attacking the vigenere cipher is interesting}

1432145551

5411312

3331354

2541343

4352321

4521542

3541254

4431221

12521452323



14 32 14 55 51 54 11 31 23 33 13 54 25 41 34 34 35 23 21 45 21 54 23 54 12 54 44 31 22 11 25 21 45 23 23

ZpyLfxGmelDeftewJwFbwDGssZszbliileadaa

MOECTFILIKEISTHEPASSWORD

MOECTFILiKEiSTHEPASSWORD

MOECTFI1iKE  iSTHEPASSWORD





