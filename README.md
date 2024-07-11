# github-workflow-dispatch-test
Trigger a workflow from another workflow

# Why this repo ?

* This was created in order to test differnt methos to trigger a workflow other than push event, or schedule etc.

* This repo contains 2 different methods we can trigger a workflow from another workflow


# Details

## workflow_dispatch method

* We will utilize GitHub REST API end point for triggering a workflow by providing necessary credential.

[create-a-workflow-dispatch-event](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28#create-a-workflow-dispatch-event)

i am passing ***GitHub app installation access*** token to trigger the workflow. Similarly we can pass either *Github app user access token* or *fine grained personal access tokens*

```
- name: Token generator
  uses: githubofkrishnadhas/github-access-using-githubapp@v1
  with:
    github_app_id: ${{ secrets.WDC_APP_ID }}
    github_app_private_key : ${{ secrets.WDC_PVT_KEY }}
    github_account_type : organization

```

* use the above action to generate a GitHub app Installation token and later pass it to the GItHub rest API call to do the authentication. The token will be available as **GH_APP_TOKEN** env variable in workflow.

* `WDC_APP_ID` , `WDC_PVT_KEY` are respectively the app id and private key of github app which is stored as secrets in github .

```
    - name: Trigger another workflow via API
      env:
        OWNER_REPO: ${{ github.repository }}
        WORKFLOW_ID: workflow_dispatch_child.yaml  # or the workflow ID if you prefer
        REF: main  # or the branch you want to use
      run: |
        curl -L \
          -X POST \
          -H "Accept: application/vnd.github+json" \
          -H "Authorization: Bearer $GH_APP_TOKEN" \
          https://api.github.com/repos/${{ env.OWNER_REPO }}/actions/workflows/$WORKFLOW_ID/dispatches \
          -d '{"ref":"main","inputs":{"name":"Kunai the Octocat","home":"Tokiyo, Japan"}}'
        
```

* OWNER_REPO --> this will be the complete repo name in format of `github orgname/repository name`

* WORKFLOW_ID --> This can be either the workflow file name or ID of the workflow 

* Inside inputs you can pass necessary params to next workflow.

* This will trigger a workflow named `workflow_dispatch_child.yaml` post completing the API call and pass name and home as inputs for next workflow.


## worlfow_run method

* `workflow_run` method provides more importance to workflow name. here it is `Build`

```
name: Build
on:
  workflow_dispatch: # this is optional
  workflow_run:
  # This workflow will only run in main branch when Trigger a workflow using workflow_run is comlpeted
    workflows: ["Trigger a workflow using workflow_run"] 
    types: [completed]
      # branches-ignore:
      #   - "feature/*"
    branches:
      - "main"

```

* The name inside workflows: ["x"] , this x will be the workflow name and the workflow which will be triggered first then Build.

* types : There are 3 values  [workflow_run](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_run)

    * completed
    * in-progress
    * requested

* when using `requested`, as soon as the parent workflow is triggered child is also. The requested activity type does not 
  occur when a workflow is re-run.

* when using `in-progress`, as the parent workflow is executing child workflow is triggered

* when using `completed`, the child workflow will wait till parent is completed.

* `branches`: this we can provide a list of branch names or with regex [eg: feature/* for all feature branches starting with feature/ in name] to run on these branches

* `branches-ignore`: This we can also provide a list of names or also with regex . the workflow wont run on branches
  specifiedd under branches-ignore

* **`branches` and `branches`-ignore can not be used simultaneously.**


just for identification added print statements on child workflow ro specify from which workflo_run, the job is triggered.

![alt text](<Print triggering workflow and build number.jpeg>)
