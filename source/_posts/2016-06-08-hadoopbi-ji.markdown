---
layout: post
title: "Hadoop笔记"
date: 2016-06-08 22:52:09 +0800
comments: true
categories: hadoop
---
### 1. 关于 Hadoop
1. FileSystem.get( conf ) 会得到缓存的filesystem，因为是缓存的filesystem，这个filesystem是不能关闭的。一旦关闭之后，其他的机器有可能也在读文件，关闭以后其他的机器就不能再读了
2. Hadoop 里面有两个mapreduce的包，一个是hadoop.mapred，一个是hadoop.mapreduce，这两个包不一样的。

    - org.apache.hadoop.mapred 是老的api，org.apache.hadoop.mapreduce是新的api
    - 新的API使用context，并允许用户代码与mapreduce系统进行通信，旧的里面是通过OutputCollector和Reporter两个来通信。但在新的里面，所有的一切都是通过context

3. 作业控制通过Job而不是JobClient，作业配置通过Configuration而不是JobConf，老的API通过JobConf对通用的Configuration进行扩展，但在新的里面一切都是Configuration

4. Set hadoop heap size before the job running: 
 >     export HADOOP_HEAPSIZE=4096

### 2. 关于DistributedCache

1. Use distributed cache: 
```
public void addToDistributedCache(Configuration conf, Path path) throws IOException {
 FileSystem fs = FileSystem.get(conf);
 FileStatus fileStatus = fs.getFileStatus(path);

 if (fileStatus.isDirectory()) {
   FileStatus[] allStatus = fs.listStatus(path);
   for (FileStatus status : allStatus) {
     DistributedCache.addCacheFile(status.getPath().toUri(), conf);
     LOG.info("add file : " + status.getPath().toString());
   }
 } else {
   DistributedCache.addCacheFile(path.toUri(), conf);
   LOG.info("add file: " + path.toString());
 }
}
```
2. DistributedCache 无法使用：可能的原因是使用的conf是在new Job

3. 如果distributed cache使用了很多的文件都是part-r-00000这样的文件，可以使用alias这样的办法处理:
```
job.addCacheFile(new URI(options.getIdf() + "#" + IDF_ALIAS));
```

### 3. 关于Lzo和Lzop
1. 使用lzo压缩格式压缩文件
```
OutputFormat.setCompressOutput(job, true);
OutputFormat.setOutputCompressor( job, LzopCodec.class)
```

2. LzoCodec 和 LzopCodec的区别
```
lzo 是一个压缩的lib，可以用来快速的压缩和解压缩，用来map和reduce中间
lzop是一个文件压缩的方式，和gzip相同，用来输出
```

### 4. 关于shuffle error
```
job 出现了 org.apache.hadoop.mapreduce.task.reduce.Shuffle$ShuffleError: error in shuffle in fetcher#3, 
网上有很多调整内存的方法，好像都没用。试了一会，调整reduce的内存，还是没用。最后，调整了reducer 的个数，job跑过。
```
