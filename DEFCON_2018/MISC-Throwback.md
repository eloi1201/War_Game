# MISC-Throwback
---
<img width="778" alt="screen shot 2018-05-13 at 3 50 42 pm" src="https://user-images.githubusercontent.com/14992494/39997216-bb0dd9d0-57b4-11e8-831c-905c48540b1c.png">

```
text = "Anyo!e!howouldsacrificepo!icyforexecu!!onspeedthink!securityisacomm!ditytop!urintoasy!tem!"
```

The flag consists of transformed characters from exclamation marks in the text.

For example, there are 4 characters "Anyo" to reach first exclamation mark in the text. So the first exclamation mark is "d" which is 4th character of alphabet. Then the second exclamation mark is "a" starting character of alphabet.

I got flag string after writing python script.

```
Null2Root:~ eloi$ python
Python 2.7.10 (default, Feb  7 2017, 00:08:15) 
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.34)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> encStr = "Anyo!e!howouldsacrificepo!icyforexecu!!onspeedthink!securityisacomm!ditytop!urintoasy!tem!"
>>> encList = encStr.split("!")
>>> encList
['Anyo', 'e', 'howouldsacrificepo', 'icyforexecu', '', 'onspeedthink', 'securityisacomm', 'ditytop', 'urintoasy', 'tem', '']
>>> for i in range(len(encList)):
...     print len(encList[i]),
... 
4 1 18 11 0 12 15 7 9 3 0
>>> for i in range(len(encList)):
...     print chr(len(encList[i])+96),
... 
d a r k ` l o g i c `
>>> 

```

### Flag

dark logic
