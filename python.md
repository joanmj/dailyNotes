1. 读取文件时的第一行第一个字符是"\ufeff",而不是内容

   解决方法:在读取B.txt 时,指定编码方式为 "utf-8-sig"即可. ”\ufeff“叫BOM("ByteOrder Mark")用来声明该文件的编码信息."uft-8-sig"中sig全拼为 signature 也就是"带有签名的utf-8", 因此"utf-8-sig"读取带有BOM的"utf-8文件时"会把BOM单独处理,与文本内容隔离开,也是我们期望的结果.

   ```python
   with open("B.txt", 'r', encoding='utf-8-sig') as f:
       line = f.readline()[0:-1]  #去掉末尾换行符
       print(line)
       with open("%s.txt" % line, 'r', encoding='utf-8') as f1:
           print(f1.readline())
       f1.close()
   f.close()
   ```

   

## python基础



python