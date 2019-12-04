---
type: Container
title: Dockerfile常用命令
date: 2019-12-04
category: Container
description: Dockerfile常用命令
---




## Dockerfile


- [Dockerfile](#dockerfile)
  - [Docker image build functions](#docker-image-build-functions)
  - [Dockerfile](#dockerfile-1)
      - [Dockerfile Format](#dockerfile-format)
  - [Dockerfile Instructions](#dockerfile-instructions)
### Docker image build functions

1. BaseImage(DockerCommit)
2. Dockerfile(env)


### Dockerfile 

Dockerfile is nothing but the source code of building Docker images

##### Dockerfile Format

- Format
    - \# Comment
    - Instruction arguments
    - instructions in a Dockerfile in order
    - The First instruction must be `FROM` in order to specify the base image 

- Dockerfile
    - .dockeringore(ingore some files)
    - docker build
        - make a container
        - run argements on container
        - commit 
    - docker tag
    - docker push
- Environment replacement
    - Environment variables (declared with the `ENV` statement) can also be used in certain instructions as variables to be interpreted by the Dockerfile
    - Environment variables are notated in the Dockerfile either with `$variable_name` or `${variable_name}`
    - the `${variable_name}` syntax
        1. `${variable:-word}` : if variable was been setted then the result will be that value, if the value was not been set,the `word` will be the result(set the default value)
        2. `${variable:+word}` : if the variable was been setted then the word will be the result, otherwise the result is the empty string


### Dockerfile Instructions
- FROM
    - must be the first instruction
    - in fact, base image can be any image,default docker engine will  find it in the local machine,if not docker engine will find it in docker hub,if eihter not find, docker build will except an error
    - Syntax 
        - FROM <repository>[:tag]
        - FROM <repository>@<digest>(hash code)
            - <reposotiry>: base image name
            - <tag>: base image lable, default latest


- MAINTANIER(depreacted)
    - indent the dockerfile author
    - Syntax
        - MAINTAINER <author's detail>
        - MAINTAINER "luxsun<luxsun@163.com>"

- LABEL
    - indent metadata to an image

    - Syntax
        - LABEL <\key>=<\value><\key>=<\value>
        - A LABEL is a key-value pair
        - AN image can have more than one lable

- COPY
    - 复制文件到创建的image中

    - Syntax
        - COPY \<src>...\<dest>
        - COPY ["\<src>", ... "\<dest>"]
            - \<src>: 要复制的源文件或目录, 支持使用通配符
            - \<dest>: 目标路径,即正在穿件的image的文件系统路径;建议\<dest> 使用绝对路径,否则,COPY指定则以WORKDIR为其起始路径
        - 路径中有空白字符时,通常使用第二种格式

    - 文件复制准则
        - \<src>必须是build上下文中的路径,不能是其父目录中的文件
        - 如果\<src>是目录,则其内部文件或子目录会被递归复制,但\<src>目录自身不会被复制
        - 如果指定了多个\<src>,或在\<src>中使用了通配符,则\<dest>必须是一个目录,且必须以`/`结尾
        - 如果\<dest>事先不存在,它将会被自动创建,这包括其父目录路径

- ADD
    - ADD 指令类似COPY指令,ADD支持使用TAR文件和URL路径
    - Syntax
        - ADD \<src> ... \<dest>
        - ADD ["\<src>", ... "\<dest>"]
    - 操作准则
        - 同COPY指令
        - 如果\<src>为URL且\<dest>不以/结尾,则\<src>指定的文件将被下载并直接被创建为\<dest>;如果\<dest>以/ 结尾,则文件名URL指定的文件将被直接下载并且保存为\<dest>/\<filename>
        - 如果\<src>是一个本地系统上的压缩格式的tar文件,它将被展开为一个目录,其行为类似于"tar -x"命令,然而,通过URL获取到的tar文件将不会自动展开
        - 如果\<src>有多个,或其简介或直接使用了通配符,则\<dest>必须是一个以/ 结尾的目录路径;如果\<dest>不以 / 结尾,则其被视为一个普通文件,\<src>的内容将被直接写入到\<dest>中;

- WORKDIR
    - 用于为Dockerfile中的所有RUN,CMD,ENTRYPOINT,COPY和ADD指定工作目录
    - Syntax
        - WORKDIR \<dirpath>
            - 在Dockerfile中,WORKDIR可出现多次,其路径也可以为相对路径,不过,其是相对此前一个WORKDIR指令指定的路径
            - WORKDIR也可调用由ENV指定定义的变量
            - 例如
                - WORKDIR /var/log
                - WORKDIR $statepath

- VOLUME
    - 用于在image中创建一个挂载点目录,以挂载Docker host上的卷或其他容器上的卷
    - Syntax
        - VOLUME \<mountpoint>
        - VOLUME ["\<mountpoint>"]
    - 如果挂载点目录路径下此前文件存在,docker run 命令会在卷挂载完成后将此前的所有文件复制到新挂载的卷中


- EXPOSE
    - 用于为容器打开指定要监听的端口以实现与外部通信
    - Syntax
        - EXPOSE \<port>[/\<protocol>][\<port>][\/<protocol>]
            - \<protocol>用于指定传输层协议,可为tcp或udp二者之一,默认为tcp协议
    - EXPOSE指令可一次指定多个端口,例如:
        - EXPOSE 11211/udp 11211/tcp


- ENV
    - 用于为镜像定义所需的环境变量,并可被Dockerfile文件中位于其后的其他指令(如ENV,ADD,COPY)等调用
    - 调用格式为$variable_name或${variable_name}
    - Syntax
        - ENV \<key>\<value>
        - ENV \<key>=\<value> ... 
            - 第一种格式中,\<key>之后的所有内容均会被视作其\<value>的组成部分,因此一次只能设置一个变量
            - 第二种格式可以一次设置多个变量,每个变量为一个"\<key>=\<value>"的键值对,如果\<value>中包含空格,可以以反斜线(\)进行转义,也可通过对\<value>加引号进行标识;另外,反斜线也可用于续行.
            - 定义多个变量时,建议使用第二种方式,以便在同一层中完成所有功能.

- CMD