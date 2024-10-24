# Steps for Configuring GCP Alerting for GKE Pods Across All Projects:

        Install Node Exporter and Kube-State Metrics Exporter:
        
        I installed the Node Exporter and Kube-State Metrics Exporter in all GKE clusters to collect relevant metrics.
        Enable Managed Prometheus:
        
        I enabled Managed Prometheus on each GKE cluster and activated the Managed Collector via the GKE cluster dashboard.
        Additionally, I enabled 13 components for cloud monitoring in each GKE cluster under GKE Cluster > Details > Features > Cloud Monitoring. These components include:
        System, API Server, Scheduler, Controller Manager, Persistent Volume (Storage), Pods, Deployment, StatefulSet, DaemonSet, Horizontal Pod Autoscaler, cAdvisor, and Kubelet.
        Verify Metrics Collection:
        
        To verify that the metrics were being collected correctly, I navigated to the GKE Cluster Dashboard > Metrics Explorer and ran the "up" query in PromQL.
        The query returned logs, confirming that the Node Exporter and Kube-State Metrics were correctly configured in the cluster.
        Aggregate Metrics from All Projects:
        
        Since I wanted to gather metrics from all GCP projects' GKE clusters into a single GCP project, I went to GKE Cluster Dashboard > Observability Scope > Metric Scope.
        There, I added all other GCP projects to the current GCP project (e.g., disearch), allowing me to collect metrics from all the other projects into one place.
        Configure Alerts:
        
        GCP already provides recommended alerts for GKE clusters, such as Pod Memory Utilization, Pod CPU Utilization, and Pod Restarts.
        To enable these alerts:
        Go to Kubernetes Engine > Observability and select the desired metric (Memory, CPU, etc.).
        Click on Recommended Alerts, which include 3 pre-configured alerts for Pod CPU, Pod Memory, and Pod Restarts.
        Select all three alerts and configure a notification channel by clicking on Manage Notifications. For the destination, I chose Google Chat and specified the desired Google Chat space for the alerts to be sent.
        Once saved, GCP will automatically create these 3 alerts for the GKE clusters in the current project.
        Customize Alerts for All Projects:
        
        By default, GCP creates alerts specific to the current project's GKE cluster, applying filters like project name or cluster ID.
        To make the alerts applicable across all projects:
        I navigated to the alerts in the current project, removed the filters that limited the alerts to a single project, and made them standard alerts for all projects.
        Now, these alerts will collect metrics from all GKE clusters across all GCP projects. If any cluster pod exceeds the threshold in any project, a notification will be sent to the specified Google Chat space.
        By following these steps, I successfully created 3 alerts that are responsible for collecting and sending notifications for all GCP project GKE clusters from a single GCP project.


## gke-cluster-metrics-alerts-send-on-google-workspace

## Site Link

    https://cloud.google.com/stackdriver/docs/managed-prometheus/exporters/node_exporter

Step to accomplish the task.

- Install node exporters on kubernets cluster gmp-public namespace. 
- Enable manager promethoues service on kubernetes cluster and manage collector on monitoring section(you will get managed collector option by searching "gke cluster dashboard" on gcp console). Now the install exports will export the metric and promethous services will get time series metric and collector will collect it.. You can also verify that your node exporter is successfully exporting gke infrastructure metrics by using below promql query on  **gcp "gke cluster dashboard" under "metrics explorer" in promql section**

    up{job="node-exporter", cluster="disearch-cluster", namespace="gmp-public", project_id="diseracharetecprod"}

    

- Now you can create monitoring dashboard on google cloud with that metrics. And you can also create alerts base on that metrics. And send alerts to specfic destination. like In my case i am sending alert by using notification chennel named gcp cloud google chat to google chat workspace.
- Create notification chennel service like **GOOGLE CHAT** under gcp cloud alert notification chennel. For this you need to give google chat space id here. You can get this from your google chat workspace. In order to add Google Chat as a notification channel, you must first add the **Google Cloud Monitoring** App to the chat space. You can add the app directly to a space by typing @Google Cloud Monitoring. I have checked google cannot install google cloud monitoring app by just typing on google chat work space console. You need to click on "+" there and search you google cloud monitoring app in search bar and then click on it to make it installable. After successfully installing the app. it would give you space-id that you can use in gcp alerts under notification chennel. And use this notification chennel in you alerts. GCP will send the alert notification to specfic assign destination..
- You can also create dashboard for monitoring

