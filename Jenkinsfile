node('master')
{
    tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'

    stage ('cleanup')
    {
        step([$class: 'WsCleanup'])
    }
    
    stage ('Preparation (Checking out).')
    {
        //git url:'https://github.com/MNT-Lab/mntlab-pipeline.git', branch:'pheraska'
        git url:'https://github.com/kickman2l/test-jenkins.git', branch:'master'
    }

    stage ('Building code.')
    {
        sh '''
        export PATH=$PATH:${JENKINS_HOME}/tools/hudson.plugins.gradle.GradleInstallation/gradle3.3/bin/
        export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
        gradle build
        ''';
    }

    stage ('Testing.')
    {
        parallel JUnit:
        {
            sh '''
            export PATH=$PATH:${JENKINS_HOME}/tools/hudson.plugins.gradle.GradleInstallation/gradle3.3/bin/
            export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
            gradle test
            ''';
        },
        Jacoco:
        {
            sh '''
            export PATH=$PATH:${JENKINS_HOME}/tools/hudson.plugins.gradle.GradleInstallation/gradle3.3/bin/
            export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
            gradle cucumber
            ''';
        },
        Cucumber:
        {
        sh '''
            export PATH=$PATH:${JENKINS_HOME}/tools/hudson.plugins.gradle.GradleInstallation/gradle3.3/bin/
            export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
            gradle jacoco
            ''';
        }
        failFast: true|false
    }
    
    stage ('Triggering job and fetching artefact after finishing.')
    {
        build job: 'kickman', parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${env.BRANCH_NAME}"]]
        step ([$class: 'CopyArtifact', projectName: 'kickman']);
    }
    
    stage ('Packaging and Publishing results.')
    {
        sh '''
            cp ${WORKSPACE}/build/libs/$(basename "$PWD").jar ${WORKSPACE}
            tar -zxvf pheraska_dsl_script.tar.gz jobs.groovy
            tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile $(basename "$PWD").jar
            ''';
            archiveArtifacts artifacts: 'pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz'
    }

    stage ('Asking for manual approval.')
    {
        timeout(time:3, unit:'MINUTES') 
        {
            input message:'Approve deployment?'
        }
    }
    
    stage ('Deployment.')
    {
            sh '''
               export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
               java -jar $(basename "$PWD").jar
               '''
    }
    
    stage ('Sending status.')
    {
        echo "${currentBuild.result}"
       if(currentBuild.result == 'SUCCESS')
       {
           echo "Job ${JOB_NAME} successfully done!"
       }
       else
       {
           echo "Something goes wrong job ${JOB_NAME} finished with ${currentBuild.result}"
       }
    }
}

