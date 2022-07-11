# 1. Pipeline中，使用agent docker{image xxxx}，报错 docked deamon permission denied.

```
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```
## Resolved: 
```
方法一： 不推荐，只是降低了了文件的访问许可门槛，让张三李四都能访问。 安全隐患 + reboot会还原。
just open terminal and type this command // 在运行docker的那个agent里面 ，给 docker daemon抬旗， 直接最高权限运行

sudo chmod 666 /var/run/docker.sock

chmod 666 file/folder means that all users can read and write but cannot execute the file/folder;
chmod 777 file/folder allows all actions for all users;
chmod 744 file/folder allows only user (owner) to do all actions; group and other users are allowed only to read.


方法二： 权限还是660，不提权，但是增加 对应的group以及 user。

 - 创建的group （必须加起码一个user，不然 sudo groups不显示，只会提示 already exist）。加入user等操作，重启后不会消失
 - 所以 permission denied的问题最好通过 groups解决，也规避了security risk。
 - 一个程序，要么root访问，要么在其对应的groups里面的user可以访问 -660权限。

  If you want to run docker as non-root user then you need to add it to the docker group.

  Create the docker group if it does not exist
  
  $ sudo groupadd docker
  
  Add your user to the docker group.
  
  $ sudo usermod -aG docker $USER
  -这里的$USER, 好巧不巧的是，你Jenkins的agent username是 ubuntu，而你ssh练的时候正好也是 ubuntu，所以uubuntu就可以用docker了。如果你用比如 Jenkins-local的话，
  那你需要把 “Jenkins” 加入到group中！ 

  记得 $reboot --重启ec2， groups操作要重启才能生效， 但是chmod操作立即生效。
```
https://www.digitalocean.com/community/questions/how-to-fix-docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket




# 2.   运行command权限不够要sudo， sudo了又要密码。我ssh连接的，哪来的密码？？

```
pipeline {
agent {
        label'local'
}
    stages {
        stage('Hello') {
            steps {
                sh 'whoami'
                ----> Jenkins 
                sh 'apt-get update'
                sh 'apt install python3-pip -y'
                sh 'pip3 install awscli --upgrade'
                ----> Reading package lists...
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
W: Problem unlinking the file /var/cache/apt/pkgcache.bin - RemoveCaches (13: Permission denied)
W: Problem unlinking the file /var/cache/apt/srcpkgcache.bin - RemoveCaches (13: Permission denied)
            }
        }
    }
}
权限不够 --> 抬旗 +_sudo
pipeline {
agent {
        label'local'
}
    stages {
        stage('Hello') {
            steps {
                sh 'whoami'
---------------> jenkins  or unknown userid 1312312
                sh 'sudo apt-get update'
                sh 'sudo apt install python3-pip -y'
                sh 'sudo pip3 install awscli --upgrade'
--------------> + sudo apt-get update
sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper
            }
        }
    }
}

```
## Resolved: 
```
+ args， 以root身份执行jenkinsifle
pipeline {
agent {
    docker {
        image 'node:12.16.1-alpine'
        label 'local'
        args  '-u root:root'
    }
}
    stages {
        stage('Hello') {
            steps {
                sh 'whoami'
                --> root
                sh 'node -v'
                ---> 12.16.1
            }
        }
    }
}
```
# 3. 各种command not found

```
xxx command not found
```
## Resolved: 
```
1. 考虑所处的agent。 这台agent上 装了什么？有什么环境？

2.考虑是不是在docker +image的环境下，这种container里 有什么环境？

3. 如果确实装了，但就是 读不出来， 请移步问题12， 有可能是用户路径问题，你在a用户下装的命令，b用户下读不出命令，建议走问题12 global tool方法。
```

# 4. react deploy to S3 index.html显示空白的问题
## Resolved: 
```
npm run build生成build文件夹后，
1. 在本地 index.html空白问题。 在打包之前,在package.json中private下(位置任意)添加"homepage": "./" ，再重新build一下。
{
  "name": "bookinglet",
  "homepage": "./", <----------------------
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "bootstrap": "^5.2.0-beta1",
    "react": "^18.1.0",
    "react-dom": "^18.1.0",
    "react-scripts": "5.0.1",
    "web-vitals": "^2.1.4"
  },

2. S3上 hosting的 index.html显示空白 -->
*不能上传build文件夹，而要把里面的 所有文件上传。你想啊，一个用户输入网址，www.xxx.com 不会输入www.xxx.com/homepage 这样，所以你一旦进入www.xx.com就相当于来到了root directory了，你得有个东西招待他呀，那root里面肯定要有个index.html啊，不然当然是光的了。 S3 static hosting里面问你填index.html问的是你主页name叫什么，而不是主页的path是什么。


*(???存疑）如果上传的是文件夹，那么s3给你的url是空白页，请去cloudfront中设置origin default path,把它设置成 ./build/index.html。 并且origindomianName选择s3的website entrypoint而不是 s3本身的地址。



https://www.freecodecamp.org/news/how-to-host-and-deploy-a-static-website-or-jamstack-app-to-s3-and-cloudfront/#storing-your-website-on-s3

```

