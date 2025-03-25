def registry= "975050242866.dkr.ecr.ca-central-1.amazonaws.com"
def tag = ""
def ms = "result"
def region = "ca-central-1"

pipeline{
    agent any
    stages{
        stage("init"){
            steps{
                script{
                    tag = getTag()
                  //  ms = getMsName()
                }
            }
        }
        stage("Build Docker image"){
            steps{
                script{
                    sh "docker build . -t ${registry}/${ms}:${tag}"
                }
            }
        }

        stage("Login to Ecr"){
            steps{
                script{
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}"
                    }
                }
            }

        stage("Docker push"){
            steps{
                script{
                        sh "docker push ${registry}/${ms}:${tag}"
                    }
                }
            }

        stage("Deploy to Dev"){
            when{branch 'develop'}
            steps{
                script{
                    withAWS(region: region, credentials:'aws_creds'){
                        sh "aws eks update-kubeconfig --name result-dev"
                        sh 'sudo curl -o /usr/local/bin/kubectl -LO "https://dl.k8s.io/release/v1.28.5/bin/linux/amd64/kubectl"' 
                        sh 'sudo chmod +x /usr/local/bin/kubectl'
                        sh "./kubectl set image deploy/result result=${registry}/${ms}:${tag} -n result"
                        sh "./kubectl rollout restart deploy/result -n result"
                    }
                }
            }
        }
    }
}

def getMsName(){
    print env.JOB_NAME
    return env.JOB_NAME.split("/")[0]
}

def getTag(){
    sh "ls -l"
    def version = readJSON file: 'package.json'
    version = version["version"]
    print "version: ${version}"

    def tag = ""
    if (env.BRANCH_NAME == "main"){
        tag = version
    } else if(env.BRANCH_NAME == "develop"){
        tag = "${version}-develop"
    } else {
        tag = "${version}-${env.BRANCH_NAME}"
    }
    return tag
}
