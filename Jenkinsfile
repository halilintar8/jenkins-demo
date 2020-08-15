pipeline{
      // 定义groovy脚本中使用的环境变量
      environment{
        // 将构建任务中的构建参数转换为环境变量
        IMAGE_TAG3 =  sh(returnStdout: true,script: 'echo $image_tag3').trim()
	      /// IMAGE_TAG = "latest"
        ORIGIN_REPO =  sh(returnStdout: true,script: 'echo $origin_repo').trim()
        REPO =  sh(returnStdout: true,script: 'echo $repo').trim()
        BRANCH =  sh(returnStdout: true,script: 'echo $branch').trim()            

      }

      // 定义本次构建使用哪个标签的构建环境，本示例中为 “slave-pipeline”
      agent{
        node{
          label 'slave-pipeline'
        }
      }

      // "stages"定义项目构建的多个模块，可以添加多个 “stage”， 可以多个 “stage” 串行或者并行执行
      stages{
        // 定义第一个stage， 完成克隆源码的任务
        stage('Git'){
          steps{
            git branch: '${BRANCH}', credentialsId: 'bintang_lenovo', url: 'git@bitbucket.org:m_bintang/jenkins-demo.git'
          }
        }

        // 添加第二个stage， 运行源码打包命令
        stage('Package'){
          steps{
              container("maven") {
                  sh "mvn package -B -DskipTests"
              }
          }
        }


        // 添加第三个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
        //stage('Image Build And Publish'){
        //  steps{
        //      container("kaniko") {
        //          sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${ORIGIN_REPO}/${REPO}:${IMAGE_TAG} --skip-tls-verify"
        //      }
        //  }
        //}

        stage('Docker Build Image') {
          steps{
                container('docker') {
                    echo "Building docker image"                    
                    sh "docker build -t halilintar8/demo-pipeline:latest ."                    
                }
          }          
        }

        stage("Push image to Docker Hub") {
            steps {
                container('docker') {
                  echo "Push docker image to hub.docker.com"
                    script {
                      //docker.withRegistry('', 'dockerhub') {
                      docker.withRegistry('', 'hub_docker_halilintar8') {
                      
                        sh "docker tag ${ORIGIN_REPO}/${REPO} ${ORIGIN_REPO}/${REPO}:${IMAGE_TAG3}"
                        sh "docker push ${ORIGIN_REPO}/${REPO}:${IMAGE_TAG3}"
                        
                      }
                    }
                }
            }
        }

        /// 添加第四个stage, 部署应用到指定k8s集群
        stage('Deploy to Kubernetes') {
          steps {
            container('kubectl') {
              echo "Deploy to Kubernetes Cluster"
                script {

                  step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: 'https://10.8.1.120:6443', credentialsId:'k8sCertAuth', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG3'])

                }  
              
            }
          }
        }
      }
}
