pipeline {
  agent any

  tools {
    maven "M3"
    // jdk "jdk21"
  }

  environment {
      def mvn = tool 'M3'
      // JAVA_HOME = "/Users/macbookpro/Library/Java/JavaVirtualMachines/openjdk-21.0.2"

      NEXUS_VERSION = "nexus3"
      NEXUS_PROTOCOL = "http"
      NEXUS_URL = "127.0.0.1:8081/#admin"
      NEXUS_REPOSITORY = "Devops-maven-jenkins-sonarqube-nexus-Pipeline"
      NEXUS_CREDENTIAL_ID = "nexus_credentials"
      ARTIFACT_VERSION = "1.0-SNAPSHOT"
  }

  stages {

    // stage('Stage to debog jdk env variable issues') {
    //     steps {
    //         sh "echo $JAVA_HOME"
    //     }
    // }

    stage('Checkout') {
      steps{
        git branch: 'main', url: 'https://github.com/ajifatu/Devops-maven-jenkins-sonarqube-nexus-Pipeline.git'
      }
    }

    stage('Build') {
      steps {
        sh "${mvn}/bin/mvn clean package "
        sh "/Users/macbookpro/.jenkins/tools/hudson.tasks.Maven_MavenInstallation/M3/bin/mvn clean package -Dmaven.compiler.fork=true -Dmaven.compiler.executable=/Users/macbookpro/Library/Java/JavaVirtualMachines/openjdk-21.0.2/bin/javac"

      }
    }

    stage('SonarQube Analysis') {
      steps{
        withSonarQubeEnv('sonar-server') {
        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=Devops-maven-jenkins-sonarqube-nexus-Pipeline -Dsonar.projectName=Devops-maven-jenkins-sonarqube-nexus-Pipeline -Dsonar.host.url=http://localhost:9000 -Dsonar.login=squ_d5becd72989777bc860f4e7601f1ceadf17e0ce2"
      }
      }
    }

  stage('Publish to nexus') {
    steps {
      script {
        // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
        pom = readMavenPom file: "pom.xml";
        // Find built artifact under target folder
        filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
        // Print some info from the artifact found
        echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
        // Extract the path from the File found
        artifactPath =  [0].path;
        // Assign to a boolean response verifying If the artifact name exists
        artifactExists = fileExists artifactPath;

        echo "Before if condition"
        if(artifactExists) {
            echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
            echo "Artifact actually exists"
            nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,
                protocol: NEXUS_PROTOCOL,
                nexusUrl: NEXUS_URL,
                groupId: pom.groupId,
                version: ARTIFACT_VERSION,
                repository: NEXUS_REPOSITORY,
                credentialsId: NEXUS_CREDENTIAL_ID,
                artifacts: [
                    // Artifact generated such as .jar, .ear and .war files.
                    [artifactId: pom.artifactId,
                    classifier: '',
                    file: artifactPath,
                    type: pom.packaging]
                ]
            );

        } else {
            echo "Artifact does not exists"
            error "*** File: ${artifactPath}, could not be found";
        }
        echo "After if condition"
      }
    }
  }

  //  stage('Publish to nexus') {
  //   steps {
  //     nexusArtifactUploader(
  //       nexusVersion: NEXUS_VERSION,
  //       protocol: NEXUS_PROTOCOL,
  //       nexusUrl: NEXUS_URL,
  //       groupId: 'sn.dev',
  //       version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
  //       repository: NEXUS_REPOSITORY,
  //       credentialsId: NEXUS_CREDENTIAL_ID,
  //       artifacts: [
  //           [artifactId: "Devops-maven-jenkins-sonarqube-nexus-Pipeline",
  //            classifier: '',
  //           //  file: 'target/Devops-maven-jenkins-sonarqube-nexus-Pipeline.jar',
  //           //  type: 'jar']
  //           file: "target/maven-sample-1.0-SNAPSHOT.jar",
  //           type: 'jar']
  //       ]
  //    )
  //   }
  // }


 }
}



