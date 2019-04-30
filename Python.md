# Python

## WinError 32：另一个程序正在使用此文件，进程无法访问。

```python
def rename_file(files):
	for file in files:
		name = os.path.split(file)[1]
		dir = os.path.split(file)[0]
		if name.endswith("WAV"):
			newname = re.sub(r"WAV", r"wav", name)
			newfile = os.path.join(dir, newname)
			print(newfile)
			os.rename(file, newfile)
```

原因很可能是IO没关掉。

网上说出现WinError 2的原因是要把工作目录切换到当前目录

所以需要加os.chdir(path)切换到工作路径，为了观察路径切换的效果，我新加了os.getcwd()来获取当前的工作目录。

```python
def rename_file(files):
	for file in files:
		name = os.path.split(file)[1]
		dir = os.path.split(file)[0]
		if name.endswith("WAV"):
			newname = re.sub(r"WAV", r"wav", name)
			newfile = os.path.join(dir, newname)
			print(os.getcwd())#修改文件名之前当前目录
			os.chdir(dir)#切换目标路径
			print(os.getcwd())#检查此时当前目录
			os.rename(file, newfile)
```

可以看到程序运行结果是：

​	D:\keyboard_tone_detection-master\vad-master
​	D:\Download\试采-0314-3人-女-女-女-安卓\0314\宠物

但仍出现WinError 32错误，

找了一圈原因，突然意识到Adobe Audition正在占用此文件【笑哭。。】

## 

## os.path

| os.path.abspath(path)               | 返回绝对路径                                                 |
| ----------------------------------- | ------------------------------------------------------------ |
| os.path.basename(path)              | 返回文件名                                                   |
| os.path.commonprefix(list)          | 返回list(多个路径)中，所有path共有的最长的路径               |
| os.path.dirname(path)               | 返回文件路径                                                 |
| os.path.exists(path)                | 如果路径 path 存在，返回 True；如果路径 path 不存在，返回 False。 |
| os.path.lexists                     | 路径存在则返回True,路径损坏也返回True                        |
| os.path.expanduser(path)            | 把path中包含的"~"和"~user"转换成用户目录                     |
| os.path.expandvars(path)            | 根据环境变量的值替换path中包含的"$name"和"${name}"           |
| os.path.getatime(path)              | 返回最近访问时间（浮点型秒数）                               |
| os.path.getmtime(path)              | 返回最近文件修改时间                                         |
| os.path.getctime(path)              | 返回文件 path 创建时间                                       |
| os.path.getsize(path)               | 返回文件大小，如果文件不存在就返回错误                       |
| os.path.isabs(path)                 | 判断是否为绝对路径                                           |
| os.path.isfile(path)                | 判断路径是否为文件                                           |
| os.path.isdir(path)                 | 判断路径是否为目录                                           |
| os.path.islink(path)                | 判断路径是否为链接                                           |
| os.path.ismount(path)               | 判断路径是否为挂载点                                         |
| os.path.join(path1[, path2[, ...]]) | 把目录和文件名合成一个路径                                   |
| os.path.normcase(path)              | 转换path的大小写和斜杠                                       |
| os.path.normpath(path)              | 规范path字符串形式                                           |
| os.path.realpath(path)              | 返回path的真实路径                                           |
| os.path.relpath(path[, start])      | 从start开始计算相对路径                                      |
| os.path.samefile(path1, path2)      | 判断目录或文件是否相同                                       |
| os.path.sameopenfile(fp1, fp2)      | 判断fp1和fp2是否指向同一文件                                 |
| os.path.samestat(stat1, stat2)      | 判断stat tuple stat1和stat2是否指向同一个文件                |
| os.path.split(path)                 | 把路径分割成 dirname 和 basename，返回一个元组               |
| os.path.splitdrive(path)            | 一般用在 windows 下，返回驱动器名和路径组成的元组            |
| os.path.splitext(path)              | 分割路径，返回路径名和文件扩展名的元组                       |
| os.path.splitunc(path)              | 把路径分割为加载点与文件                                     |
| os.path.walk(path, visit, arg)      | 遍历path，进入每个目录都调用visit函数，visit函数必须有3个参数(arg, dirname, names)，dirname表示当前目录的目录名，names代表当前目录下的所有文件名，args则为walk的第三个参数 |
| os.path.supports_unicode_filenames  | 设置是否支持unicode路径名                                    |

## re正则表达式

re.sub(pattern,replace,string)

示例：

--去除中英文外的字符

```python
new_str = re.sub('[^\w\u4e00-\u9fff]+', '','江苏 » 无锡市:婚礼司仪roger')
```

Python正则表达式中中文的Unicode编码取值范围是【\u4e00-\u9fa5】，【\u4e00-\u9fff】包括了中日韩汉字。

--去掉字符串末尾的,?.

```python
word = re.sub("[\,\?\.]*$","",word)
```

[]里面是匹配的内容，*表示多次匹配，$表示只在字符串末尾匹配。

## 执行shell命令

```python
import subprocess

cutaudio = "ffmpeg -i {0} -vn -acodec copy -ss {1} -t {2} {3}".format(file, startTime, duration, newfile)
returncut = subprocess.call(cutaudio, shell=True)
print(returncut)
```

也可以试试（不知是否可行）

```python
os.system("ffmpeg -i {0} -vn -acodec copy -ss {1} -t {2} {3}".format(file, startTime, duration, newfile))
```

## 将时间转换成时：分：秒

```python
m, s = divmod(seconds, 60)
h, m = divmod(m, 60)
duration = str("%02d:%02d:%.3f" % (h, m, s))
```

其中 divmod(a,b)函数是计算除数和余数，返回这两个数组成的元祖。



