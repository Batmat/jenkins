pipeline {

    options {
        timeout(time: 12, unit: 'HOURS') // global timeout to kill rogue builds
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '20'))
        timestamps()
    }

    environment {
        RUN_TESTS = false // TEMPORARY!
        FAIL_FAST = true
        mvnCmd = "mvn -Pdebug -U clean install"// ${false ? '-Dmaven.test.failure.ignore=true' : '-DskipTests'} -V -B -Dmaven.repo.local=${pwd()}/.repository"
    }

    // Not strictly required, but better check/fail fast if tools are NOT available
    tools {
        maven 'mvn'
        jdk 'jdk8'
    }

    agent none

    stages {

        stage("Build") {
            /*
            options {
                timeout(time: 3, unit: 'HOURS')
            }
            */
            steps {

                parallel(
                        "Linux": {
                            node('linux') {

                                withMaven(jdk: 'jdk8', maven: 'mvn', mavenLocalRepo:'.repository') {
                                    sh mvnCmd
                                }

                                script {
                                    def files = findFiles(glob: '**/target/*.jar, **/target/*.war, **/target/*.hpi')
                                    renameFiles(files, "linux")
                                }
                            }
                        },
                        "Windows": {
                            node('windows') {

                                withMaven(jdk: 'jdk8', maven: 'mvn', mavenLocalRepo:'.repository') {
                                    bat "$mvnCmd -Duser.name=yay" // INFRA-1032 workaround
                                }

                                script {
                                    def files = findFiles(glob: '**/target/*.jar, **/target/*.war, **/target/*.hpi')
                                    renameFiles(files, "windows")
                                }
                            }
                        }, failFast: true // FAIL_FAST
                )
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/target/*.jar, **/target/*.war, **/target/*.hpi',
                            fingerprint: true
                    script {
                        if (RUN_TESTS == false) {
                            junit healthScaleFactor: 20.0, testResults: '**/target/surefire-reports/*.xml'
                        }
                    }
                }
            }
        }
    }

}

// This hacky method is used because File is not whitelisted,
// so we can't use renameTo or friends
void renameFiles(def files, String prefix) {
    for(i = 0; i < files.length; i++) {
        def newPath = files[i].path.replace(files[i].name, "${prefix}-${files[i].name}")
        def rename = "${files[i].path} ${newPath}"
        if(isUnix()) {
            sh "mv ${rename}"
        } else {
            bat "move ${rename}"
        }
    }
}
