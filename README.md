# gke-cluster-metrics-alerts-send-on-google-workspace

## Site Link

    https://cloud.google.com/stackdriver/docs/managed-prometheus/exporters/node_exporter

Step to accomplish the task.

- Install node exporters on kubernets cluster gmp-public namespace
- Enable manager promethoues service on kubernetes cluster and manage collector on monitoring section
- Create notification chennel service like **GOOGLE CHAT** under gcp cloud alert notification chennel. For this you need to give google chat space id here. You can get this from your google chat workspace. In order to add Google Chat as a notification channel, you must first add the **Google Cloud Monitoring** App to the chat space. You can add the app directly to a space by typing @Google Cloud Monitoring. I have checked google cannot install google cloud monitoring app by just typing on google chat work space console. You need to click on "+" there and search you google cloud monitoring app in search bar and then click on it to make it installable. After successfully installing the app. it would give you space-id that you can use in gcp alerts under notification chennel. And use this notification chennel in you alerts. GCP will send the alert notification to specfic assign destination..
- You can also create dashboard for monitoring

## link for creating space-id 

    https://cloud.google.com/monitoring/support/notification-options#google-chat


## Sending alerts logs to teams chennel or google chat space

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


## For sending logs to team

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
          "short_description": "Alert Notification: Elasticsearch query rule '{{context.rule.name}}' is active.",
          "log_levels": "{{#context.hits}}{{#_source}}{{logLevel}}{{/_source}}{{/context.hits}}",
          "instances": "{{#context.hits}}{{#_source}}{{instance}}{{/_source}}{{/context.hits}}",
          "transaction_id": "{{#context.hits}}{{#_source}}{{transactionId}}{{/_source}}{{/context.hits}}",
          "message": "{{#context.hits}}{{#_source}}{{message}}{{/_source}}{{/context.hits}}",
          "timestamp": "{{context.date}}",
          "link": "{{context.link}}",
           "all_context": "{{.}}"
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



