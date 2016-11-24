###protobuf 了解

Protocol Buffers 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

###编译环境配置

下载protobuf : protobuf-java-3.0.0.zip

安装 :
```
unzip protobuf-java-3.0.0.zip
cd protobuf-java-3.0.0
./configure
make
make check
make install
protoc --version
```
如果报错: 
```
protoc: error while loading shared libraries: libprotoc.so.10: cannot open shared object file: No such file or directory
```
错误原因： 
`protobuf的默认安装路径是/usr/local/lib，而/usr/local/lib 不在Ubuntu体系默认的 LD_LIBRARY_PATH 里，所以就找不到该lib`
解决方式:
```
sudo touch /etc/ld.so.conf.d/libprotobuf.conf
vi /etc/ld.so.conf.d/libprotobuf.conf
// 把protobuf的安装路径写进去
sudo ldconfig
```
###使用

####书写.proto文件

新建.proto文件
```
syntax = "proto3";
// 新版本的编译器不需要有syntax的标记
// 因为3版本和2版本在语法上有区别比如required/optional等关键字删除

option java_outer_classname = "DemoProtoBean";// 生成的java类文件名称
// 注意我没有写package是自己生成对象后手动加的

message Demo {// 实际的javabean文件名
    int32 id = 1;
    string name= 2;
    string address = 3;
    repeated string phones = 4;
    map<string,string> tags = 5;
}
```
语法了解可以 点击 鸟窝博客 ,虽然是2版本的,但是很详

生成.java文件
```
protoc --java_out=./ ./DemoProto2.proto
注意路径,第一个为输出路径,第二个为.proto文件路径 
生成.java文件不做叙述
```
####序列化和反序列化
序列化
```
// 序列化
byte[] bytes = ArrayList arrayList = new ArrayList();
        arrayList.add("18518111111");
        Map<String, String> map = new HashMap<>();
        map.put("1", "2");
DemoProtoBean.Demo.newBuilder().setId(1).setName("guodegang").setAddress("beijing")
                    .addAllPhones(arrayList)
                    .putAllTags(map)
                    .build().toByteArray();
// map的元素可以直接在builder里一个个put
```
反序列化
```
DemoProtoBean.Demo demo = DemoProtoBean.Demo.parseFrom(bytes);
```
###黑科技protostuff

protostuff是protobuf的加强版,protostuff动态支持了protobuff的预编译的过程,不需要对.proto文件进行预编译生成.java文件. 
序列化
```
Demo d = new Demo(1, "guodegang", "beijing", arrayList, map);
Schema<Demo> schema = RuntimeSchema.createFrom(Demo.class);
ProtostuffIOUtil.toByteArray(d, schema, LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE))
```
反序列化
```
Demo d1 = new Demo();
ProtostuffIOUtil.mergeFrom(bytes2, d1, schema);
```
> schema的就是protostuff动态生成的模板;可以卡电脑我们能够对JavaBean直接进行序列化了,而且代码量非常少但是有个缺点,schema如果每次都要生成的话,会极大的影响效率.使用的时候应该尽量预知shcema或者第一次的时候生成放到cachemap里.

###性能对比protostuff vs protobuf

对上述Demo实体,分别使用两种序列化技术进行10000000次的操作
```
for (int i = 0; i < 10000000; i++) {
    byte[] bytes = DemoProtoBean.Demo.newBuilder().setId(1).setName("bochenlong").setAddress("zhongguancun").addAllPhones(arrayList).putAllTags(map).build().toByteArray();
    DemoProtoBean.Demo demo = DemoProtoBean.Demo.parseFrom(bytes);
}
Schema<Demo> schema = RuntimeSchema.createFrom(Demo.class);
```
```
for (int i = 0; i < 10000000; i++) {  
     byte[] bytes2 = ProtostuffIOUtil.toByteArray(d, schema, LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE));
    Demo d1 = new Demo();
    ProtostuffIOUtil.mergeFrom(bytes2, d1, schema);
}
```
结果:
序列化: protobuf - protostuff 8189-3095
反序列化:protobuf - protostuff 5371-4980
综合                                       13312-8001

注意上面讲过schema每次都生成的情况下,结果会非常可怕,比如综合数据会 
protobuf耗时时间：8301 
protostuff耗时时间：74077
