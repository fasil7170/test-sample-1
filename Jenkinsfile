pipeline {
agent any

environment {
    DOCKER_IMAGE="nexus.local:5000/demo-app"
}

stages {

stage('Checkout') {
steps {
git 'https://github.com/yourrepo/app.git'
}
}

stage('Unit Test') {
steps {
sh 'mvn test'
}
}

stage('SonarQube Analysis') {
steps {
withSonarQubeEnv('sonar-server') {
sh 'mvn sonar:sonar'
}
}
}

stage('Trivy FS Scan') {
steps {
sh 'trivy fs .'
}
}

stage('Build Package') {
steps {
sh 'mvn clean package'
}
}

stage('Upload to Nexus') {
steps {
sh 'mvn deploy'
}
}

stage('Docker Build') {
steps {
sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
}
}

stage('Image Scan') {
steps {
sh 'trivy image $DOCKER_IMAGE:$BUILD_NUMBER'
}
}

stage('Docker Push') {
steps {
sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
}
}

stage('Update Manifest Repo') {
steps {
sh '''
sed -i "s/tag:.*/tag: $BUILD_NUMBER/" k8s/deployment.yaml
git commit -am "update image"
git push
'''
}
}

}
}
