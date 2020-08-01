---
title: pipeline语法实例
date: 2020-04-22 17:38:51
tags:
- jenkins
- pipeline
categories: jenkins
---

一些pipeline语法实例

<!--more-->

```groovy
node {
    stage('目录') {
        sh 'pwd'
      }
    stage('克隆代码') {
       // git url: "ssh://git@172.81.248.104:19999/root/xqtv-management.git"
        script {
            build_tag = sh(returnStdout: true,script: 'git log -1 --pretty=format:%h').trim()
        }
      }
    stage('目录') {
        sh 'pwd'
        sh 'date'
      }
    stage('代码打包') {
        script {
            app = input message: '选择端口号', ok: '确认', parameters: [choice(choices: ['9871', '9872'], description: 'sfs端口号', name: 'sfs_port')]  //参数化构建
        }
        sh '# /var/jenkins_home/MVN/mvn/bin/mvn clean install'
      }
     stage('部署jar包') {
        sh "echo $app"
        sh "echo ${build_tag}"
      }
}

```

 

```sh
node('haimaxy-jnlp') {
    stage('pull') {
        echo "1.Prepare Stage"
        checkout scm
        echo env.BRANCH_NAME
        echo "${workspace}"
        sh "env"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        dir('util'){
        //git branch: 'master', url: 'ssh://git@172.81.248.104:19999/maimai/youli-util.git'
        }
        dir('Service'){
        //git branch: 'master', url: 'ssh://git@172.81.248.104:19999/maimai/youli-Service.git'
        }
        dir('api'){
        //git branch: 'master', url: 'ssh://git@172.81.248.104:19999/maimai/youli-Client.git'
        }
        sh "ls"
        /*sh """
            cd ${workspace}/util
            /home/jenkins/maven/bin/mvn clean install -Dmaven.test.skip=true
            cd ${workspace}/Service
            /home/jenkins/maven/bin/mvn clean install -Dmaven.test.skip=true
            cd ${workspace}/api
            sed -i "/dubbo.registry.address/s/127.0.0.1:2181/zookeeper1?backup=zookeeper2,zookeeper3/" src/main/resources/application.properties
            /home/jenkins/maven/bin/mvn clean install -Dmaven.test.skip=true
            ls -l && pwd
        """*/
        /*script {
            sh 'ls'
            build_tag = sh(returnStdout: true, script: 'cd api && git rev-parse --short HEAD').trim()
            //echo env.GIT_COMMIT 
            //echo env.GIT_BRANCH 
            //echo env.GIT_REVISION 
            echo  "${build_tag}"
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}" 
            }
        }*/
        //sh "docker build -t 192.168.220.101/library/api:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        /*withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword} https://192.168.220.101"
            //sh "docker push 192.168.220.101/library/api:${build_tag}"
            sh 'ls'
        }*/
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        /*if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }*/
        sh "pwd && ls -l"
        //sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        //sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        //sh "mv k8s.yaml api-${build_tag}.yaml"
        //sh "kubectl apply -f api-${build_tag}.yaml --record"
    }
}
```

