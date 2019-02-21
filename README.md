# lldb
lldb sos分析coredump

基于coredump文件分析需要安装lldb,lldbvm,netcore sdk自带了sos插件 只需要使用的时候加载就行。下面进行一个简单的分析实例：

1.根据dockerfile创建基础镜像
  docker build -t lldb39-core215 -f url
  之所以要创建这个镜像 ，因为我们分析dump文件的时候 需要安装一堆插件  而且都很大，每次安装很不方便。
  
2.根据我们需要的docker镜像去生成coredump 文件，一般默认没有权限创建，我们需要修改 docker run --privileged=true
然后-->https://github.com/dotnet/coreclr/blob/master/Documentation/botr/xplat-minidump-generation.md#configurationpolicy

每一个core sdk自带了createdump 进程函数
先 docker exec -it dockername /bin/bash ->/usr/share/dotnet/shared/Microsoft.NETCore.App/{sdk-version}/createdump pid

注意：pid是当前进程基于docker的进程id，而不是docker容器基于宿主机的pid

3.我们将得到的coredump 从docker 容器拷贝 到要分析的宿主机
  docker cp dockername:/tmp/coredump /cnl
  
4.拉取安装了core sdk 的lldb镜像，并跑起来
docker run -d -v /cnl:/dump --rm -it --name dockername dockerurl /bin/bash

5.上面都是所有的准备工作，下面就是我们分析工作开始
  进入lldb39 docker容器环境
  docker exec -it dockername /bin/bash
  然后开始加载coredump
 lldb-3.9 dotnet -c /dump/coredump.1 -o "plugin load /usr/share/dotnet/shared/Microsoft.NETCore.App/{sdk-version}/libsosplugin.so"
 
 注意：这里我发现了一个问题，就是我们lldb基础镜像打包自带的core sdk必须要和coredump打包的sdk版本一致
 
 详细细节->
 https://github.com/dotnet/coreclr/blob/master/Documentation/building/debugging-instructions.md
 
6.就是好好学习sos命令的时候啦啦啦啦
 
