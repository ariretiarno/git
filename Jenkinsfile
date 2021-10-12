pipeline {
      /// Jenkins agent
      agent { label 'flutter' }


      /// Jenkins stages
      stages {
            /// This stage Checkout contain initial job, for assing message, number, version name, and android-sdk location
            stage('Checkout') {
                  steps {
                        script {
                              env.COMMIT_MESSAGE = commitMessage()
                              env.COMMIT_NUMBER = commitSha1()
                              env.VERSION_NAME = "v${getVersionName()}"
                              env.ANDROID_HOME = '/opt/android-sdk'
                        }
                        script {
                              recordCommitMsg(env.VERSION_NAME, env.COMMIT_MESSAGE)
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
      /// Then after all of that, we can now do an action http post to discord
      /// For informing every member of mobile-devops
      post {
            /// This sript will running on succes
            success {
                  script {
                        def path = "ariretiarno-flutter"
                        def applicationName = "ariretiarno-flutter"
                        def slackChannel = "#mobile-build"
                        def message = "> ${path}: `${env.VERSION_NAME}`\n> `${commitAuthor()}`: ${identifier()}\n\n"
                        if (env.BRANCH_NAME ==~ /\d+\.(?:\d+|x)(?:\.\d+|x)?/) {
                              /// If the tag was created then the [preview-apk & appbundle] will sended to discord
                              /// The preview-apk name will be named, for the example app-preview_v.0.1.0.apk
                              /// The bundle name will be named, for the example aap-production_v.0.1.0+1.aab
                              message += "`[$BUILD_NUMBER] BUILD ${currentBuild.currentResult}`: $JOB_NAME\n"
                              message += "[preview-apk] ${artifactPath}flutter-apk/app-preview_"+env.VERSION_NAME+".apk\n"
                              message += "[bundle] ${artifactPath}bundle/productionRelease/app-production_"+env.VERSION_NAME+".aab\n"
                              notify("slack","${currentBuild.currentResult}","${message}","${applicationName}","${slackChannel}")
                        } else if (env.BRANCH_NAME == "main") {
                              /// If the tag was created then the [staging-apk & appbundle] will sended to discord
                              /// The staging-apk name will be named, for the example app-staging.0.1.0.apk
                              /// The preview-apk name will be named, for the example app-preview_v.0.1.0.apk
                              message += "`[$BUILD_NUMBER] BUILD ${currentBuild.currentResult}`: $JOB_NAME\n"
                              message += "[staging-apk] ${artifactPath}flutter-apk/app-staging_"+env.VERSION_NAME+".apk\n"
                              message += "[preview-apk] ${artifactPath}flutter-apk/app-preview_"+env.VERSION_NAME+".apk\n"
                              notify("slack","${currentBuild.currentResult}","${message}","${applicationName}","${slackChannel}")
                        } else if (env.BRANCH_NAME.startsWith("PR-")) {
                              /// If the tag was created then the [staging-apk] will sended to discord
                              /// The staging-apk name will be named, for the example app-staging.0.1.0.apk
                              message += "`[$BUILD_NUMBER] BUILD ${currentBuild.currentResult}`: $JOB_NAME\n"
                              message += "[staging-apk] ${artifactPath}flutter-apk/app-staging_"+env.CHANGE_ID+".apk\n"
                              notify("slack","${currentBuild.currentResult}","${message}","${applicationName}","${slackChannel}")
                        }
                  }
            }
            /// But if there's failur it will be here in a no time
            failure {
                  script {
                        def path = "ariretiarno-flutter"
                        def applicationName = "ariretiarno-flutter"
                        def slackChannel = "#mobile-build"
                        def message = "> ${path}: `${env.VERSION_NAME}`\n> `${commitAuthor()}` : ${identifier()}\n\n"
                        message += "`[$BUILD_NUMBER] BUILD ${currentBuild.currentResult}`: $JOB_NAME\n"
                        message += "Ups, an error occured"
                        notify("slack","${currentBuild.currentResult}","${message}","${applicationName}","${slackChannel}")
                  }
            }
      }
}