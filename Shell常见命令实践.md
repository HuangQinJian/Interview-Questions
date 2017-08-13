**ls -la :** 给出当前目录下所有文件的一个长列表，包括以句点开头的“隐藏”文件 

```
[bae@cp01-qa-yun-004.cp01.baidu.com huangqinjian]$ ls -a
.  ..  1  online_tools  online_tools_0803
```

![这里写图片描述](http://img.blog.csdn.net/20170804151206138?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---
**ll：** 竖列显示所有文件

```[bae@cp01-qa-yun-004.cp01.baidu.com huangqinjian]$ ll```

![这里写图片描述](http://img.blog.csdn.net/20170804151401272?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---

**pwd：** 查看当前路径

```
[bae@cp01-qa-yun-004.cp01.baidu.com online_tools]$ pwd
/home/bae/huangqinjian/online_tools
```

![这里写图片描述](http://img.blog.csdn.net/20170804195119529?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---

**cd：**切换目录
```
[bae@cp01-qa-yun-004.cp01.baidu.com huangqinjian]$ cd online_tools
[bae@cp01-qa-yun-004.cp01.baidu.com online_tools]$ pwd
/home/bae/huangqinjian/online_tools
```
![这里写图片描述](http://img.blog.csdn.net/20170804194851257?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---

**cat：** 显示文件内容 

```
[bae@cp01-qa-yun-004.cp01.baidu.com online_tools]$ cat upload.py
```

![这里写图片描述](http://img.blog.csdn.net/20170804195306625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---

**top：** 查看cpu、内存

```
[bae@cp01-qa-yun-004.cp01.baidu.com online_tools]$ top
```

![这里写图片描述](http://img.blog.csdn.net/20170804195501309?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---

**man：** 查看某个命令的帮助   

> man ls 显示ls命令的帮助内容

---

**diff：** 比较文件内容
  

> diff dir1 dir2 比较目录1与目录2的文件列表是否相同，但不比较文件的实际内容，不同则列出


```
[bae@cp01-qa-yun-004.cp01.baidu.com online_tools]$ diff ci data
Only in ci: ActionUserFeedback.class.php
Only in data: island
```

![这里写图片描述](http://img.blog.csdn.net/20170804195958023?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzU1MTIyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---

**vim：** 进入vim编辑文件

例如：`vim index.html`

按住`i`键进入编辑模式,编辑完按住`ESC`取消编辑,输入`:wq`保存,`:q`是不保存。

不保存退出的方法:很多时候打开了文件，或者修改了一些地方，才发现错了，非常需要不保存退出。

先按`ESC`，再输入冒号，在输入命令时，直接输入`q!`

---

**rm：** 删除文件命令
```
[bae@cp01-qa-yun-004.cp01.baidu.com html]$ rm index_demo.html
```
**格式：rm file**
删除文件file，系统会先询问是否删除。

**格式：rm -f file**
强行删除file，系统不再提示。

**格式：rm -rf dir**
强行删除目录dir下的所有文件、子目录下的所有文件和目录、删除dir本身。

---

**cp：** 复制文件
```
cp -rp /home/d001　/home/Documents
```

复制/home下d001到/home下Documents

-r 是遍历目录，即复制整个目录
-p 是保留原有属性

> cp　afile　afile.bak    把文件复制为新文件afile.bak
> 
> cp　afile　/home/bible/ 把文件afile从当前目录复制到/home/bible/目录下

---

**sz filename ：** 下载一个文件

**sz filename1 filename2：**下载多个文件

> 下载dir目录下的所有文件，不包含dir下的文件夹：sz dir/*
> 
> [bae@cp01-qa-yun-004.cp01.baidu.com html]$ sz index.html

---

**rz：** 上传文件

输入rz回车后，会出现文件选择对话框，选择需要上传文件，一次可以指定多个文件，上传到服务器的路径为当前执行rz命令的目录。

---

**su：** 切换用户     

> su – root　　　　切换到root用户

---

**vi下面如何进行回车换行？**

> ESC + I + Enter

---

**启动进程**

进入到进程的目录下 执行 `./＋进程名字`

```
[bae@cp01-qa-yun-004.cp01.baidu.com ~]$ ./start.sh
```
上面的命令运行是可能会出现权限不足的问题，最后跟大家说一个授权命令，假如我们想要给这个文件下的所有.sh文件授权，我们可以写`chmod u+x *.sh` 给sh文件授权，当然了我们也可以将*替换为具体的文件名，依据需要来定。