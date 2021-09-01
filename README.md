# circleci-monorepo-with-setup-workflow

A test repository to try monorepo support using CircleCI with setup workflow. This features is in the [status of open preview](https://discuss.circleci.com/t/setup-workflows-open-preview/39516) and will potentially change.

## Project structure

```
.
├── .circleci
│   ├── config.yml  # general CircleCI config
│   └── setup-workflow-config.yml # job and workflow configurations for all projects
├── app1
└── app2
```

## Can we split setup-workflow.config.yml per project?

No. Multiple path-filer orb jobs like

```
#.circleci/config.yml

version: 2.1

setup: true

orbs:
  # https://github.com/CircleCI-Public/api-preview-docs/blob/path-filtering/docs/path-filtering.md
  path-filtering: circleci/path-filtering@0.0.2

workflows:
  setup-workflow:
    jobs:
      # multiple configs for setup-workflow cause {"message":"Pipeline is not in setup state."} error
      - path-filtering/filter:
          config-path: .circleci/app1.yml
          mapping: |
            app1/.* app1 true
          base-revision: origin/master
      - path-filtering/filter:
          config-path: .circleci/app2.yml
          mapping: |
            app2/.* app2 true
          base-revision: origin/master
```


produce the error below.

```
{
  "continuation-key": "************************************************************************************************************************************************************************************************************************************************************************************************************************",
  "configuration": "# Use the latest 2.1 version of CircleCI pipeline process engine. See: https://circleci.com/docs/2.0/configuration-reference\nversion: 2.1\n\nparameters:\n  app2:\n    type: boolean\n    default: false\n\n# See https://circleci.com/docs/2.0/configuration-reference/#jobs\njobs:\n  job1:\n    #working_directory: ~/my-app\n    docker: # executor type\n      - image: buildpack-deps:trusty # primary container will run Ubuntu Trusty\n    steps:\n      - run:\n          name: list files\n          command: ls -l\n\nworkflows:\n  version: 2\n  app2:\n    when: << pipeline.parameters.app2 >>\n    jobs:\n      - job1\n",
  "parameters": {
    "app2": true
  }
}
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1114  100    45  100  1069   1551  36862 --:--:-- --:--:-- --:--:-- 38413
{"message":"Pipeline is not in setup state."}

Exited with code exit status 1
CircleCI received exit code 1
```

So we cannot explicitly configure workflow per application like,

```
      - path-filtering/filter:
          config-path: .circleci/app1.yml
          mapping: |
            apps/app1/.* app1 true
          base-revision: origin/master
      - path-filtering/filter:
          config-path: .circleci/app2.yml
          mapping: |
            apps/app2/.* app2 true
          base-revision: origin/master
```


## References

* [Setup Workflows Open Preview - Announcements - CircleCI Discuss](https://discuss.circleci.com/t/setup-workflows-open-preview/39516)