# 5. multiple-agent-labels-in-a-declarative-jenkins-pipeline

## Resolved: 
```
docker
agent {
    docker {
        image 'maven:3.8.1-adoptopenjdk-11'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
}


agent {
    docker {
        image 'myregistry.com/node'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
dockerfile

agent {
    // Equivalent to "docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        additionalBuildArgs  '--build-arg version=1.0.2'
        args '-v /tmp:/tmp'
    }
}
dockerfile

agent {
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
kubernetes

agent {
    kubernetes {
        defaultContainer 'kaniko'
        yaml '''
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: aws-secret
        mountPath: /root/.aws/
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  volumes:
    - name: aws-secret
      secret:
        secretName: aws-secret
    - name: docker-registry-config
      configMap:
        name: docker-registry-config
'''
   }
   
url ->
https://www.jenkins.io/doc/book/pipeline/syntax/
```



# 6. Jenkins pipeline how to change to another folder

```
Jenkins pipeline how to change to another folder
```
## Resolved: 
```steps {
    sh "pwd"
    dir('./your-sub-directory/dir') {
      sh "pwd"
    }
    sh "pwd"
} 
<!-- dir后面的路径是按源根目录的root来算的路径，而不是 jenkinsifle所在的路径为根目录 -->
<!-- 本来前端后端就不应该放在一个repo里面 -->
```


# 7. Jenkins build fails with "Treating warnings as errors because of process.env.CI = true"
```
[ERROR] {some text}: {some text} is outdated. Please run next command `npm update`
[INFO] Treating warnings as errors because process.env.CI = true.
[INFO] Most CI servers set it automatically.
Failed to compile.
```
## Resolved: 
```
现在大部分项目都会支持ci工具来做持续集成，所以现在默认值为ci=true，这样让大部分的library会按照ci=true的环境来运行.而不是local环境。
但是有些library会因此报错，而且会halt build导致fail to compile。
1. jenkinsifle 中 
 environment {
        CI= 'false'
    }
2. 在 jenkins ui中，设置global config ->
-You can set CI env variable to false through Manage Jenkins > Configure System > Global properties section.
-Add a new env variable CI with the value false.
3. 在package.json中更改，改成默认为 false:
  "scripts": {
    "start": "react-scripts start",
    "build": "CI=false && react-scripts build",  // Add CI=False here
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },

```
# 8. fatal: couldn't find remote ref refs/heads/master

```
Started by user root
hudson.plugins.git.GitException: Command "git fetch --tags --force --progress --prune -- origin +refs/heads/master:refs/remotes/origin/master" returned status code 128:
stdout: 
stderr: fatal: couldn't find remote ref refs/heads/master

	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandIn(CliGitAPIImpl.java:2671)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:2096)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.access$500(CliGitAPIImpl.java:84)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl$1.execute(CliGitAPIImpl.java:618)
	at jenkins.plugins.git.GitSCMFileSystem$BuilderImpl.build(GitSCMFileSystem.java:366)
	at jenkins.scm.api.SCMFileSystem.of(SCMFileSystem.java:197)
	at jenkins.scm.api.SCMFileSystem.of(SCMFileSystem.java:173)
	at org.jenkinsci.plugins.workflow.cps.CpsScmFlowDefinition.create(CpsScmFlowDefinition.java:118)
	at org.jenkinsci.plugins.workflow.cps.CpsScmFlowDefinition.create(CpsScmFlowDefinition.java:70)
	at org.jenkinsci.plugins.workflow.job.WorkflowRun.run(WorkflowRun.java:311)
	at hudson.model.ResourceController.execute(ResourceController.java:101)
	at hudson.model.Executor.run(Executor.java:442)
Finished: FAILURE
```
## Resolved: 
```
Master 现在叫main 了
```



