#!/usr/bin/env groovy

def cleanup_docker() {
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
  dir("${WORKSPACE}@tmp") {
    deleteDir()
  }
  dir("${WORKSPACE}@script") {
    deleteDir()
  }
  dir("${WORKSPACE}@script@tmp") {
    deleteDir()
  }
}

NODE_VERSIONS = [
  '8-stretch',
  '10-stretch',
];

NODE_VERSION_FOR_LATEST_TAG = '10-stretch';

IMAGE_NAME = '5minds/selenium_dockerized';


def create_dockerfile_build_step(node_version) {
  return {
    stage('Build and Publish') {

      def branch_is_master = BRANCH_NAME == 'master';

      def base_image_tag = "b${BUILD_NUMBER}";

      if (!branch_is_master) {
        def cleaned_branch_name = BRANCH_NAME.replace('/', '_');
        base_image_tag = "${cleaned_branch_name}-b${BUILD_NUMBER}";
      }

      def full_image_name = "${IMAGE_NAME}:${node_version}_${base_image_tag}";

      try {
        sh("docker build --build-arg NODE_IMAGE_VERSION=${node_version} --no-cache --tag ${full_image_name} .");
        def docker_image = docker.image(full_image_name);

        withDockerRegistry([ credentialsId: "5mio-docker-hub-username-and-password" ]) {
          docker_image.push();

          if (branch_is_master) {
            docker_image.push("${node_version}-latest");

            if (node_version == NODE_VERSION_FOR_LATEST_TAG) {
              docker_image.push('latest');
            }
          }
        }

      } finally {
        sh("docker rmi ${full_image_name} || true");
      }
    }
  }
}

node('docker') {
  checkout scm;

  def dockerfile_builds = [:];

  NODE_VERSIONS.each {

    dockerfile_builds[it] = create_dockerfile_build_step(it);

  };

  parallel dockerfile_builds;

  cleanup_workspace();

  // Ignore any failures during docker clean up.
  // 'docker image prune --force' fails if
  // two builds run simultaneously.
  try {
    cleanup_docker();
  } catch (Exception error) {
    echo "Failed to cleanup docker $error";
  }

}
