## 漏洞扫描



sonarQube	



## 第三方漏洞扫描



dependency check

```
# 多模块应用
mvn dependency-check:aggregate


# 单体应用
mvn dependency-check:check

```





## 静态环境扫描





## 动态环境扫描





## 容器扫描







## 版权扫描



mvn开源协议扫描：

```shell
# 多模块应用
mvn org.codehaus.mojo:license-maven-plugin:2.2.0:aggregate-add-third-party
mvn org.codehaus.mojo:license-maven-plugin:2.2.0:aggregate-add-third-party -Dlicense.thirdPartyFilename=license.txt -Dlicense.outputDirectory=$WORKSPACE/target

# 单体应用
mvn org.codehaus.mojo:license-maven-plugin:2.2.0:add-third-party
```





## Jenkins上





sonarQube

```
                script {
                    // requires SonarQube Scanner 2.8+
                    //这里填写，在系统管理中配置的sonar服务器名称
                    scannerHome = tool 'sonar-engine'
                }
                dir(env.frontendSourceCodeHome) {
                    sh 'echo run build frontend'
                    sh 'npm config set registry https://registry.npm.taobao.org'
                    sh "npm install"
                    sh "npm run build"
                }

                dir(env.backendSourceCodeHome) {
                    sh 'rm -rf ' + env.backendSourceCodeHome + '/src/main/resources/static/*'
                    sh 'cp -rf ' + env.frontendSourceCodeHome + '/dist/* ' + env.backendSourceCodeHome + '/src/main/resources/static/'

                    sh 'mvn -U -e -DskipTests clean package'
                }
                dir(env.backendSourceCodeHome) {
                    withSonarQubeEnv('sonarqube-server') {
                            sh "${scannerHome}/bin/sonar-scanner " +
                            //在sonar中项目的名称
                            "-Dsonar.projectKey=test-auto-create " +
                            // -Dsonar.host.url=http://10.10.10.10:9000                            
                            // "sqp_6366688e82fe7e22b80cbcedbccef15b30d5b933" +
                            "-Dsonar.sourceEncoding=UTF-8 " +
                            //项目类型
                            "-Dsonar.language=java " +
                            //需要过滤的文件类型
                            "-Dsonar.exclusions=**/*.css,**/*.js,**/*.xml " +
                            //需要检查代码的路径
                            "-Dsonar.sources=src/main/java " +
                            //需要检查代码源码路径
                            "-Dsonar.java.binaries=target/classes"
                    }

                    // withSonarQubeEnv('sonarqube-server') {
                    //     withMaven(maven:'Maven-3.6.3') {
                    //         sh 'mvn clean package sonar:sonar'
                    //     }
                    // }       
                    
                    // 这个睡眠时为了防止没有分析完成就去请求结果
                    timeout(time: 5, unit: 'MINUTES') {
                        script {
                            sleep(10)
                            def qg = waitForQualityGate()
                            sh 'echo status ' + qg.status
                            if (qg.status != 'OK') {
                                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
```

