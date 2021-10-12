pipeline {
      /// Jenkins agent
      agent { label 'flutter' }


      /// Jenkins stages
      stages {
            /// This stage Checkout contain initial job, for assing message, number, version name, and android-sdk location
            stage('Checkout') {
                  steps {
                        script {
                            sh 'echo checkout'
                        }
                  }
            }
            /// This Stage will run flutter test
            stage('Run flutter test') {
                  when {
                        /// Keep in mind that branch with a name "bump_version" wouldn't be tested because it's version bump anyway
                        expression {
                              !env.BRANCH_NAME.startsWith('bump_version')
                        }
                  }
                  steps {
                        script {
                              /// Clean before test
                              sh "echo test"
                             
                        }
                  }
            }

            /// This stage will run staging script
            stage('Build-Staging') {
                  when {
                        /// This stage only triggered by Pull Request creation
                        expression {
                              /// Information : branch PR- is representative from Pull Request action
                              env.BRANCH_NAME.startsWith("PR-")
                        }
                  }
                  
                  steps {
                        /// This add file from jenkins custom file
                        /// edit on the https://jenkinsx.ariretiarnoa2z.com/job/flutter_ariretiarno/configfiles/
                        
                        /// This script will run flutter build apk with flavor
                        /// Then move the generated apk, in order to rename to a new file
                        script {
                               sh 'echo staging'
                               
                        }
                  }
            }

            stage('Build-Preview') {
                  when {
                        /// This stage only triggered by merge to a main (after PR)
                        expression {
                              env.BRANCH_NAME == 'main'
                        }
                  }
                  steps {
                        /// This add file from jenkins custom file
                    
                        /// This script will run flutter build apk with flavor
                        /// Then move the generated apk, in order to rename to a new file
                        script {
                               sh 'echo preview'
                               
                        }
                        /// This script will clean & run flutter build apk with flavor
                        /// Then move the generated apk, in order to rename to a new file
                        script {
                              sh 'echo staging'
                        }
                  }
            }

            stage('Build-Production') {
                  when { 
                        expression {
                              env.BRANCH_NAME ==~ /\d+\.(?:\d+|x)(?:\.\d+|x)?/
                        }
                  }
                  steps {
                        /// This add file from jenkins custom file
                        /// edit on the https://jenkinsx.ariretiarnoa2z.com/job/flutter_ariretiarno/configfiles/
                        configFileProvider(
                              [configFile(fileId: 'secrets-production', variable: 'SECRETS_PROD')]) {
                                    sh 'cat $SECRETS_PROD > .env_production'
                        }
                        /// This script will run flutter build apk with flavor
                        /// Then move the generated apk, in order to rename to a new file
                        script {
                               sh 'echo build prod'
                        }
                        /// This script will clean & run flutter build apk with flavor
                        /// Then move the generated apk, in order to rename to a new file
                        
                        
                        /// Do something
                        /// We can do a fastlane task here
                  }
            }
      }
}