## link for creating space-id 

    https://cloud.google.com/monitoring/support/notification-options#google-chat


## Sending application alerts logs to teams chennel or google chat space from elastic cloud

Below are the links that would help on this..


For alerts webhook configuration

    https://www.elastic.co/guide/en/kibana/current/webhook-action-type.html
    or 
    
    https://www.elastic.co/guide/en/kibana/current/cases-webhook-action-type.html
    
    or 

    https://www.elastic.co/guide/en/kibana/current/create-and-manage-rules.html#defining-rules-actions-details

   or 

   https://www.elastic.co/guide/en/elasticsearch/reference/current/actions-webhook.html


    
For making elastic search query

    https://www.elastic.co/guide/en/kibana/current/rule-type-es-query.html#_add_action_variables_2


## For sending logs to team from elastic cloud

you need to create a chennels on team and get chennel email from team. You can easily get email by right clicking on teams chennel and you can use it elastic search alerts chennel for sending application logs on team..

    example on setting up alerts rule for error

        NAME: Alert name
        tag: Error
        Elasticsearch query: KQL or Lucene(Use KQL or Lucene to define a text-based query)
        Select a data view: data view(disearch_dev)               ----> for this you first make it sure the your application is sending indexes on elastic search or not.. You can see this in **Index management**. Once confirm go to **Data veiw** section and use that index for creating a data veiw. Once data veiw will create, you can use this in your alerts.
        
        Now Define your query:
                logLevel : ERROR
                
                Set the group, threshold, and time window 
                
                when
                count()
                
                over
                all documents
                
                Is above
                0
                
                for the last
                30 seconds
                Set the number of documents to send
                
                Size
                100 
        Check every: 1 minute
        Actions section:
        
        Connector name: WEBHOOK3
        
        Webhook connector  ----> here you need to add webhook connector(url) and give name of the connector. you can take connector on the destination where you want to send application logs. Like in my case i am sending logs to google chat space and on teams chennel. for google chat space i create webhook there and got webhook connector from there and using that connector here.
        
        Add connector
        WEBHOOK3
        
        Action frequency
        
        For each alert: On check intervals
        Run when: Query matched
        
        Body       ----> webhook body template in json format

        {
          "text": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active:\nLog Levels: '{{#context.hits}}{{#_source}}{{logLevel}}{{/_source}}{{/context.hits}}'\nInstances: '{{#context.hits}}{{#_source}}{{instance}}{{/_source}}{{/context.hits}}'\nTransactionId: '{{#context.hits}}{{#_source}}{{transactionId}}{{/_source}}{{/context.hits}}'\nMessage: '{{#context.hits}}{{#_source}}{{message}}{{/_source}}{{/context.hits}}'\nTimestamp: '{{context.date}}'\nLink: '{{context.link}}'",
          "formattedText": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active:\n* Timestamp: '{{context.date}}'\n* Link: '{{context.link}}'"
        }

        
        
