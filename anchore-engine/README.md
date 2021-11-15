## Examples for using the anchore/anchore-engine@1.8 CircleCi orb:

Adding a public image scan job to a CircleCi workflow:
```
version: 2.1
orbs:
  anchore: anchore/anchore-engine@1
workflows:
  scan_image:
    jobs:
      - anchore/image_scan:
          image_name: anchore/anchore-engine:latest
```

Adding a private image scan job to a CircleCi workflow:
```
version: 2.1
orbs:
  anchore: anchore/anchore-engine@1
workflows:
  scan_image:
    jobs:
      - anchore/image_scan:
          image_name: anchore/anchore-engine:latest
          private_registry: True
          registry_name: docker.io
          registry_user: "${DOCKER_USER}"
          registry_pass: "${DOCKER_PASS}"
```
Adding image scanning to your container build pipeline job.
```
version: 2.1
orbs:
  anchore: anchore/anchore-engine@1
jobs:
  local_image_scan:
    executor: anchore/anchore_engine
    steps:
      - setup_remote_docker
      - checkout
      - run:
          name: build container
          command: docker build -t "anchore/anchore-engine:ci" .
      - anchore/analyze_local_image:
          image_name: example/test:latest
          dockerfile_path: ./Dockerfile
      - store_artifacts:
          path: anchore-reports
```

Put a custom policy bundle in to your repo at .circleci/.anchore/policy_bundle.json
Job will be marked as 'failed' if the Anchore policy evaluation gives 'fail' status
```
version: 2.1
orbs:
  anchore: anchore/anchore-engine@1
jobs:
  local_image_scan:
    executor: anchore/anchore_engine
    steps:
      - checkout:
          path: ~/project/src/
      - run:
          name: build container
          command: docker build -t ${CIRCLE_PROJECT_REPONAME}:ci ~/project/src/
      - anchore/analyze_local_image:
          image_name: ${CIRCLE_PROJECT_REPONAME}:ci
          dockerfile_path: ./Dockerfile
      - anchore/parse_reports
      - store_artifacts:
          path: anchore-reports
```

Build and scan multiple images, using a custom policy bundle.
```
version: 2.1
orbs:
  anchore: anchore/anchore-engine@1
jobs:
  local_image_scan:
    executor: anchore/anchore_engine
    steps:
      - setup_remote_docker
      - checkout
      - run:
          name: build containers
          command: |
            docker build -t "example/test:dev" dev/
            docker build -t "example/test:staging" staging/
            docker build -t "example/test:latest" prod/
      - anchore/analyze_local_image:
          image_name: "example/test:dev example/test:staging example/test:latest"
          policy_failure: True
      - anchore/parse_reports
      - store_artifacts:
          path: anchore-reports
```