# 9. Why there is no Source Code Management tab in a Jenkins pipeline job?
## Resolved: 
```
FreeStyle里面有soure code management来选定code来自哪个git-repo
Pipeline里面其实也有，但是不叫Source Code Management,直接在选定jenkinsfile script form SCM下啦菜单的Repository URL就已经是该repo地址，因为他
默认 jenkinsfile go with this repo together.

-Script Path :指定 jenkisnfile在哪，有可能不在root，那你就要写 relative-path ,e.g.  ./cicd/jenkinsfile 

-Branches to build : List of branches to build. Jenkins jobs are most effective when each job builds only a single branch. 不推荐 a single job builds multiple branches. 一个pipeline也就是一个job，一般只允许一个branch，规避multiple branch。


git code ==等价于 
在input script中写:
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/jglick/simple-maven-project-with-tests.git'
            }
	   }
```

----------------------------------------------------

# 10.  每一个pipeline都是一个job，每次build 并不会清空之前的文件夹

## Resolved: 
```
create 一个新的pipeline的时候，会创建一个新的workspace，每次build的时候 比如会git clone repo的代码， 后面build如果不涉及重新clone 代码的话，代码还会存在，并不是每次build都重新起一个线程的， 这也就是为什么，你修改了config之后，明明没有任何get code from Github的操作，但是你还是能运行npm install等操作，他install的上次download下来的repo code
```
----------------------------------------------------

# 11.  WebHook doesn't work on first-time build in the pipeline

## Resolved: 
```
如果是新建的pipeline，新建时候设置了trigger webhook，并且github settings也设置了，第一次build 如果是auto webhook的话不 生效，也就是说 要手动build一次，后面才会自动hook触发。 并且要手动build 起码成功1次， webhook才会生效。

原理： webhook只会发送post请求给jerkins server，但jenkenis并不知道是哪条pipeline会执行这条post，要手动build第一次，养成良好习惯。

```
----------------------------------------------------

# 12.  jenkins agent中明明装了node js， 但是pipeline中缺显示报错 node command not found / Usage : Global Tool Configuration

```
master server -- whoami --jenkins --- 如果在server上装了node --- can find command
slave server -- whoami -- ssh-username ---如果在server上装了node --node command not found
症状： -在jenkinsfile中如果执行whoami，在master- node中显示Jenkins，在slave-node中显示 ssh-username，
      -在ssh界面执行 whoami，在master-node中显示ssh-username,在slave-node中显示 ssh-username.
      -并且如果在ssh界面下安装了node，那么在jenkensifle中，master-node会显示 can find node, slave-node会显示can not find node.


根本原因是 user的问题，你如果用agent运行，你默认的用户名为 你ssh的用户，也即为你server上安装nodejs的用户， 但是 pipeline中其实去找的用户名为 Jenkins，寻找名为Jenkins名下的nodejs文件。
```
## Resolved: 
```
Global Tool Configuration 帮你以Jenkins身份运行各版本，因为运行的是 

When we use nodejs for first time by referring to this name and run the job, jenkins will automatically download the mnodejs tar file, extract and save it in /var/lib/jenkins/tools/ folder

From the next build it will use the nodejs from that folder, it won’t download everytime

```
### 也可以看作如下场景的work around： 你只有一个server或者agent，要在上面跑 不同版本的 nodejs,如何控制？？
```
https://devopspilot.com/content/jenkins/tutorials/common/03-global-tool-configurations

tools {
    nodejs "nodejs-14.14.0"
}
tools {
    nodejs "nodejs-18.0.0"
}
```

----------------------------------------------------

# 13. How to create a Jenkins slave agent ?

```
https://www.youtube.com/watch?v=-NUQhwmhTCw

https://www.youtube.com/watch?v=99DddJiH7lM
```
## Resolved: 
```
1. 打开master和slave的22port
2. 随便哪个地方创建一堆密钥: ssh-keygen  filename比如叫 jenkins-ssh-key 
3. 私钥 上传至Jenkins credentials， kind选择ssh username with private key, id随便叫 会显示在后面的keychain里面，就是credentials的名字，username随便叫，会显示在后续的选择下拉框里面，
4. **************公钥存在 slave上， vi ~/.ssh/authorized_keys ,把公钥加入， wq保存退出***********老是忘记！
5.  Master controller上要装java, 再装jenkins, slave node上要装 java ！！！不然读不到remoting.jar
5.  manage nodes and clouds -- new node --往下一步步填，特别注意Remote root directory项
```
----------------------------------------------------

