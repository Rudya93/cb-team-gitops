pipeline {
    agent {
        kubernetes {
        label 'team-automation'
        yaml """
kind: Pod
spec:
  containers:
  - name: cli
    image: caladreas/cbcore-cli:latest
    imagePullPolicy: Always
    command:
    - cat
    tty: true
"""
        }
    }
    environment {
        CREDS   = credentials('jenkins-api')
        CLI     = "java -jar /usr/bin/jenkins-cli.jar -noKeyAuth -s http://cjoc/cjoc -auth"
    }
    stages {
        stage('Update Team Recipes') {
            when { changeset "recipes/recipes.json" }
            steps {
                container('cli') {
                    sh '${CLI} ${CREDS} team-creation-recipes --put < recipes/recipes.json'
                }
            }
        }
        stage('Update Teams') {
            when { changeset "teams/*.json" }
            steps {
                container('cli') {
                    
                    // cbc teams hex --put < team-hex.json
                    sh '${CLI} ${CREDS} teams hex --put < team-hex.json'
                }
            }
        }
    }
}
