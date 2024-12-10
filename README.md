
​MySQL数据库导出数据时，允许使用包含导出定义的SELECT语句进行数据的导出操作。该文件被创建到服务器主机上，因此必须拥有文件写入权限（FILE权限）才能使用此语法。“SELECT...INTO OUTFILE 'filename'”形式的SELECT语句可以把被选择的行写入一个文件中，并且filename不能是一个已经存在的文件。SELECT...INTO OUTFILE语句的基本格式如下：



```
SELECT columnlist  FROM table WHERE condition  INTO OUTFILE 'filename'  [OPTIONS]
```


可以看到SELECT columnlist FROM table WHERE condition为一个查询语句，查询结果返回满足指定条件的一条或多条记录；INTO OUTFILE语句的作用就是把前面SELECT语句查询出来的结果，导出到名称为“filename”的外部文件中；\[OPTIONS]为可选参数选项，OPTIONS部分的语法包括FIELDS和LINES子句，其可能的取值如下：



* FIELDS TERMINATED BY 'value'：设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符（\\t）。
* FIELDS \[OPTIONALLY] ENCLOSED BY 'value'：设置字段的包围字符，只能为单个字符，若使用了OPTIONALLY关键字，则只有CHAR和VERCHAR等字符数据字段被包围。
* FIELDS ESCAPED BY 'value'：设置如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认值为“\\”。
* LINES STARTING BY 'value'：设置每行数据开头的字符，可以为单个或多个字符，默认情况下不使用任何字符。
* LINES TERMINATED BY 'value'：设置每行数据结尾的字符，可以为单个或多个字符，默认值为“\\n”。


FIELDS和LINES两个子句都是自选的，但是如果两个都被指定了，则FIELDS必须位于LINES的前面。


使用SELECT...INTO OUTFILE语句，可以非常快速地把一张表转储到服务器上。如果想要在服务器主机之外的部分客户主机上创建结果文件，不能使用SELECT...INTO OUTFILE，应该使用“MySQL –e "SELECT ..." \> file\_name”这类的命令来生成文件。


SELECT...INTO OUTFILE是LOAD DATA INFILE的补语，用于语句的OPTIONS部分的语法包括部分FIELDS和LINES子句，这些子句与LOAD DATA INFILE语句同时使用。


【例11\.10】使用SELECT...INTO OUTFILE将test\_db数据库中的person表中的记录导出到文本文件，SQL语句如下：





```
mysql> SELECT *  FROM test_db.person INTO OUTFILE 'D:/person0.txt';
```


语句执行后报错信息如下：






```
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```


这是因为MySQL默认对导出的目录有权限限制，也就是说使用命令行进行导出的时候，需要指定目录。那么指定的目录是什么呢？



查询指定目录的命令如下：





```
show global variables like '%secure%';
```


执行结果如下：






```
+-------------------------+-----------------------------------------------+
| Variable_name           | Value                                         |
+-------------------------+-----------------------------------------------+
|require_secure_transport | OFF                                           |
|secure_file_priv         | D:\ProgramData\MySQL\MySQL Server 9.0\Uploads\|
+-------------------------+-----------------------------------------------+
```


因为secure\_file\_priv配置的关系，所以必须导出到D:\\ProgramData\\MySQL\\MySQL Server 9\.0\\Uploads\\目录下，该目录就是指定目录。如果想自定义导出路径，需要修改my.ini配置文件。打开路径D:\\ProgramData\\MySQL\\MySQL Server 9\.0，用记事本打开my.ini文件，然后搜索以下代码：






```
secure-file-priv="D:/ProgramData/MySQL/MySQL Server 9.0/Uploads"
```


在上述代码前添加\#注释掉，然后添加以下内容：






```
secure-file-priv="D:/"
```


结果如图11\.1所示。