# 14.  [SSH] WARNING: SSH Host Keys are not being verified. Man-in-the-middle attacks may be possible against this connection.

## Resolved: 
```
一般选择 Non Host Key Verification Strategy , 如果要 去掉该warning， 参考 
14:45 开始
https://www.youtube.com/watch?v=99DddJiH7lM

大致思路： 报错是因为这个ssh key的ownership不是Jenkins，所以没有verify的。 我们要把 private key 复制粘贴到 Jenkins的 .ssh目录下，/var/lib/jenkins/.ssh 并且要让Jenkins拥有该ssh目录的ownership.

```
----------------------------------------------------

# 15. [SSH] Remote file system root /agent2 does not exist. Will try to create it...
# java.io.IOException: Could not copy remoting.jar into '/agent2' on agent
# Caused by: com.trilead.ssh2.SFTPException: Permission denied (SSH_FX_PERMISSION_DENIED: The user does not have sufficient permissions to perform the operation.)

## Resolved: 
```
该问题 属于 16.Remote root directory问题

```
----------------------------------------------------
# 16. Remote root directory

```
如果填 相对路径，默认前置为 /home/ssh-username/
比如填写: data -->  /home/username/data 相对路径 会在user目录下生成data文件夹
	/data --> /data 绝对路径，会直接跑root目录下生成文件夹，user都超过权限了管不了了 。这种是会报错的！
	./data -->/home/username/data 相对路径 会在user目录下生成data文件夹
	/home/username/data -->相对路径，一步到位，会在user目录下生成data文件夹

所以如果你填错了 root directory ，你的ssh-user是没用权限 操作root目录下的文件的，就会报15. 里面的错误。

巧记： 开头带上了 "/" , 就是绝对路径 ，就要填 /home/fxy4560654这种，否则必报错。
```
## Resolved: 
```
```
----------------------------------------------------
# 17.  # of exectuors 

## Resolved: 
```
1. 一般 Jenkins controller 上不放 exectuors，有安全风险。
2. 一般 agent上放几个exectuors，取决于额有几个cpu或者vcpu,数量 小于等于。
```
----------------------------------------------------
# 18. 如何使用 smtp服务在 jenkinsfile中发送cicd结果

## Resolved: 
```
1. 在SES中申请 邮箱的identity， 并且邮件验证 ，获得stmp服务器dns， ---> 下面aws会帮你用fxy@gmail.com的邮箱来发送消息。
2.去Jenkins里设置admin邮箱为： System Admin e-mail address -> Jenkins Do Not Reply <fxy4560654@gmail.com> 
   前面一部分 是说 发送的时候的用户名，比如 xiaoyu FAN<fxy456@gmail.com>，后面一部分填写你 stmp邮箱 - 相当于 你现在使用这个stmp邮箱在Jenkins上用aws来发邮件。
3.设置extend email notification 各种 TLS, port 465, 以及 credentials等等，
4. 这个credentials要去aws ese和iam中单独创建，并不是aws本身的access key


https://www.youtube.com/watch?v=CGSwDpDfEMw
https://www.youtube.com/watch?v=7y-2MzSjjeo
https://www.youtube.com/watch?v=g2TSOIKzVUI&t=768s
https://www.youtube.com/watch?v=OOCvCdZLAhc
```
----------------------------------------------------
# 19. 如何在echo中引用变量${var}
## Resolved: 
```
要用双引号，e.g. echo"you have successfully passed the testing process: \n project name:${PROJECT_NAME}"

单引号没用的 .
```
----------------------------------------------------
# 20. echo中如何换行?

## Resolved: 
```
 echo"you have successfully passed the testing process: \n project name:${PROJECT_NAME}"
```
----------------------------------------------------
# 21. Jenkinsfile中的env.variable变量引用。

```
您可以在通过env对象的管道步骤中访问环境变量，例如，env.BUILD_NUMBER将返回当前的内部版本号。

您也可以使用简写版本BUILD_NUMBER，但是在此变体中，这可能会使某些用户感到困惑-它缺少BUILD_NUMBER来自环境变量的上下文。

```
## Resolved: 
```
pipeline {
    agent {
        label '!windows'
    }

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                echo "Database engine is ${DB_ENGINE}"
                echo "DISABLE_AUTH is ${DISABLE_AUTH}"
                sh 'printenv'
            }
        }
    }
}

1. environment里定义，下面 ${}调用
2.要用双引号

https://blog.51cto.com/devopsvip/3183551

```
----------------------------------------------------
# 22. Jenkinsfile中的parameters用法。

