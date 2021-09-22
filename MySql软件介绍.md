### MySQL产品的介绍和安装

#### 1、干净卸载MySQL

- 首先用普通软件卸载MySql

- 其次C:\Program Files (x86)\MySQL， 把MySQL文件夹给整体卸载（卸载残留）

- 隐藏文件ProgramData 里的MYSQL给删掉

- 如果还没卸载干净的话，清理注册表（一般来说不需要，除非安装时出问题了）

  ```markdown
  windowns + r -> regedit - >具体CSDN
  ```

  

#### 2、安装

![image-20210912125028924](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912125028924.png)

![image-20210912125815786](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912125815786.png)

![image-20210912125941598](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912125941598.png)

![image-20210912130130285](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912130130285.png)

![image-20210912130322835](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912130322835.png)

![image-20210912130521707](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912130521707.png)





#### 3、配置文件

在路径C:\ProgramData\MySQL\MySQL Server 8.0下的my.ini配置文件用记事本打开

找到服务端配置[mysqld]

```markdown
在这里可以更改端口号

找到安装目录：basedir="C:/Program Files/MySQL/MySQL Server 8.0/"

找到文件目录：datadir=C:/ProgramData/MySQL/MySQL Server 8.0/Data
          sql中的数据通过管理，都放到数据文件里面去了，这是路径
          
 可以更改字符集：character-set-server=utf8
 
 数据库存储引擎：default-storage-engine=INNODB  他是用来执行数据库语句的。
 
 
 #### 总而言之服务端的配置可以在哪里改，该完后一定要注意，一点要双击重新启动一下my.ini配置文件，才算配置成功
```



#### 4、如何产看服务名称

1、win + r –> regegit ->

2、计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\【MySQL80】

3、DIsplayName -> 数据：【MySQL80】 便是服务名称



#### 5、数据库常见内容

![image-20210912155155324](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912155155324.png)



#### 6、图形画界面工具

新建 数据连接名（任意）-> MySQL栏 ->我的SQL主机地址：（localhost）,如果是其他的要填写其它的主机名或ip -》输入用户名root -> 密码 -> 端口号 -> 数据/库 (啥也不填表示所有的数据库) -> 连接

![image-20210912163811867](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912163811867.png)

![image-20210912163852290](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912163852290.png)





一个询问就代表一个sql文件，文件名.sql的文件打开就在mysql中显示询问命令语句。



字体：![image-20210912164321734](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912164321734.png)

或者ctrl + 鼠标滚动轴



> 每条sql命令后面加上封号；





#### 7、执行sql脚本（可完全执行的脚本）

以.sql文件名结尾

在mysql里执行它

![image-20210912170051530](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912170051530.png)

定义主机名右击 -> 执行sql脚本 -> 找到以文件.sql文件 -> 执行即可



弄好之后，这个软件不能主动刷新，需要手动刷新，才能出现执行后的结果

![image-20210912170229122](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210912170229122.png)

### 连接查询几种模式



![image-20210916104129848](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210916104129848.png)

![image-20210916104148214](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210916104148214.png)

### mysql创建数据库的方式

![image-20210920110540530](MySql%E8%BD%AF%E4%BB%B6%E4%BB%8B%E7%BB%8D.assets/image-20210920110540530.png)

### 数据库数据存储的位置

```markdown
porgramdata -> mysql -> mysql server -> data
```

