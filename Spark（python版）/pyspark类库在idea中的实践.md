**安装python类库**

```
pip install pyspark pyhive pymysql jieba -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

- 阿里云 http://mirrors.aliyun.com/pypi/simple/

- 豆瓣(douban) http://pypi.douban.com/simple/

- 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/

- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

- 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

**PythonAPI的学习之pyspark环境整理**

写一个小脚本

这个脚本的作用就是把写的python文件由Linux宿主机上传至docker容器内，这里我的spark环境和hadoop环境都是基于Hadoop的，所以在idea中编写完python代码之后复制到linux宿主机上，然后通过脚本上传至容器内

```sh
#!/bin/bash
CURRENT_DIR=$(cd $(dirname $0); pwd)
DOCKER_ID=$(docker ps|grep iceberg|awk '{print $1}') 
if [ -n "$1" ]
then
    	echo 'upload $1 file begin...'
	echo "fileName==>$CURRENT_DIR/$1"
	if [ -f $CURRENT_DIR/$1 ];then
		
        	docker cp $CURRENT_DIR/$1 $DOCKER_ID:/usr/local/pythonFiles
        	echo 'upload successful!'
	else
		echo 'error! file not exist!'
	fi
else
    echo 'error! python file is must!'
fi
```

开启pyspark服务，并提交python文件进行计算

```
cd /usr/local/spark/bin
# 提交任务
./spark-submit --master local[*] /usr/local/pythonFiles/test.py 2
```

PythonAPI编写代码

```python
# coding:utf8
from pyspark import SparkConf, SparkContext

if __name__ == "__main__":
    conf = SparkConf().setMaster("local[*]").setAppName("WordCountHelloWorld")
    # 通过SparkConf对象构建SparkContext对象
    sc = SparkContext(conf=conf)

    # 需求：wordcount单词计数，读取HDFS上的words.txt文件，对其内部的单词统计出现的数量
    # 读取文件
    file_rdd = sc.textFile("hdfs://hadoop:8020/words.txt")
    # 将单词进行切割，得到一个存储全部单词的集合对象
    words_rdd = file_rdd.flatMap(lambda line: line.split(" "))
    # 将单词转为元组对象，key是单词，value是数字1
    words_with_one_rdd = words_rdd.map(lambda x: (x, 1))
    # 将元组的value按照key来分组，对所有的value执行聚合操作（相加）
    result_rdd = words_with_one_rdd.reduceByKey(lambda a, b: a + b)
    # 通过collect方法收集rdd的数据打印输出结果
    print(result_rdd.collect())
```
