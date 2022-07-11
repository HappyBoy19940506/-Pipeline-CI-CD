----------------------------------------------------
# 1. nodejs- how to use Jest to test

## Resolved: 

```
1. install jest
    npm install --save-dev jest
    
    Using Babel
  npm install --save-dev babel-jest @babel/core @babel/preset-env
    
   Using TypeScript
  npm install --save-dev @babel/preset-typescript
  
2. npm run test
  "scripts": {
    "test": "jest"
  }

3. Code Coverage using Jest
   npm test – –coverage


https://blog.jmtalarn.com/running-node-js-react-tests-same-project/
https://www.browserstack.com/guide/unit-testing-of-react-apps-using-jest
https://juejin.cn/post/7049536297584558087
https://jestjs.io/docs/getting-started
```
----------------------------------------------------
----------------------------------------------------
# 2. Groovy语法中如何 引用变量？/ 单引号双引号三引号如何引用变量

## Resolved: 
```

字符串类型	字符串语法	可插值	可多行	转义字符
单引号	        '...'	  ❌	  ❌	     \
三重单引号	'''...'''	❌	 ✅	    \
双引号     	"..."	  ✅	  ❌	     \
三重双引号	""""..."""	 ✅	 ✅	    \
斜线	        /.../	  ✅	  ✅	      \
美元斜线	$/.../$	  ✅	    ✅	     $


可插值 的意思就是 可以在里面写 ${env}
可多行 的意思就是 回车键能否显示 ，是否需要 \n
```

----------------------------------------------------
----------------------------------------------------
# 3. Use aws steps Plugin to upload files to s3.

## Resolved: 
```
1. 安装pipeline：aws steps插件，这样 syntax就会出现 withAWS{ }
2. 设置常规三个变量：
        AWS_S3_CREDENTIALS = "AWS-Credentials-Root-AccessKey"
        AWS_S3_REGION = "ap-southeast-2"
        AWS_S3_BUCKET = "uat.happyboy.link"
    其中 第一个credentials变量的值 其实是另一个变量（也就是Jenkins credentials）的名字，最终都指向了 一对 access key和  secret key
    第二个 变量是区域， 第三个 bucket名字。
   -一些情况下，你可以在environment的block中写 AWS_DEFAULT_REGION =XXX, AWS_ACCESS_KEY =XXX, 来直接给该机子上的这些变量赋值，这样就可以相当于config了aws，
   -可以直接run aws命令了，但是这样坏处是暴露了密码，不安全。所以可以通过 withAWS -credentials的方法隐藏起来，不在jenkisifle中show出来，但是也可以赋上值。
   
3.               withAWS(credentials: "${AWS_S3_CREDENTIALS}", region: "${AWS_S3_CREDENTIALS}"){
            // "${AWS_S3_CREDENTIALS}" 和 "${AWS_S3_CREDENTIALS}" 用双引号加上variable（前面预设了），来调用，这样的话，方便以后更改，
            // 如果有变化，s3的步骤不变，只需要改变 三个aws变量的值就可以了，方便！
                    sh'aws s3 ls'  //用来检测是否access key有用
                    sh'pwd'
                    // 代码中带了dir{ }所以 此时目录为 ./front_end/
                    sh "aws s3 rm s3://${AWS_S3_BUCKET} --recursive"
                    //清空 s3
                    sh "aws s3 cp ./build s3://${AWS_S3_BUCKET} --recursive --acl bucket-owner-full-control"
                    //上传文件，依次，acl disable
                    //这种方式上传会把build文件夹下所以东西都依次上传，但是不带build folder本身，上传到 s3bucket的root 目录，方便你web hosting
                    //如果你想要 上传整个folder，你需要写成： sh "aws s3 sync ./build s3://${AWS_S3_BUCKET}/build"也就是说用sync，并且你要提前也在s3里面写上build
                    //文件夹
                        }
       
       -其实这里可写成  withAWS(credentials: "AWS-Credentials-Root-AccessKey", region: AWS_S3_REGION){ -不方便修改Jenkins的credentials
       -或者也可写成  withAWS(credentials: AWS_S3_CREDENTIALS, region: AWS_S3_REGION){   -有点晦涩，其实 "${x}"  在这里 可以直接写成 x
       -或者也可写成  withAWS(credentials: "AWS-Credentials-Root-AccessKey", region: "ap-east-1"){ -全部用"x"，不方便后期修改

```
----------------------------------------------------
----------------------------------------------------
# 4. clean workspace.

## Resolved: 
```
  post{
        always {
            cleanWs()
        }
      }  
```
----------------------------------------------------
----------------------------------------------------
# 5. 按照AWS CLI 2

## Resolved: 
```
注意， jr课件上的aws安装是错的，会装成 版本1 。我们应该装 版本2

1. 装 Linux x86（64bit） 很低概率装arm，x86占绝大部分市场
2.  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
3. 当然，你要保证 curl 和unzip 可以found，如果command not found，要安装。

    比如。如果你按照的是 ubuntu20.04 基本上自带了 curl命令，如果没有，自行apt安装。
    
    Update your Ubuntu box, run: sudo apt update && sudo apt upgrade
    Next, install cURL, execute: sudo apt install curl
    Verify install of curl on Ubuntu by running: curl --version
        
        但是unzip这个命令正常情况下是不带的，要自己安装
        https://www.tecmint.com/install-zip-and-unzip-in-linux/
        -sudo apt install unzip


https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
```
----------------------------------------------------
----------------------------------------------------
# 6. 在docker image: node 12.16.0-apline 或者 某些操作系统中 apt-get: not found

