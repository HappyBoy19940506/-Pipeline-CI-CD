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
# 1. 

## Resolved: 
```

```
--------------------------------------------------------------------------------------------------------
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
