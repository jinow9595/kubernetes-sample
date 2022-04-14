/* pipeline 변수 설정 */
def DOCKER_IMAGE_NAME = "test/webflux-repo" // 생성하는 Docker image 이름
def DOCKER_IMAGE_TAGS = "testtag"           // 생성하는 Docker image 태그
def NAMESPACE = "kube-public"
def VERSION = "${env.BUILD_NUMBER}"
def DATE = new Date();

//root@cd-jenkins-dcc687b7d-n9h2t:/var/jenkins_home/workspace/testJenkins@script/9b04a074a89e2d2786e5b310ea8f6148b62faee48db94610fdb202207ff031fe# cat Jenkinsfile

podTemplate(label: 'builder',
            containers: [
                //containerTemplate(name: 'node', image: 'node:alpine3.15', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'maven', image: 'maven:3.8.5-openjdk-11', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.23.2', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v3.8.1', command: 'cat', ttyEnabled: true)
            ],
            volumes: [
                hostPathVolume(mountPath: '/home/maven/.maven', hostPath: '/home/admin/k8s/jenkins/.maven'),
                hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
            ]) {
    node('builder') {
        stage('Checkout') {
            // gitlab으로부터 소스 다운
            sh "echo Checkout"
            checkout scm
        }
        stage('Build') {
            container('maven') {
                sh "echo Build > maven"
                sh "mvn package"
            }
        }
        stage('Docker build') {
            container('docker') {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD')]) {
                        // ./build/libs 생성된 jar파일을 도커파일을 활용하여 도커 빌드를 수행한다
                        sh "echo Docker build > docker"
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAGS} ."
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAGS}"
                }
            }
        }
        stage('Run kubectl') {
            container('kubectl') {
                sh "echo Run kubectl kubectl"
                sh "kubectl get pods"
            }
        }
        stage('Run helm') {
            container('helm') {
                sh "echo Run helm helm"
                sh "helm list"
            }
        }
    }
}