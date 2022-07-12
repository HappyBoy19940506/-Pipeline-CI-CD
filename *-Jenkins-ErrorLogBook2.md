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
      
这里面没办法指定agent，所以只能在前面指定在哪个agent运行。 你如果在docker运行的话 也没必要clean，因为它会自动删掉容器的。      
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
# 16. Input in Jenkins
## Resolved: 
```
1.  先说说层级  pipeline{}-  envirnoment{} agent{} stages{}post{} -- 在stages{}里面: stage{} stage{}... --在stage{}里面 steps{ echo ..} input{}
   -所以说，input{} 所处的层级时和 steps一样的层级，在 stage{}的下面。
    *** 并且 input{} 标签 和 when{}一样，不能单独存在于stage{}里，stage{}里面必须要有steps{}
2.  此种 input写法 有什么问题？
    pipeline {
    agent any
    stages {
        stage('input') {
        steps {
        sh '''
          hostname
          cat /etc/redhat-release
        '''
      }
        input {
         message 'ready to go?'
        }
    }
  }
}

---> input 在 agent any中 执行，占用一个executor名额，并一直在waiting，这样后面有任务过来会 堵塞。

3。 所以 这样写比较好： 让 input 在agent none中执行。 然后在 stage阶段 再指定具体agent，这样的话pipeline阶段不指定agent，就不会产生堵塞，在你input{}之后才会执行
对应的agent. 补充一点： stage里面 的input 和 when，无论写在step前后，都是在step之前执行的。并且，一个stage中只能有一个steps！！！！

    pipeline {
    agent none
    stages {
        stage('input') {
        agent any
        steps {
        sh '''
          hostname
          cat /etc/redhat-release
        '''
      }
        input {
         message 'ready to go?'
        }
    }
  }
}
-- 如果global写agent2，但是stage写none，是 不起作用的。
4. 和 when{}一起配合使用， when+input +steps
 - when 需要写在input前面，  -when 中必须要有branch 'xxx'条件，否则会报错。
- Example 21. beforeInput
pipeline {
    agent none
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                beforeInput true
                branch 'production'
            }
            input {
                message "Deploy to production?"
                id "simple-input"
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}

```
https://www.jenkins.io/doc/book/pipeline/syntax/#input
https://www.youtube.com/watch?v=QD6vhbadRJs&t=444s
----------------------------------------------------
----------------------------------------------------
# 17. when{...} Empty when closure, remove the property or add some content. @ line 10, column 13.
              
## Resolved: 
```
when 中必须要有branch 'xxx'条件，否则会报错。
*********   
when {
                beforeInput true.  
                branch 'production' --必要项
                environment name: 'DEPLOY_TO', value: 'production'
                
            }
 *********              
 when {
                allOf {
                    branch 'production'
                    environment name: 'DEPLOY_TO', value: 'production'
                }
            }
*********               
when {
                branch 'production'
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
*********           
when {
                expression { BRANCH_NAME ==~ /(production|staging)/ }
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            

```
https://www.jenkins.io/doc/book/pipeline/syntax/#when
----------------------------------------------------
----------------------------------------------------
# 18. Set a Timeout in a Jenkins Pipeline?

## Resolved: 
```
     options {
        timeout(time: 10, unit: 'SECONDS') 
      }
---- 和input/steps/when同一层级

  options {
    timeout(time: 7, unit: 'SECONDS') 
  }
 或者放在 pipeline{}之下，和 agent tools stages 同一级别 - 相当于给整条build计时，超时就报错
 
 --- input一定要配合timesout， global如果时间 小于 stage的timeout时间， 那么global到时间就直接结束了
 
```
https://www.youtube.com/watch?v=OChOtpK0fUE
----------------------------------------------------
----------------------------------------------------
# 19. unzip awscliv2.zip报错 replace aws/THIRD_PARTY_LICENSES? [y]es, [n]o, [A]ll, [N]one, [r]ename:  NULL
(EOF or read error, treating as "[N]one" ...)
Archive:  awscliv2.zip
replace aws/THIRD_PARTY_LICENSES? [y]es, [n]o, [A]ll, [N]one, [r]ename:  NULL
(EOF or read error, treating as "[N]one" ...)
## Resolved: 
```
-q  quiet mode (-qq => quieter)
-o  overwrite files WITHOUT prompting  
-f  freshen existing files, create none
-n  never overwrite existing files 

According to http://www.manpagez.com/man/1/unzip/ you can use the -o option to overwrite files:

unzip -o /path/to/archive.zip

Note that -o, like most of unzip's options, has to go before the archive name.


```
--------------------------------------------------------------------------------------------------------
# 20. 宇宙无敌之Nested Stages,套娃解决 Running Multiple Stages with the Same agent,

## Resolved: 
```
假设现在有如下情景：
1. 你需要input来进行确认，为了在等待期间 不造成堵塞，你必须使用global的 agent none
2. 如此，你需要在stages-stage level中使用 agent 1.2.3
3。 正好此时你需要用 agent -docker，出现的问题是： 如果你 每个stage中都写上agent-docker的话，在每个stage会独立创建单独的container，并不是在同一个容器内。
    你只能把所有原先的stage的步骤 写到  1个stage的steps里面。
4. 如何解决 ？？？
  -用nested stage， 也就是 正常结构的 stages -[stage -steps]-[stage-steps],可以变成 stages -[stage -(这里加一套完整的pipeline结构，比如agent，stages，post）]-[stage-steps]
  - 到达的效果： 在不同的stage内，运行同一个agent，如此，普通agent不需要挨个写上，docker-agent可以用同一个container。
  
  pipeline {
    agent none
    stages {
        stage("build and test the project") {
            agent {
                docker "our-build-tools-image"
            }
            stages {
               stage("build") {
                   steps {
                       sh "./build.sh"
                   }
               }
               stage("test") {
                   steps {
                       sh "./test.sh"
                   }
               }
            }
            post {
                success {
                    stash name: "artifacts", includes: "artifacts/**/*"
                }
            }
        }
    }
}

```
https://www.jenkins.io/blog/2018/07/02/whats-new-declarative-piepline-13x-sequential-stages/#running-multiple-stages-with-the-same-agent-or-environment-or-options
----------------------------------------------------
----------------------------------------------------
# 21. when{condition}老是返回false导致skipped stage

