def gv
pipeline {
    agent any
    parameters {
        choice(name: "Version", choices: ["1.1.0", "1.2.0", "1.3.0" ], description:"")
        booleanParam(name: "executeTests", defaultValue: true, description:"")
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build") {
            steps {
                script {
                    gv.buildApp() 
                }
            }
        }
        stage("test") {
            when {
                expression {
                    params.executeTests
                }
            }
            steps {                  
                script {
                    gv.testApp() 
                }
            }
        }
        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                    env.ENV = input message:  "Select the env. to deploy to", ok "Done", parameters: [choice(name: "ONE", choices: ["dev", "staging", "production" ], description:"")]
                }
                echo "Deploying to ${ENV}"
            }               
        }
    }                      
}                          