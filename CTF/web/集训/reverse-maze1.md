## maze1

地图题，用脚本画出地图

```python
a='  *******   *  **** * ****  * ***  *#  *** *** ***     *********'
for i in range(8):
    for j in range(8):
        print(a[j+i*8],end='')
    print('\n',end='')
    
  ******
*   *  *
*** * **
**  * **
*  *#  *
** *** *
**     *
********
```

跟进各个字符可发现
**O LEFT   o RIGFHT   . UP   0 DOWN**

nctf{o0oo00O000oooo..OO}

```sh
./homewrok_maze
Input flag:
nctf{o0oo00O000oooo..OO}
Congratulations!
```