## Resolved: 
```

The image you're using is Alpine based, so you can't use apt-get because it's Ubuntu's package manager.

To fix this just use:

apk update and apk add

https://stackoverflow.com/questions/45142855/bin-sh-apt-get-not-found

只有在ubuntu环境下 才有apt命令
```
----------------------------------------------------
----------------------------------------------------
# 7. Jenkins  Email Notification报错：Not sent to the following valid addresses: 39xxxx196@qq.com 

## Resolved: 
```
收件人 和发件人 必须都处于 同一个aws account下的同一个 region的 identity 列表上 ，并且 verified。
```
----------------------------------------------------
----------------------------------------------------
# 8.  how to uninstall Docker on Ubuntu

## Resolved: 
```
有时候，也许需要删除docker，怎么卸载呢？
如果你是通过snap安装，
sudo snap disable docker
sudo snap remove --purge docker
如果你是正常安装apt：
https://www.youtube.com/watch?v=3vBBzf0P9D8

sudo apt-cache policy docker.io
sudo apt purge docker.io
sudo apt autoremove
```
----------------------------------------------------
----------------------------------------------------
# 9. how to install Docker on Ubuntu

## Resolved: 
```
https://snapcraft.io/install/docker/ubuntu

这个snap方法好，版本更新问题。

sudo apt update
sudo apt install snapd


sudo snap install docker

```
--------------------------------------------------------------------------------------------------------
# 10.  agent block也可以用在 单独的stage中， 比如

## Resolved: 
```
--放在 stage 和 step的 中间。

Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'gradle:6.7-jdk11'
                    // Run the container on the node specified at the
                    // top-level of the Pipeline, in the same workspace,
                    // rather than on a new node entirely:
                    reuseNode true
                }
            }
            steps {
                sh 'gradle --version'
            }
        }
    }
}

参考： https://www.jenkins.io/doc/book/pipeline/docker/
```
--------------------------------------------------------------------------------------------------------
# 11. 通过#10的思路，还可以在不同stage中运行不同docker的容器：

## Resolved: 
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent none
    stages {
        stage('Back-end') {
            agent {
                docker { image 'maven:3.8.1-adoptopenjdk-11' }
            }
            steps {
                sh 'mvn --version'
            }
        }
        stage('Front-end') {
            agent {
                docker { image 'node:16.13.1-alpine' }
            }
            steps {
                sh 'node --version'
            }
        }
    }
}
```
----------------------------------------------------
----------------------------------------------------
# 12. 通过#10的思路，还可以在agent运行 dockerfile而不是官方image

## Resolved: 
```
https://www.youtube.com/watch?v=Pi2kJ2RJS50

Jenkinsfile (Declarative Pipeline)
pipeline {
    agent { dockerfile true }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
                sh 'svn --version'
            }
        }
    }
}
```
----------------------------------------------------
----------------------------------------------------
# 13.  为什么我设置的chmod666等操作，重启ec2之后都还原了？但是 addgroup 操作没有还原。

## Resolved: 
```
 - Reboot =  STOP+ Start 操作上没有区别，
 -两者 都会发生 chmod的权限还原，但不会发生 groups的还原。
 -比如 docker.socket chmod666之后，如果重启 会丢失权限666，还原成660.
 -但是创建的group （必须加起码一个user，不然 sudo groups不显示，只会提示 already exist），加入user等操作，重启后不会消失
 -所以 permission denied的问题最好通过 groups解决，也规避了security risk。
 - 一个程序，要么root访问，要么在其对应的groups里面的user可以访问 -660权限。
 - 记得 $reboot --重启ec2， groups操作要重启才能生效， 但是chmod操作立即生效。

（如果真的想解决chmod会还原的话 --> 把自己那个用户加入到 dialout组 然后重启
sudo usermod -a -G dialout 你用户名字.  这样你在此用户下chmod操作不会丢失）
```
----------------------------------------------------
----------------------------------------------------
# 14. pipeline中 $docker stop --time=1 48030bf java.io.IOException: Failed to kill container '4803‘

## Resolved: 
```
-pipeline中 最后无法结束container，以及删除container

-在ssh的 terminal中也无法删除。 无法运行 stop kill 等命令，提示：Error response from daemon: Cannot kill container: 105: permission denied

docker重大bug，要删除原有版本， 然后使用snap重新安装到 最新的 Docker version 20.10.14, build a224086349 这个版本才会解决。

此外，千万不要用网上流传甚广的 apparmor 的方法，会删掉你所有的app！
```
----------------------------------------------------
----------------------------------------------------
# 15. ssh终端中 Error response from daemon: Cannot kill container: 105: permission denied

## Resolved: 
```
docker重大bug，要删除原有版本， 然后使用snap重新安装到 最最新的 Docker version 20.10.14, build a224086349 这个版本才会解决。

此外，千万不要用网上流传甚广的 apparmor 的方法，会删掉你所有的app！

```
----------------------------------------------------
----------------------------------------------------
# 16.

## Resolved: 
```

```
----------------------------------------------------
----------------------------------------------------
# 1. 

## Resolved: 
```

```
----------------------------------------------------
----------------------------------------------------
# 1. 

## Resolved: 
```

```
----------------------------------------------------
----------------------------------------------------
# 1. 

## Resolved: 
```

```
--------------------------------------------------------------------------------------------------------
# 1. 

## Resolved: 
```

```
----------------------------------------------------
----------------------------------------------------
# 1. 

## Resolved: 
```

```
----------------------------------------------------