## Resolved: 
```
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"

                echo "Biography: ${params.BIOGRAPHY}"

                echo "Toggle: ${params.TOGGLE}"

                echo "Choice: ${params.CHOICE}"

                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
https://devopscube.com/declarative-pipeline-parameters/

jenkinsfile Parameter Best Practices
The following are some of the best practices you can follow while using parameters in a Jenkinsfile.

Never pass passwords in the String or Multi-line parameter block. Instead, use the password parameter of access Jenkins credentials with credential id as the parameter.

Try to use parameters only if required. Alternatively, you can use a config management tool to read configs or parameters in the runtime.

Handle the wrong parameter execution in the stages with a proper exception handling. It avoids unwanted step execution when a wrong parameter is provided. It happens typically in multi-line and string parameters.

```
----------------------------------------------------
# 23.  关于ci的一些观念改变。

```
以前我以为ci是在merge之前 也就是 pull request创建的时候 开始做的，但是一细想不对啊， 他还没有代码push更新，跑ci有什么用？
而且这也不符合ppline的逻辑。
```
## Resolved: 
```
1. 就是 push之后才会触发ci ， dev分支的代码就是会在push或者merge完成后才会触发 dev的ci，

如果CI失败，则所有人应该分析错误， 相关人员应该拉取一个修复分支（相当于feature分支），进行修复并提交代码，合并到develop分支，直到develop分支ci变绿，变绿之前，所有人不能合并其他任何feature分支。

如果CI成功，则相关的节点会有相关的发布包被push到二进制仓库，并且打上 “snapshot-commit号“标签。

比如：今天打开dev-branch发现3个merge request，第一个 reivew了，没问题，通过merge，然后ci触发，显示成功，没问题
第二个，review了，没问题，merge进去之后，发来log，报错了，这时候应该 解决这个ci问题。
第三个pr，就晾着，别来添乱。你如果再加入第三个就乱了，也不知道是第二个错还是第三个错了。
	
2. Feature分支上的持续集成
开发者在feature分支上push代码后执行， 包括：代码质量检查， 代码静态扫描，单元测试，程序编译。

如果CI失败，feature分支上的相关程序员需要修复相关的问题，如果上次提交的pipeline没有执行完成，开发者提交了新的代码，则老的pipeline停止， 新的pipeline包括了上次的所有代码提交的持续集成检查。

如果CI成功，则可以发次MR进行代码review。 代码review的时候，review人员可以参考Feature分支上的CI结果。

具体操作：-------------------->

git 插件的 Branch Specifier (blank for 'any') 一般默认写的是*/master 是部署的 master 分支，可以改成对应的分支就能部署分支了

分支选择里填 */feature/*,然后 CI/CD 会拉取最近的一次提交

<Wildcards>
The syntax is of the form: REPOSITORYNAME/BRANCH. In addition, BRANCH is recognized as a shorthand of */BRANCH, '*' is recognized as a wildcard, and '**' is recognized as wildcard that includes the separator '/'. Therefore, origin/branches* would match origin/branches-foo but not origin/branches/foo, while origin/branches** would match both origin/branches-foo and origin/branches/foo.
------------------>

https://www.v2ex.com/t/523861

https://blog.csdn.net/xiphi_6/article/details/122968094

https://testerhome.com/topics/27657?order_by=like&


```
----------------------------------------------------
# 24. EC2 上装Jenkins

```
(https://github.com/HappyBoy19940506/JR-DevOpsNotes/tree/master/WK4_Jenkins/Wk4.2/1.Jenkins-Installation-on-Ubuntu)
```
## Resolved: 
```
1. java
2. jenkins
3. systemctl status jenkins
4. sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```
----------------------------------------------------
# 25. 初始化jenkins要做的一些准备 

```

```
## Resolved: 
```
1. security group 开放 20 ，8080，50000端口
2. plugin
   1. CloudBees AWS Credentials + Pipeline:AWS STEPS
   2. Amazon Web Services SDK :: All
   2. docker for pipeline
   3. nodejs plugin - used for global configuration 顺便去设置一下版本哈！
   4. blue ocean 
   5. Email Extension Plugin
	
3. agent -go to #13
  ( ssh-keygen -t rsa )

4 . docker ( if you are on agent) -
	https://www.simplilearn.com/tutorials/docker-tutorial/how-to-install-docker-on-ubuntu
	

```
----------------------------------------------------

