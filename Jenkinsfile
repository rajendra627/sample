podTemplate(label: 'mypod', serviceAccount: 'jenkins', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.0', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'maven-firefox', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat', envVars: [
        containerEnvVar(key: 'HTTP_PROXY', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'HTTPS_PROXY', value: 'https://64.102.255.40:80'),
        containerEnvVar(key: 'http_proxy', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'https_proxy', value: 'https://64.102.255.40:80'),
    ]),
    containerTemplate(name: 'maven-chrome', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat', envVars: [
        containerEnvVar(key: 'HTTP_PROXY', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'HTTPS_PROXY', value: 'https://64.102.255.40:80'),
        containerEnvVar(key: 'http_proxy', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'https_proxy', value: 'https://64.102.255.40:80'),
    ]),
    containerTemplate(name: 'selenium-hub', image: 'selenium/hub:3.4.0'),
    containerTemplate(name: 'selenium-chrome', image: 'selenium/node-chrome:3.4.0', envVars: [
        containerEnvVar(key: 'HUB_PORT_4444_TCP_ADDR', value: 'localhost'),
        containerEnvVar(key: 'HUB_PORT_4444_TCP_PORT', value: '4444'),
        containerEnvVar(key: 'DISPLAY', value: ':99.0'),
        containerEnvVar(key: 'SE_OPTS', value: '-port 5556'),
        containerEnvVar(key: 'HTTP_PROXY', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'HTTPS_PROXY', value: 'https://64.102.255.40:80'),
        containerEnvVar(key: 'http_proxy', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'https_proxy', value: 'https://64.102.255.40:80'),
    ]),
    containerTemplate(name: 'selenium-firefox', image: 'selenium/node-firefox:3.4.0', envVars: [
        containerEnvVar(key: 'HUB_PORT_4444_TCP_ADDR', value: 'localhost'),
        containerEnvVar(key: 'HUB_PORT_4444_TCP_PORT', value: '4444'),
        containerEnvVar(key: 'DISPLAY', value: ':98.0'),
        containerEnvVar(key: 'SE_OPTS', value: '-port 5557'),
        containerEnvVar(key: 'HTTP_PROXY', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'HTTPS_PROXY', value: 'https://64.102.255.40:80'),
        containerEnvVar(key: 'http_proxy', value: 'http://64.102.255.40:80'),
        containerEnvVar(key: 'https_proxy', value: 'https://64.102.255.40:80'),
    ])
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]) {

    node('mypod') {
        ////////// Step 1 //////////
        stage('Git Clone') {
            parallel (
                DUTClone: {
                    parallel (
                        DockerContainer: {
                            container('docker'){
                                stage ('Docker') {
                                    sh "docker info"
                                    sh "docker pull ubuntu"
                                }
                            }
                        },
                        HelmContainer: {
                            container('helm'){
                                stage('Helm'){
                                    sh "helm ls"
                                    sh "helm status jenkins"
                                }
                            }
                        },
                        KubectlContainer: {
                            container('kubectl'){
                                stage('Kubectl'){
                                    sh "kubectl version"
                                    sh "kubectl get all --all-namespaces"
                                }
                            }
                        }
                    )
                },
                TOOLS: {
                    stage('Checkout Tools Source Code'){
                        git 'https://github.com/carlossg/selenium-example.git'
                    }
                }

            )
        }
        stage('Build & Unit Test') {
            parallel(
                DUTBUILD: {
                    parallel (
                        DockerContainer: {
                            container('docker'){
                                stage ('Docker') {
                                    sh "docker info"
                                    sh "docker pull ubuntu"
                                }
                            }
                        },
                        HelmContainer: {
                            container('helm'){
                                stage('Helm'){
                                    sh "helm ls"
                                    sh "helm status jenkins"
                                }
                            }
                        },
                        KubectlContainer: {
                            container('kubectl'){
                                stage('Kubectl'){
                                    sh "kubectl version"
                                    sh "kubectl get all --all-namespaces"
                                }
                            }
                        }
                    )
                },
                TOOLSTEST: {
                    parallel (
                        firefox: {
                            container('maven-firefox') {
                                stage('Test firefox') {
                                    sh 'env'
                                    sh 'ip a'
                                    sh 'echo "64.102.255.40   proxy.esl.cisco.com" >> /etc/hosts'
                                    sh 'apk update && apk add ca-certificates && update-ca-certificates && apk add openssl && apk add wget'
                                }
                            }
                        },
                        chrome: {
                            container('maven-chrome') {
                                stage('Test chrome') {
                                    sh 'env'
                                    sh 'ip a'
                                    sh 'echo "64.102.255.40   proxy.esl.cisco.com" >> /etc/hosts'
                                    sh 'apk update && apk add ca-certificates && update-ca-certificates && apk add openssl && apk add wget'
                                    sh 'mvn -B clean test -Dselenium.browser=chrome -Dsurefire.rerunFailingTestsCount=5 -Dsleep=0 -DproxySet=true -DproxyHost=proxy.esl.cisco.com -DproxyPort=80'
                                    sh 'mkdir reports'
                                    sh 'cd workspace'
                                    sh 'find . -name "*.html" -exec cp {} reports \\;'
                                    archiveArtifacts artifacts: "./reports/*"
                                } 
                            }
                        }
                    )
                }
            )
        }
        stage('Functional Tests') {
            echo "Done"
        }
        stage('Publish Docker Image and Helm Charts') {
            echo "Done"
        }
        stage('Deploy to Dev Test Environment') {
            echo "Done"
        }
        stage('Dev Tests') {
            echo "Done"
        }
        stage('Cleanup Dev Test Environment Deployment') {
            echo "Done"
        }
        stage('Go for Staging Environment Deployment?') {
            echo "Done"
        }
        stage('Deploy to Staging Environment') {
            echo "Done"
        }
        stage('Staging Tests') {
            echo "Done"
        }
        stage('Cleanup Staging Environment Deployment') {
            echo "Done"
        }
        stage('Go for Production Environment Deployment?') {
            echo "Done"
        }
        stage('Deploy to Production Environment') {
            echo "Done"
        }
        stage('Production Tests') {
            echo "Done"
        }
    }
}
