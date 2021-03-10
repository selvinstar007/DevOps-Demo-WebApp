pipeline 
{
    agent any
    
    tools
    {
        maven 'Maven3.6.3'
        jdk 'JDK8'
        
    }
    stages 
    {
        stage('run ansible to create aws vm')
        {
            steps
            {
                ansiblePlaybook installation: 'Ansible', playbook: '/var/lib/jenkins/ec2createfile.yaml'
            }
        }
        
        stage('Add wait time for aws vm refresh')
        {
            steps
            {
                sleep time: 3, unit: 'MINUTES'
            }
        }
        
        stage('run ansible to install tomcat')
        {
            steps
            {
                ansiblePlaybook installation: 'Ansible', playbook: '/var/lib/jenkins/install-tomcat-postgres.yaml'
            }
        }
        
        stage('Add wait time post tomcat install')
        {
            steps
            {
                sleep time: 5, unit: 'MINUTES'
            }
        }
        
        stage("source code checkout")
        {
            steps
            {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/master']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/selvinstar007/DevOps-Demo-WebApp.git']]])
            }
        }
        
        stage('static code analysis')
        {
            steps 
            {
                withSonarQubeEnv('SonarQube')
                {
                    sh 'mvn validate sonar:sonar -Dsonar.login=admin -Dsonar.password=sonar -Dsonar.exclusions=src/test/java/servlet/*.java'
                }
            }
        }
        
        stage('confluence - Static code analysis')
        {
            steps
            {
                jiraComment body: 'Static code analysis - Complete', issueKey: 'SQUAD-6'
            }
        }

        stage("Maven test build")
        {
            steps
            {
                sh "mvn compile"
            }
        }        
        
        stage("Deploy to Test")
        {
            steps
            {
                deploy adapters: [tomcat8(credentialsId: 'tomcatpassword123', 
                path: '', 
                url: 'http://172.31.10.1:8080/')], 
                contextPath: '/QAWebapp', 
                war: '**/*.war'
            }
        }
        
        stage('confluence - Deploy to test')
        {
            steps
            {
                jiraComment body: 'Deploy to test vm complete', issueKey: 'SQUAD-6'
            }
        }
        
        stage("Upload to Artifactory")
        {
            steps
            {
                rtMavenResolver(
                id: 'resolver-unique-id',
                serverId: 'Artifactory',
                releaseRepo: 'libs-release',
                snapshotRepo: 'libs-snapshot'
                )  
 
                rtMavenDeployer(
                id: 'deployer-unique-id',
                serverId: 'Artifactory',
                releaseRepo: 'libs-release-local',
                snapshotRepo: 'libs-snapshot-local'
                )
            }
        }
        
        stage("UI Test")
        {
            steps
            {
                sh 'mvn -f functionaltest/pom.xml test'
            }
        }
        
        stage('confluence - UI test')
        {
            steps
            {
                jiraComment body: 'UI test - Complete', issueKey: 'SQUAD-6'
            }
        }
        
        stage('Test publish HTML report')
        {
            steps
            {
                publishHTML([allowMissing: false, 
                alwaysLinkToLastBuild: false, 
                keepAll: false, 
                reportDir: '\\functionaltest\\target\\surefire-reports', 
                reportFiles: 'index.html', 
                reportName: 'HTML Report', 
                reportTitles: 'HTML Report'])
            }
        }
        
        stage("Blazemeter")
        {
            steps
            {
                blazeMeterTest credentialsId: 'blazemeter', 
                getJtl: true, 
                getJunit: true, 
                testId: '9018087.taurus', 
                workspaceId: '756604'       
            }
        }
        
        stage('confluence - Blazemeter test')
        {
            steps
            {
                jiraComment body: 'Blazemeter test - Complete', issueKey: 'SQUAD-6'
            }
        }
        
        stage("Deploy to Prod")
        {
            steps
            {
                deploy adapters: [tomcat8(credentialsId: 'tomcatpassword123', 
                path: '', 
                url: 'http://172.31.11.1:8080/')], 
                contextPath: 'ProdWebapp', 
                war: '**/*.war'
            }
        }
        
        stage('confluence - Deploy to prod')
        {
            steps
            {
                jiraComment body: 'Deploy to prod - Complete', issueKey: 'SQUAD-6'
            }
        }
        
        stage("Sanity Test")
        {
            steps
            {
                sh 'mvn -f Acceptancetest/pom.xml test'
            }
        }
        
        stage('confluence - Sanity test')
        {
            steps
            {
                jiraComment body: 'Sanity test - Complete', issueKey: 'SQUAD-6'
            }
        }
        
        stage('Prod publish HTML report')
        {
            steps
            {
                publishHTML([allowMissing: false, 
                alwaysLinkToLastBuild: false, 
                keepAll: false, 
                reportDir: '\\Acceptancetest\\target\\surefire-reports', 
                reportFiles: 'index.html', 
                reportName: 'Sanity Test Report', 
                reportTitles: 'Sanity Test Report'])
            }
        }
        
        stage('confluence - Prod deploy and test - Complete')
        {
            steps
            {
                jiraComment body: 'Prod deploy and test - Complete', issueKey: 'SQUAD-6'
            }
        }
    }
    
    post
    {
        always 
        {
            slackSend channel: '#alerts',
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
