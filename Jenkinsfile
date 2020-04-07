def ORG = "mayadataio"
def REPO = "openebs-upgrade"
def TAG = ""
pipeline {
    agent any
    stages {
        stage('Build Image') {
            steps {
                script {
                  GIT_SHA = sh(
                            returnStdout: true,
                            script: "git log -n 1 --pretty=format:'%h'"
                            ).trim()
                  echo "Building Go Code"
                  sh "docker build -t ${ORG}/${REPO}:ci-${GIT_SHA} ."
                }
            }
        }
        stage('Dependencies'){
	     when { expression { env.CHANGE_ID == null } }
            //when {
               // allOf {
                 //   expression { env.CHANGE_ID != null }
                  //  expression { env.CHANGE_TARGET != null }
                //}
            //}  //This is to run a stage only when not building a PR
            steps {   
                script {
                    sh """
                        git clone git@github.com:mayadata-io/maya-io-release.git                        
                    """
                    if (env.BRANCH_NAME == 'master')  {
                             TAG = sh (returnStdout: true,script: "./maya-io-release/utils/version_override ${REPO} ${env.BRANCH_NAME}").trim()
                             echo "$TAG"
                    } else {
                        TAG = sh (returnStdout: true,script: "./maya-io-release/utils/tag_fetch.sh ${REPO} ${env.BRANCH_NAME}").trim()
                        echo "$TAG"
                    }   
                 }
            }
        }
        stage('Push Image') {
            steps {
                script {
		             withCredentials([usernamePassword( credentialsId: 'docke_cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                         if(env.BRANCH_NAME == TAG){
                                     echo "Pushing the image with the tag..."
                                     sh "docker login -u${USERNAME} -p${PASSWORD} "
			                         sh "docker tag ${ORG}/${REPO}:ci-${GIT_SHA} ${ORG}/${REPO}:${TAG} && docker push ${ORG}/${REPO}:${TAG}"
                         } else if (env.BRANCH_NAME == 'master')  {
                               sh """
                                   git tag -fa "${TAG}" -m "Release of ${TAG}"
                                  """
                               sh "git tag -l"
                               sh """
                                  git push https://${user}:${pass}@github.com/mayadata-io/${REPO}.git --tag
                                   """
                            
                            } else {
			                   echo "WARNING: Not pushing Image"
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'This will always run'
            deleteDir()
        }
        changed {
/*            slackSend channel: '#jenkins-builds',
                   color: 'good',
                   message: "Build ${currentBuild.fullDisplayName} is now stable :dance: :thumbsup: "
            echo 'This will run only if the state of the Pipeline has changed'
*/            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
