#!/usr/bin/env groovy

def cleanup_docker(imageId) {
  sh "docker rmi ${imageId}"

  // Build stages in dockerfiles leave dangling images behind (see https://github.com/moby/moby/issues/34151).
  // Dangling images are images that are not used anywhere and don't have a tag. It is safe to remove them (see https://stackoverflow.com/a/45143234).
  // This removes all dangling images
  sh "docker image prune --force"

  // Some Dockerfiles create volumes using the `VOLUME` command (see https://docs.docker.com/engine/reference/builder/#volume)
  // running the speedtests creates two dangling volumes. One is from postgres (which contains data), but i don't know about the other one (which is empty)
  // Dangling volumes are volumes that are not used anywhere. It is safe to remove them.
  // This removes all dangling volumes
  sh "docker volume prune --force"
}

def cleanup_workspace() {
  cleanWs()
  dir("${env.WORKSPACE}@tmp") {
    deleteDir()
  }
  dir("${env.WORKSPACE}@script") {
    deleteDir()
  }
  dir("${env.WORKSPACE}@script@tmp") {
    deleteDir()
  }
}

pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        script {
          branch_is_master = env.BRANCH_NAME == 'master';

          if (branch_is_master) {
            image_tag = "b${env.BUILD_NUMBER}";
          } else {
            def cleaned_branch_name = env.BRANCH_NAME.replace('/', '_');
            image_tag = "${cleaned_branch_name}-b${env.BUILD_NUMBER}";
          }

          image_name = '5minds/selenium_dockerized';
          full_image_name = "${image_name}:${image_tag}"

          sh("docker build --no-cache --tag ${full_image_name} .")

          dockerImage = docker.image(full_image_name);
        }
      }
    }
    stage('publish') {
      steps {
        withDockerRegistry([ credentialsId: "5mio-docker-hub-username-and-password", url: "" ]) {
          script {
            dockerImage.push();

            if (branch_is_master) {
              dockerImage.push('latest');
            }
          }
        }
      }
    }
  }
  post {
    always {
      script {
        cleanup_workspace();

        // Ignore any failures during docker clean up.
        // 'docker image prune --force' fails if
        // two builds run simultaneously.
        try {
          cleanup_docker(full_image_name);
        } catch (Exception error) {
          echo "Failed to cleanup docker $error";
        }
      }
    }
  }
}