## Resolved: 
```
                    when {
                       allOf {
                             branch 'main' //问题所在
                             environment name: 'PROJECT_ENV', value: 'Prod'
                             environment name: 'PROJECT_NAME', value: "${PROJECT_NAME}" 
                            }
                         beforeInput true  //在input block之前执行
                    }
                    
 问题出在 branch‘main’上。
 branch
    Execute the stage when the branch being built matches the branch pattern (ANT style path glob) given, for example: when { branch 'master' }. Note   that this only works on a multibranch Pipeline.
    -只支持 多branch 的pipeline，我的pipeline只监听一个branch，所以怎么读都爆错。 要用这个必须 是 multibranch pipeline - blue ocean
    
```
----------------------------------------------------
----------------------------------------------------
# 22. script{ def params = 'ss'} 传不出去。 在script外面 无法调用。

## Resolved: 

```
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                script{
                    def HAHA = true
                    echo "${HAHA}"
                }
                
                
            }
        }
    }
}
------ 可以echo，因为在script里面
pipeline {
 agent any
    stages {
        stage('Hello') {
            steps {
                script{
                    def HAHA = true
                }
                   echo "${HAHA}"
                
            }
        }
    }
}
----echo不出来， haha读不到。

```
----------------------------------------------------
----------------------------------------------------
# 23. how to call parameters name with whitespace,like name: 'How are you doing?'

## Resolved: 

```
You can use params['IP Address'].

Think of params as a Map containing a key 'IP Address'. 

If the key didn't have a space, then you could use 

params.IPAddress 
or 

params['IPAddress'], 

but when there's a space, you can only use the latter syntax.

- 注意点1 params 不是 param，别老是少个s！！！
- 注意点2 空格的形式是 params['string'] params后面没有.  不是params.NAME的形式
- 注意点3 在when{}进行判断时，environment判断是：  environment name: 'PROJECT_NAME', value: "${PROJECT_NAME}"
                           而parameters的判断是 : expression{ params['Is a back-up bucket considered ?'] == true }
- 注意点4 判断时，注意是 == 而不是 = ！！！！
- 注意点5 判断的值 除了true和false 别的都要加 引号！

```
----------------------------------------------------
----------------------------------------------------
# 24.when{} 的一些用法和注意事项，when的意思就是可执行or不执行，没when的stage就一定要执行，所以记住，when不能用在必需的stage上，只能用在可有可无的stage上，条件通过就run，不通过也不报错，只是skip，所以千万不能用在 必要的stage上

## Resolved: 

```
1. 可以用来skip一些stage - 在when中写条件，如果满足，就执行该stage，否则跳过，但是并不影响后面的stage运行，比如when条件是prod，就执行stage中的cd

2. 可以用来做一些 parallel的stage，比如同时传向2个bucket，或者2个平台，互相之间不影响。 你可以在when中判断 输入的parameters来决定是否在此stage中执行还是跳过

3. when 和input 同时用： when中可以添加 beforeInput :true来决定顺序， input是用来pause然后让人决定继续还是放弃的。
  - 在input之前，when中判断是否应该在此环境下 提出input让人来决定
  - 在input之后  input让abort就 直接整条ppl放弃，如果proceed，那就开启when判断 再决定此stage要不要执行。
  - input中也可以带parameters，并且在when中做判断。详见#25

    when {
        allOf {                        
        // branch 'main'  ---- Only Support Multibranch Pipeline
        environment name: 'PROJECT_ENV', value: 'Prod'  -- pipeline中定义的env
        environment name: 'PROJECT_NAME', value: "${PROJECT_NAME}" -- pipeline中定义的env
        expression{ params['Which Environment are you going to deploy?'] == 'Production' } -- pipeline中定义的params,expression{ }判断是否为true
        
        environment name:'Which Environment are you going to deploy? , value:‘Production'  --这个很特殊，是input定义的params，但是这里必须用envionment                            的syntax，非常诡异
        
        }
        beforeInput true 
        }
                        
```
----------------------------------------------------
----------------------------------------------------
# 25.input { } 打断整条ppl，等待输入，一定配合timeout使用

## Resolved: 

```         options{
            //timeout
            }
            input {
                message "Should we continue?"
                ok "Yes, we should."
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                    booleanParam defaultValue: true, description: 'I\'ve checked all resources are setting well.', name: 'Check time'
                     choice choices: ['Production', 'Staging'], description: "Production", name: 'Which'                  
                }
```
### 这里特别要小心的是， 这个parameters选项，虽然叫parameters，但是要想在input{}外引用，必须用env方式， 即为 ${env.PRESON} ${env.Which} 如果在when{}中引用，则为environment name:'Which Environment are you going to deploy? , value:‘Production'  ,且 ${env.PRESON}中name不能带space，不能想parameters那样写成['sss']的形式



----------------------------------------------------
----------------------------------------------------