**did same for send logs to teams using email connector**
        
        Elastic-Cloud-SMTP (preconfigured)
        
        Email connector
        
        Add connector
        Elastic-Cloud-SMTP
        
        Action frequency
        
        For each alert: On check intervals
        Run when:Query matched
        To:33bb2100.aretecinc.com@amer.teams.ms              -----> got from teams chennels. where i am sending logs 
        
        Cc
        Bcc
        
        Subject: ALERTS
        
        Message
        
        Elasticsearch query rule '{{rule.name}}' is active:
        
        - Log Levels: {{#context.hits}}{{#_source}}{{logLevel}}{{/_source}}{{/context.hits}}
        - Instances: {{#context.hits}}{{#_source}}{{instance}}{{/_source}}{{/context.hits}}
        - TransactionId: {{#context.hits}}{{#_source}}{{transactionId}}{{/_source}}{{/context.hits}}
        - Message: {{#context.hits}}{{#_source}}{{message}}{{/_source}}{{/context.hits}}
        - Timestamp: {{context.date}}
        - Link: {{context.link}}


## curl command for testing the google chat space webhook 

        curl -X POST \
          'https://chat.googleapis.com/v1/spaces/AAAAjnK_a3M/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=JfKiIUzuSZX8UWIDjntsicO6Ak7_hB5e47pbPmmO0EY' \
          -H 'Content-Type: application/json' \
          -d '{
            "text": "Alert: Elasticsearch query rule is active.\n\n- **Log Level**: ERROR\n- **Instance**: app-instance-01\n- **Transaction ID**: 12345\n- **Message**: Error message details\n- **Timestamp**: 2024-10-15T08:00:00Z\n- **Link**: https://your-log-url.com"
          }'
## Test connector

    you can also test your connector like email, teams, webhook.. etc in connectors section.. For this go to **Stack Management > Alerts and Insights> connectors**.

    And you can also see connectors logs under connectors section... 

## I was testing the connector for webhook. but faced 400 bad request error. I was using the below information

        Webhook name: WEBHOOK4
        Connector setting:
        method: POST
        URL: https://chat.googleapis.com/v1/spaces/AAAAjnK_a3M/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=JfKiIUzuSZX8UWIDjntsicO6Ak7_hB5e47pbPmmO0EY      ---> WEBHOOK URL GOT FROM GOOGLE CHAT WORKSPACE
        Authentication: none
        body: 
        
        {
          "short_description": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active.",
          "timestamp": "{{context.date}}",
          "link": "{{context.link}}",
          "all_context": "{{.}}"
        
        }
        
        but i with this i was facing 400bad request error.

## Solution

        During searching, I found that google chat workspace support json but with this format.
        
        {
          "text": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active. Timestamp: {{context.date}} Link: {{context.link}} All Context: {{.}}"
        }
        
        "text", should be included. I resolve first issue.. 
        
        RESOLVED 2ND ISSUE BY ADD Add **HTTP header** in my webhook connector. like given below 
        
        HTTP header
        Key: Content-Type
        Value: application/json

**After that i was able to send messages to google chat workspace.. Below are the complete corrected information..**

-    Webhook name: WEBHOOK4
-    Connector setting:
-    method: POST
-    URL: https://chat.googleapis.com/v1/spaces/AAAAjnK_a3M/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=JfKiIUzuSZX8UWIDjntsicO6Ak7_hB5e47pbPmmO0EY      ---> WEBHOOK URL GOT FROM GOOGLE CHAT WORKSPACE
-    Authentication: none
-    body:   
        
        {
          "text": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active. Timestamp: {{context.date}} Link: {{context.link}} All Context: {{.}}"
        }


-    HTTP header
        Key: Content-Type
        Value: application/json

## If you do not get any idea, how to format the logs.. 

    use this in action body. It will give you complete logs. And you will get to know what flag you need to filter out from all logs information

   {
    "test": "All Context: {{.}}"
    }

    it will send all logs to the destination. You can easily filter require information from logs. 

    i got too much help by using above approach. like using below format to get all logs output and then filter the flage according to the requirement. 

     {
    "test": "All Context: {{.}}"
    }

## Correct webhook body format payload for extracting information from logs.

        {
          "text": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active:\nLog Levels: '{{#context.hits}}{{#_source}}{{logLevel}}{{/_source}}{{/context.hits}}'\nInstances: '{{#context.hits}}{{#_source}}{{instance}}{{/_source}}{{/context.hits}}'\nTransactionId: '{{#context.hits}}{{#_source}}{{transactionId}}{{/_source}}{{/context.hits}}'\nMessage: '{{#context.hits}}{{#_source}}{{message}}{{/_source}}{{/context.hits}}'\nTimestamp: '{{context.date}}'\nLink: '{{context.link}}'",
          "formattedText": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active:\n* Timestamp: '{{context.date}}'\n* Link: '{{context.link}}'"
        }

       webhook body payload will extract information from logs and webhook will send it to destination..

BOLD TEXT

        {
          "text": "Alert Notification: Elasticsearch query rule '{{context.alertName}}' is active:\n*Log Levels*: '{{#context.hits}}{{#_source}}{{logLevel}}{{/_source}}{{/context.hits}}'\n*Instances*: '{{#context.hits}}{{#_source}}{{instance}}{{/_source}}{{/context.hits}}'\n*TransactionId*: '{{#context.hits}}{{#_source}}{{transactionId}}{{/_source}}{{/context.hits}}'\n*Message*: '{{#context.hits}}{{#_source}}{{message}}{{/_source}}{{/context.hits}}'\n*Timestamp*: '{{context.date}}'\n*Link*: '{{context.link}}'",
          "formattedText": "Alert Notification: Elasticsearch query rule '{{context.alertName}}' is active:\n* Timestamp: '{{context.date}}'\n* Link: '{{context.link}}'"
        }


