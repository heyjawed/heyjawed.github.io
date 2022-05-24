---
title:  "Send Jenkins job/pipeline alerts on MS Teams without using any plugin"
# image : "/assets/images/post/post-1.jpg"
author: "Jawed Salim"
date: 2022-05-24 10:35:58 +0600
description : "Use Jenkins Shared Library"
tags: [Jenkins]
---

Pipeline has support for creating "Shared Libraries" which can be defined in external source control repositories and loaded into existing Pipelines.

Shared Libraries marked Load implicitly allows Pipelines to immediately use classes or global variables defined by any such libraries. To access other shared libraries, the Jenkinsfile needs to use the @Library annotation, specifying the library’s name:

## Configure incoming webhook on MS teams

Refer - https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook

> Note down the Webhook URL somewhere for later use

## Create and Send message using CURL

Refer - https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL

**Example message**

```sh

curl --request POST <WEBHOOK_URL> \
     -H 'Content-Type: application/json' \
     -d '{"text": "Hello World"}'
```

## Integrate with Jenkins Job/Pipeline

- **Login** to Jenkins UI, Go to **Manage Jenkins** --> **Mange Credentials [Security]**
- **Add creadential** for Webhook URL
  - *Kind*   - `Secret Text`
  - *Secret* - `<Webhook_url>`
  - *ID*     - unique identifier like: `ms_teams_webhook`

- Now use the below example code inside your pipeline code

**Example code**

```groovy



script {
        // Get MS Teams webhook URL from credential
        withCredentials([string(credentialsId: 'ms_teams_webhook', variable: 'WEBHOOK_SECRET')]) {
            // Store Webhook URL in a variable
            URL_WEBHOOK = WEBHOOK_SECRET
        }

        // echo "$URL_WEBHOOK"

        sh """#!/bin/bash

        curl --location --request POST '$URL_WEBHOOK' \
            --header 'Content-Type: application/json'   \
            --data-raw '{
                "@type": "MessageCard",
                "@context": "http://schema.org/extensions",
                "themeColor": "bd8feb",
                "summary": "Notification from [${env.JOB_NAME}]",
                "sections": [{
                    "activityTitle": "Notification from [${env.JOB_NAME}]",
                    "activitySubtitle": "<span style=\'\\\'\'color: #bd8feb;\'\\\'\'>Latest status of build #${env.BUILD_NUMBER}</span>",
                    "activityImage": "https://www.jenkins.io/images/logos/jenkins/jenkins.png",
                    "facts": [
                        {
                            "name": "Pipeline",
                            "value": "[${env.JOB_NAME}](${env.BUILD_URL}/console)"
                        },
                        {
                            "name": "Git Branch",
                            "value": "`${GIT_BRANCH}`"
                        },
                        {
                            "name": "Build Number",
                            "value": "${env.BUILD_DISPLAY_NAME}"
                        },
                        {
                            "name": "Build Status",
                            "value": "STARTED"
                        },
                        {
                            "name": "Total Runtime",
                            "value": "${currentBuild.durationString}"
                        }],
                    "markdown": true
                }],
                "potentialAction": [{
                    "@type": "OpenUri",
                    "name": "View Build",
                    "targets": [{
                    "os": "default",
                    "uri": "${env.BUILD_URL}"
                }]
            }]
        }'
    """
}
```