![](https://img2024.cnblogs.com/blog/270128/202412/270128-20241209103546339-236640010.gif "点击并拖拽以移动")​![](https://img2024.cnblogs.com/blog/270128/202412/270128-20241209103546426-831976458.png)


重启MySQL服务器后，再次使用SELECT...INTO OUTFILE将test\_db数据库中的person表中的记录导出到文本文件，SQL语句如下：





```
mysql>SELECT *  FROM test_db.person INTO OUTFILE 'D:/person0.txt';
Query OK, 1 row affected (0.01 sec)
```


由于指定了INTO OUTFILE子句，因此SELECT会将查询出来的3个字段值保存到C:\\person0\.txt文件中。打开该文件，内容如下：



```
1    Green        21    Lawyer
2    Suse         22    dancer
3    Mary         24    Musician
4    Willam       20    sports man
5    Laura        25    \N
6    Evans        27    secretary
7    Dale         22    cook
8    Edison       28    singer
9    Harry        21    magician
10   Harriet      19    pianist
```


默认情况下，MySQL使用制表符（\\t）分隔不同的字段，字段没有被其他字符包围。另外，第5行中有一个字段值为“\\N”，表示该字段的值为NULL。默认情况下，当遇到NULL时，会返回“\\N”，代表空值，其中的反斜线（\\）表示转义字符；如果使用ESCAPED BY选项，则N前面为指定的转义字符。



【例11\.11】使用SELECT...INTO OUTFILE语句，将test\_db数据库person表中的记录导出到文本文件，使用FIELDS选项和LINES选项，要求字段之间使用逗号分隔，所有字段值用双引号引起来，定义转义字符为单引号“\\'”，SQL语句如下：





```
SELECT * FROM  test_db.person INTO OUTFILE "D:/person1.txt"
FIELDS 
TERMINATED BY ','
ENCLOSED BY '\"'
ESCAPED BY '\''
LINES 
TERMINATED BY '\r\n';
```


该语句将把person表中所有记录导入D盘目录下的person1\.txt文本文件中。



“FIELDS TERMINATED BY ','”表示字段之间用逗号分隔；“ENCLOSED BY '\\"'”表示每个字段用双引号引起来；“ESCAPED BY '\\'”表示将系统默认的转义字符替换为单引号；“LINES TERMINATED BY '\\r\\n'”表示每行以回车换行符结尾，保证每一条记录占一行。


执行成功后，在D盘下生成一个person1\.txt文件。打开文件，内容如下：





```
"1","Green","21","Lawyer"
"2","Suse","22","dancer"
"3","Mary","24","Musician"
"4","Willam","20","sports man"
"5","Laura","25",'N'
"6","Evans","27","secretary"
"7","Dale","22","cook"
"8","Edison","28","singer"
"9","Harry","21","magician"
"10","Harriet","19","pianist"
```


可以看到，所有的字段值都被双引号引起来；第5条记录中空值的表示形式为“N”，即使用单引号替换了反斜线转义字符。



【例11\.12】使用SELECT...INTO OUTFILE语句，将test\_db数据库person表中的记录导出到文本文件，使用LINES选项，要求每行记录以字符串“\>”开始、以字符串“”结尾，SQL语句如下：





```
SELECT * FROM  test_db.person INTO OUTFILE "D:/person2.txt"
LINES 
STARTING BY '> '
TERMINATED BY '';
```


语句执行成功后，在D盘下生成一个person2\.txt文件。打开该文件，内容如下：






```
"1","Green","21","Lawyer"
"2","Suse","22","dancer"
"3","Mary","24","Musician"
"4","Willam","20","sports man"
"5","Laura","25",'N'
"6","Evans","27","secretary"
"7","Dale","22","cook"
"8","Edison","28","singer"
"9","Harry","21","magician"
"10","Harriet","19","pianist"
```


可以看到，虽然将所有的字段值导出到文本文件中，但是所有的记录没有分行，出现这种情况是因为TERMINATED BY选项替换了系统默认的换行符。如果希望换行显示，则需要修改导出语句：






```
SELECT * FROM  test_db.person INTO OUTFILE "D:/person3.txt"
LINES 
STARTING BY '> '
TERMINATED BY '\r\n';
```


执行完语句之后，换行显示每条记录，结果如下：






```
> 1    Green        21    Lawyer 
> 2    Suse         22    dancer 
> 3    Mary         24    Musician 
> 4    Willam       20    sports man 
> 5    Laura        25    \N 
> 6    Evans        27    secretary 
> 7    Dale         22    cook 
> 8    Edison       28    singer 
> 9    Harry        21    magician 
> 10   Harriet      19    pianist 
```


 



 本博客参考[slower加速器官网](https://chundaotian.com)。转载请注明出处！
