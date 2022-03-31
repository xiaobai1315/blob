Jenkins集成流水线

### 创建Jenkins流水线项目

* 流水线-定义选择Pipeline script from SCM，从远程仓库拉取Jenkinsfile文件

* SCM选择Git，输入远程仓库地址和GitHub凭证，选择分支

* 脚本路径输入Jenkinsfile，默认创建在工程的根目录

  

### 在idea 项目工程根目录创建Jenkinsfile文件

#### 从远程仓库拉取代码

```
stage('pull code') {
	steps {
		checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: 			  [[credentialsId: 'github', url: 'git@github.com:xiaobai1315/testJenkins.git']]])
		}
}
```



#### 使用Dockerfile编译、生成镜像、推送私有仓库

* 在每个微服务项目的pom.xml加入dockerfile-maven-plugin插件，注意：是插件，不是依赖

  ```
  <plugin>
       <groupId>com.spotify</groupId>
       <artifactId>dockerfile-maven-plugin</artifactId>
       <version>1.3.6</version>
       <configuration>
            <repository>${project.artifactId}</repository>
            <buildArgs>
                 <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
            </buildArgs>
       </configuration>
  </plugin>
  ```

  

* 在每个微服务项目根目录下建立Dockerfile文件

  ```
  FROM java:8
  COPY *.jar /app.jar
  CMD ["--server.port=8081"]
  EXPOSE 8081
  ENTRYPOINT ["java","-jar","/app.jar"]
  ```

* 添加harbor登录的凭证

  ```
  系统设置-凭证管理，添加凭证，选择用户名密码类型的凭证
  ```

* 生成登录harbor的流水线代码

  ```
  流水线语法，选择 withCredentials:Bind Credentials to variables
  新增-选择：UserName and Password separated,这种方式有助于引用用户名和密码变量
  选择harbor凭证，生成流水线代码
  ```

* 修改Jenkinsfile构建脚本

  ```
  def tag = "latest"
  //Harbor私服地址
  def harbor_url = "HarborIp:85"
  //Harbor的项目名称
  def harbor_project_name = "testjenkins"
  
  stage('编译，构建镜像') {
  // Jenkins参数化构造过程中添加的参数名称，代表项目名称
  	def imageName = "${project_name}:${tag}" //编译，安装公共工程
  	
    //编译，安装公共工程，-f tensquare_common  指定工程名称,如果有多个工程需要编译、打包，可以在Jenkins参数化构造过程中添加选项参数，设置可选择的工程名称，将工程名称作为参数传递给mvn指令
    sh "mvn -f tensquare_common clean install"  
    // dockerfile:build 触发dockerfile-maven-plugin插件根据工程目录下Dockerfile文件打包生成镜像文件
    sh "mvn -f ${project_name} clean package dockerfile:build"	
    
    // 给镜像打标签
    sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"
   	
    // 使用harbor凭证
    withCredentials([usernamePassword(credentialsId: 'harborlogin', passwordVariable: 'password', usernameVariable: 'username')]) {
    
    // 登录harbor
  	sh "docker login -u ${username} -p ${password} ${harbor_url}" 
  	
  	//上传镜像
  	sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
  		
  	# 启动容器
  	docker run -di -p $port:$port $imageName
  	}
  }
  ```

  

  ### 远程触发服务器拉取镜像部署

  * 安装 Publish Over SSH 插件，可以实现远程发送Shell命令

  * 拷贝公钥到远程服务器

    ssh-copy-id 192.168.66.103

  * 系统配置->添加远程服务器

  * 生成远程调用模板代码，ssh publisher

  ```
  sshPublisher(publishers: [sshPublisherDesc(configName: 'master_server',
  transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand:
  "/opt/jenkins_shell/deploy.sh $harbor_url $harbor_project_name $project_name
  $tag $port", execTimeout: 120000, flatten: false, makeEmptyDirs: false,
  noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '',
  remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')],
  usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
  ```

  

  * 远程脚本

  ```
  #! /bin/sh
  #接收外部参数 harbor_url=$1 harbor_project_name=$2 project_name=$3
  tag=$4
  port=$5
  imageName=$harbor_url/$harbor_project_name/$project_name:$tag
  echo "$imageName"
  #查询容器是否存在，存在则删除
  containerId=`docker ps -a | grep -w ${project_name}:${tag} | awk '{print $1}'` if [ "$containerId" != "" ] ; then
  #停掉容器
  docker stop $containerId
  #删除容器
  docker rm $containerId
  echo "成功删除容器" fi
  #查询镜像是否存在，存在则删除
  imageId=`docker images | grep -w $project_name | awk '{print $3}'`
  if [ "$imageId" != "" ] ; then #删除镜像
  docker rmi -f $imageId echo "成功删除镜像"
  fi
  # 登录Harbor私服
  docker login -u itcast -p Itcast123 $harbor_url
  # 下载镜像
  docker pull $imageName
  # 启动容器
  docker run -di -p $port:$port $imageName
  echo "容器启动成功"
  ```

  

  

  ```
  //gitlab的凭证
  def git_auth = "68f2087f-a034-4d39-a9ff-1f776dd3dfa8" //构建版本的名称
  def tag = "latest"
  //Harbor私服地址
  def harbor_url = "192.168.66.102:85"
  //Harbor的项目名称
  def harbor_project_name = "tensquare"
  //Harbor的凭证
  def harbor_auth = "ef499f29-f138-44dd-975e-ff1ca1d8c933"
  node { stage('拉取代码') {
         checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']],
  doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
  userRemoteConfigs: [[credentialsId: "${git_auth}", url:
  'git@192.168.66.100:itheima_group/tensquare_back.git']]])
  }
  stage('代码审查') {
         def scannerHome = tool 'sonarqube-scanner'
         withSonarQubeEnv('sonarqube6.7.4') {
             sh """
                cd ${project_name}
                ${scannerHome}/bin/sonar-scanner
  """
  stage('编译，构建镜像') { //定义镜像名称
  def imageName = "${project_name}:${tag}" //编译，安装公共工程
  sh "mvn -f tensquare_common clean install" //编译，构建本地镜像
         sh "mvn -f ${project_name} clean package dockerfile:build"
  //给镜像打标签
         sh "docker tag ${imageName}
  ${harbor_url}/${harbor_project_name}/${imageName}"
  } }
   
  //登录Harbor，并上传镜像
         withCredentials([usernamePassword(credentialsId: "${harbor_auth}",
  passwordVariable: 'password', usernameVariable: 'username')]) {
  //登录
  sh "docker login -u ${username} -p ${password} ${harbor_url}" //上传镜像
  sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
  }
  //删除本地镜像
  sh "docker rmi -f ${imageName}"
  sh "docker rmi -f ${harbor_url}/${harbor_project_name}/${imageName}"
  } }
  ```

  