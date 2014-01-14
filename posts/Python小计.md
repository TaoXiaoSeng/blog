# Python
## 记录
os.walk(dir)				遍历文件夹里的文件
str.rfind("abc")			反向查找abc的位置，返回pos
str.replace（"kk", "aa"）	替换kk为aa
cmp(a,b)					比较字符串a和b
os.path.basename(path) 		#返回文件名
os.path.exists(path) 		#路径存在则返回True,路径损坏返回False
os.path.isfile(path)  		#判断路径是否为文件
os.path.isdir(path)  		#判断路径是否为目录
os.path.islink(path)  		#判断路径是否为链接
os.path.splitext(path)  	#分割路径，返回路径名和文件扩展名的元组
							>>> os.path.splitext('file.1.2')
							('file.1', '.2')
os.remove(file)				#删除文件
os.rename(fromFile, toFile)#移动文件
