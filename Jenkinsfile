node('master')
{
    tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'

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
            export PATH=$PATH:${JENKINS_HOME}/tools/hudson.plugins.gradle.GradleInstallation/gradle3.3/bin/
            export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
            tar -zxvf pheraska_dsl_script.tar.gz jobs.groovy
            tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile ${WORKSPACE}build/libs/$(basename "$PWD").jar
            ''';
    }
    
    /*


stage 'Asking for manual approval.'

stage 'Deployment.'

stage 'Sending status.'
*/
}

