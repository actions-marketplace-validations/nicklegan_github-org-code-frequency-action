# GitHub Organization Code Frequency Report Action

> A GitHub Action to generate a report that contains code frequency metrics and programming languages used per repository belonging to a GitHub organization.

## Usage

The example [workflow](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions) below runs on a monthly [schedule](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#scheduled-events) using the amount of weeks as an interval set in the workflow (default 4 weeks) and alternatively can also be triggered manually using a [workflow_dispatch](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#manual-events) event.

```yml
name: Code Frequency Action

on:
  schedule:
    # Runs on the first day of the month at 00:00 UTC
    #
    #        ┌────────────── minute
    #        │ ┌──────────── hour
    #        │ │ ┌────────── day (month)
    #        │ │ │ ┌──────── month
    #        │ │ │ │ ┌────── day (week)
    - cron: '0 0 1 * *'
  workflow_dispatch:
    inputs:
      fromdate:
        description: 'Optional interval start date (format: yyyy-mm-dd)'
        required: false # Skipped if workflow dispatch input is not provided
      todate:
        description: 'Optional interval end date (format: yyyy-mm-dd)'
        required: false # Skipped if workflow dispatch input is not provided

jobs:
  code-frequency-report:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Code Frequency Report
        uses: nicklegan/github-org-code-frequency-action@v2.1.0
        with:
          token: ${{ secrets.ORG_TOKEN }}
          fromdate: ${{ github.event.inputs.fromdate }} # Used for workflow dispatch input
          todate: ${{ github.event.inputs.todate }} # Used for workflow dispatch input
        # org: ''
        # weeks: '4'
        # sort: 'additions'
        # sort-order: 'desc'
        # json: 'false'
        # appid: ${{ secrets.APPID }}
        # privatekey: ${{ secrets.PRIVATEKEY }}
        # installationid: ${{ secrets.INSTALLATIONID }}
```

## GitHub secrets

| Name                 | Value                                              | Required |
| :------------------- | :------------------------------------------------- | :------- |
| `ORG_TOKEN`          | A `repo`, `read:org`scoped [Personal Access Token] | `true`   |
| `ACTIONS_STEP_DEBUG` | `true` [Enables diagnostic logging]                | `false`  |

[personal access token]: https://github.com/settings/tokens/new?scopes=repo,read:org&description=Code+Frequency+Action 'Personal Access Token'
[enables diagnostic logging]: https://docs.github.com/en/actions/managing-workflow-runs/enabling-debug-logging#enabling-runner-diagnostic-logging 'Enabling runner diagnostic logging'

:bulb: Disable [token expiration](https://github.blog/changelog/2021-07-26-expiration-options-for-personal-access-tokens/) to avoid failed workflow runs when running on a schedule.

## Action inputs

| Name              | Description                                                                              | Default                     | Options                                                                                            | Required |
| :---------------- | :--------------------------------------------------------------------------------------- | :-------------------------- | :------------------------------------------------------------------------------------------------- | :------- |
| `org`             | Organization different than workflow context                                             |                             |                                                                                                    | `false`  |
| `weeks`           | Amount of weeks in the past to collect data for **(weeks start on Sunday 00:00:00 GMT)** | `4`                         |                                                                                                    | `false`  |
| `sort`            | Column used to sort the acquired code frequency data                                     | `additions`                 | `repoName, additions, deletions, alltimeAdditions, alltimeDeletions, primaryLanguage, createdDate` | `false`  |
| `sort-order`      | Selected column sorting direction                                                        | `desc`                      | `desc, asc`                                                                                        | `false`  |
| `json`            | Generate an additional report in JSON format                                             | `false`                     | `true, false`                                                                                      | `false`  |
| `committer-name`  | The name of the committer that will appear in the Git history                            | `github-actions`            |                                                                                                    | `false`  |
| `committer-email` | The committer email that will appear in the Git history                                  | `github-actions@github.com` |                                                                                                    | `false`  |

## Workflow dispatch inputs

The additional option to retrieve code frequency data using a custom date interval.
If the below fields are left empty during [workflow dispatch input](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/), the default interval option of set weeks from the current date configured in `main.yml` will be used instead.

:bulb: The result data includes the weeks which have their start date **(Sunday 00:00:00 GMT)** within the set interval.

| Name                           | Value                                   | Required |
| :----------------------------- | :-------------------------------------- | :------- |
| `Optional interval start date` | A date matching the format `yyyy-mm-dd` | `false`  |
| `Optional interval end date`   | A date matching the format `yyyy-mm-dd` | `false`  |

## CSV/JSON layout

The results of the 2nd and 3rd report column will be the sum of code frequency date for the selected interval per organization repository.

| Column                     | JSON               | Description                                         |
| :------------------------- | :----------------- | :-------------------------------------------------- |
| `Repository`               | `repoName`         | Organization owned repository                       |
| `Lines added (interval)`   | `additions`        | Number of lines of code added during set interval   |
| `Lines deleted (interval)` | `deletions`        | Number of lines of code deleted during set interval |
| `All time lines added`     | `alltimeAdditions` | Number of lines of code added since repo creation   |
| `All time lines deleted`   | `alltimeDeletions` | Number of lines of code deleted since repo creation |
| `Primary language`         | `primaryLanguage`  | The primary programming language used in the repo   |
| `All languages`            | `allLanguages`     | All programming languages used in the repo          |
| `Repo creation date`       | `createdDate`      | Date the repo has been created                      |

A CSV report file to be saved in the repository **reports** folder using the following naming format: **`organization`-`date`-`interval`.csv**.

## GitHub App authentication

As an alternative you can use GitHub App authentication to generate the report.
For larger organizations it is recommended to use this method as more API requests per hour are allowed which will avoid running into [rate limit](https://docs.github.com/developers/apps/building-github-apps/rate-limits-for-github-apps) errors.

[Register](https://docs.github.com/developers/apps/building-github-apps/creating-a-github-app) a new organization/personal owned GitHub App with the below permissions:

| GitHub App Permission                     | Access           |
| :---------------------------------------- | :--------------- |
| `Repository Permissions:Contents`         | `read and write` |
| `Organization Permissions:Administration` | `read`           |

After registration install the GitHub App to your organization. Store the below App values as secrets.

### GitHub App secrets

| Name             | Value                             | Required |
| :--------------- | :-------------------------------- | :------- |
| `APPID`          | GitHub App ID number              | `true`   |
| `PRIVATEKEY`     | Content of private key .pem file  | `true`   |
| `INSTALLATIONID` | GitHub App installation ID number | `true`   |
