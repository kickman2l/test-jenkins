node('master')
{
    Thread.setDefaultExceptionHandler(new UncaughtExceptionHandler()
    {
        public void unchaughtException(Thread t, Throwable e) {
            System.out.println("Caught " + e);
             tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'
    def errorArray = []

    stage ('cleanup')
    {
        try
        {
            step([$class: 'WsCleanup'])
        }
        catch (error)
        {
            errorArray.push("ERROR: Cant cleanup workspace!")
        }        
    }
    
    stage ('Preparation (Checking out).')
    {
        try
        {
            //git url:'https://github.com/MNT-Lab/mntlab-pipeline.git', branch:'pheraska'
            git url:'https://github.com/kickman2l/test-jenkins.git', branch:'master'
        }
        catch (error)
        {
            errorArray.push("ERROR: Cant clone from git!")
        }
    }

    stage ('Building code.')
    {
        try
        {
            sh '''
                export PATH=$PATH:${JENKINS_HOME}/tools/hudson.plugins.gradle.GradleInstallation/gradle3.3/bin/
                export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
            gradle build
            ''';
        }
        catch (error)
        {
            errorArray.push("ERROR: Cant build with gradle!")
        }
    }

    stage ('Testing.')
    {
        try
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
        }
        catch (error)
        {
            errorArray.push("ERROR: Something goes wrong with tests!")
        }
    }
    
    stage ('Triggering job and fetching artefact after finishing.')
    {
        try
        {
            build job: 'kickman', parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${env.BRANCH_NAME}"]]
            step ([$class: 'CopyArtifact', projectName: 'kickman']);
        }
        catch (error)
        {
            echo "${error}"
            errorArray.push("ERROR: Cant trigger other project!")
        }
    }
    
    stage ('Packaging and Publishing results.')
    {
        try
        {
            sh '''
            cp ${WORKSPACE}/build/libs/$(basename "$PWD").jar ${WORKSPACE}
            tar -zxvf pheraska_dsl_script.tar.gz jobs.groovy
            tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile $(basename "$PWD").jar
            ''';
            archiveArtifacts artifacts: 'pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz'
        }
        catch (error)
        {
            errorArray.push("ERROR: Cant create artifacts!")
        }
    }

    stage ('Asking for manual approval.')
    {
        try
        {
            timeout(time:3, unit:'MINUTES') 
            {
                input message:'Approve deployment?'
            }
        }
        catch (error)
        {
            errorArray.push("ERROR: Somet wrong with with approve!")
        }
    }
    
    stage ('Deployment.')
    {
        try
        {
            sh '''
            export JAVA_HOME=${JENKINS_HOME}/tools/hudson.model.JDK/java8/
            '''
        }
        catch (error)
        {
            errorArray.push("ERROR: Somet wrong with deployent!")
        }
    }
    
    stage ('Sending status.')
    {
        def arrSize = errorArray.size();
        if (arrSize != 0)
        {
            echo "${error}"
            echo "${errorArray}"
        }
        else
        {
            echo "SUCCESS: No errors found!"
        }
    }
        } 
    });
}

