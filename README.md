# Gitlab Knowledge Base


## How to release binary files

Gitlab does not support binary files for the release like GitHub, so we exploit the artifact storage to keep the jar file available forever and attach a link to the artifact in the release notes

```

release_image:
  stage: build_jar
  image: $MAVEN_GIT_IMAGE
  script:
    - mvn clean package
    - export JAR_FILE=$(ls target/*.jar | grep -v 'original' | head -n 1)
    - 'echo Built JAR file: $JAR_FILE'
    - 'echo uploadURL is ${PACKAGE_REGISTRY_URL}/$(basename "$JAR_FILE")'
    - 'mv "$JAR_FILE" pubsub-collector.jar'
  after_script:
    - echo "RELEASE_JOB_ID=$CI_JOB_ID" >> job.env # we save the job id to be able to link to the artifact in the release notes
  artifacts:
    reports:
      dotenv: job.env
    paths:
      - collector.jar
    expire_in: never # Gitlab does not support binary files for the release like GitHub,
    # so we exploit the artifact storage to keep the jar file available forever and attach a
    # link to the artifact in the release notes
  rules:
    ...

##### GitLab Release
Create GitLab Tag & Release:
  stage: gitlab_release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  script:
    - echo "Creating Tag & Release for version $CI_COMMIT_TAG"
  release:
    ref: $CI_COMMIT_TAG
    name: "release $CI_COMMIT_TAG"
    tag_name: $CI_COMMIT_TAG
    description: $CHANGELOG_PATH
    assets:
      links:
        - name: 'Jar Artifact'
          url: "https://gitlab.com/code/to/my/repo/-/jobs/$RELEASE_JOB_ID/artifacts/download?file_type=archive"
  rules:
    ...

```

