# gke-cluster-metrics-alerts-send-on-google-workspace

## Site Link

    https://cloud.google.com/stackdriver/docs/managed-prometheus/exporters/node_exporter

Step to accomplish the task.

- Install node exporters on kubernets cluster gmp-public namespace
- Enable manager promethoues service on kubernetes cluster and manage collector on monitoring section
- Create notification chennel service like **GOOGLE CHAT** under gcp cloud alert notification chennel. For this you need to give google chat space id here. You can get this from your google chat workspace. In order to add Google Chat as a notification channel, you must first add the **Google Cloud Monitoring** App to the chat space. You can add the app directly to a space by typing @Google Cloud Monitoring. I have checked google cannot install google cloud monitoring app by just typing on google chat work space console. You need to click on + there and search you google cloud monitoring app in search bar and then click on it to make it installable. After successfully installing the app. it would give you space-id that you can use in gcp alerts under notification chennel. And use this notification chennel in you alerts. GCP will send the alert notification to specfic assign destination..
- You can also create dashboard for monitoring
