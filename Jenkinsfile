pipeline {
  agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        ansiColor('xterm')
      }
    tools {
        maven 'M3'
        }
    environment {
         BUILD_ID = "${BUILD_NUMBER}"
         VERSION = readMavenPom().getVersion()

    }
    parameters {
       choice(
               choices: 'Dev\nQA',
               description: '',
               name: 'REQUESTED_ACTION')
               booleanParam(defaultValue: false, description: 'Execute deploy', name: 'shouldDeploy')
               booleanParam(defaultValue: true, description: 'Execute Build', name: 'shouldBuild')
   }

  stages {
    stage ('Initialize') {
          steps {
                parallel(
                 environment:{
                             echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                              script{

                                      def pom = readMavenPom file: 'pom.xml'
                                      echo sh(script: 'env', returnStdout: true)
                                      sh 'chmod 777 build/*.sh'
                                 }
                           },
                   verifyMaven:{
                             sh '''
                                 echo "PATH = ${PATH}"
                                 echo "M2_HOME = ${M2_HOME}"

                             '''
                           }
                         )


                   sh '''sudo yum install rpm-build
                         %_topdir    $HOME/rpmbuild
                         mkdir -p ~/rpmbuild/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS}
                         mkdir helloworld-1.0

                      '''
                  }
          }
      }

  stage ('ShouldBuild'){

                steps{
                        script{
                                 result = sh (script: "git log --oneline -1 | grep 'stage'", returnStatus: true)

                                  echo "${env.shouldDeploy}"

                                      if (result == 0){
                                            echo ("This is not a new build")
                                            env.shouldBuild = "false"
                                            env.shouldDeploy = "true"
                                            echo "shouldBuild  IS NOW   ${env.shouldBuild}"
                                            echo "shouldDeploy IS NOW   ${env.shouldDeploy}"
                                      }



                      }
                  }
                }

    stage ('TEST') {

                      steps {
                        sh '''
                                sudo make
                                ./helloworld
                                sudo make install
                                ls -ltr /usr/local/bin
                                sudo rm -rf /usr/local/bin/helloworld
                                ls -ltr /usr/local/bin
                           '''

                          }
                      }
    stage ('Package the source directory') {

                      steps {
                        sh '''
                            tar -czvf helloworld-1.0.tar.gz helloworld-1.0/

                           '''

                          }
                      }
    stage (' Prepare the source directory') {

                      steps {
                        sh '''
                            cp helloworld-1.0.tar.gz ~/rpmbuild/SOURCES

                           '''

                          }
                      }
    stage ('Create Spec file') {

                      steps {
                        sh '''
                          touch SPEC
                          printf 'Name:           helloworld\nVersion:        1.0\nRelease:        1%{?dist}\nSummary:        A hello world program\n
                          License:        GPLv3+\nURL:            https://blog.packagecloud.io\nSource0:        helloworld-1.0.tar.gz\n
                          %description\n
                          A helloworld program\n
                          %prep\n
                          %setup\n
                          \n
                          %build\n
                          make PREFIX=/usr %{?_smp_mflags}\n\n
                          %install\n
                          make PREFIX=/usr DESTDIR=%{?buildroot} install\n\n
                          %clean\n
                          rm -rf %{buildroot}\n
                          %files\n
                          %{_bindir}/helloworld\n

                          ' > SPEC

                           '''

                          }
                      }
    stage ('Build RPM') {

                            steps {
                                script {
                                          sh 'rpmbuild -ba helloworld.spec'
                                        }
                                  }
                          }




}
