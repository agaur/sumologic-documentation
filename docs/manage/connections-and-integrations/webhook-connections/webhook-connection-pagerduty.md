---
id: pagerduty
---

# Webhook Connection for PagerDuty

PagerDuty webhook connections allow you to send alert results as a PagerDuty notification. You can learn more about PagerDuty webhooks in
their [API Help](https://v2.developer.pagerduty.com/docs/webhooks-v2-overview).

[Webhook connections](set-up-webhook-connections.md) rely on HTTP endpoints that tell Sumo Logic where to send data. You can set up any number of connections. Once you set up the webhook connection you'll have the option to use it in a [Scheduled Search](schedule-searches-webhook-connections.md) or [Monitor](/docs/alerts/monitors).

PagerDuty has a [Sumo Logic Integration Guide](https://www.pagerduty.com/docs/guides/sumo-logic-integration-guide/) with these same steps.

## Create a service key for the webhook

1. In PagerDuty go to the **Services** menu and select **Service Directory**.
1. On the **Service Directory** page: 

    * If you are creating a new service for your integration, click **Add New Service**.
    * If you are adding your integration to an existing service, click the name of the service you want to add the integration to. Then click the **Integrations** tab and click the **New Integration** button.

1. In the **Integration Type** menu, select from the following based on your preference: 

    1. **Select Tool**: Search and select **Sumo Logic**. 
    1. **Use our API directly**: You may also integrate by selecting ****Events API v2**** (recommended) or **Events API v1**.

1. Enter an **Integration Name**. If you are creating a new service for your integration, in General Settings, enter a **Name** for your new service. Then, in Incident Settings, specify the **Escalation Policy**, **Notification Urgency**, and **Incident Behavior** for your new service.
1. Click the **Add Service** or **Add Integration** button to save your new integration. You will be redirected to the Integrations page for your service.
1. Copy the **Integration Key** for your new integration and keep it in a safe place for later use.

## Set up a webhook connection for PagerDuty

:::note
You need the **Manage connections** [role capability](../../users-and-roles/roles/role-capabilities.md) to create webhook connections.
:::

The URL and supported payload are different based on the version of the PagerDuty Events API you are using. Follow the steps for the relevant API version below.

### Events API V2

1. Go to **Manage Data \> Monitoring \> Connections**.
1. On the Connections page click **Add**.
1. Click **PagerDuty**.
1. In the Create Connection dialog, enter the name of the Connection.
1. (Optional) Enter a **Description** for the Connection.
1. Enter the **URL** for the endpoint: `https://events.pagerduty.com/v2/enqueue`
1. The optional input fields **Authorization Header** and **Custom Headers** do not do anything and are ignored.
1. The default **Payload** will not work with Event API V2. Change it to the following:

     ```json
     {
          "routing_key": "SERVICE KEY",
          "event_action": "trigger",
          "description": "{{Description}}",
          "client": "Sumo Logic",
          "client_url": "{{QueryURL}}",
          "payload": {
                    "summary": "summary",
                    "source": "Monitor {{Name}}",
                    "severity": "info",
                    "custom_details": {
                         "MonitorType": "{{MonitorType}}"
                    }
               }
     }
     ```

    * In the **Payload**, where it says `SERVICE KEY`, paste in the **integration key** you previously copied from PagerDuty.
    * In the **Payload** for the `description`, specify the description you want sent to PagerDuty. The above payload has specified to use the name of the alert.

1. For details on other variables that can be used as parameters within your JSON object, see [Webhook Payload Variables](set-up-webhook-connections.md).
1. Click **Save**.

### Events API V1

1. Go to **Manage Data \> Alerts \> Connections**.
1. On the Connections page click **Add**.
1. Click **PagerDuty**.
1. In the Create Connection dialog, enter the name of the Connection.
1. (Optional) Enter a **Description** for the Connection.
1. Enter the **URL** for the endpoint: `https://events.pagerduty.com/generic/2010-04-15/create_event.json`
1. The optional input fields **Authorization Header** and **Custom Headers** do not do anything and are ignored.
1. In the **Payload**, where it says `SERVICE KEY`, paste in the **integration key** you previously copied from PagerDuty.
1. In the **Payload** for the `description`, specify the description you want sent to PagerDuty.
1. For details on other variables that can be used as parameters within your JSON object, see [Webhook Payload Variables](set-up-webhook-connections.md).

     ![PagerDuty default payload.png](/img/connection-and-integration/PagerDuty-default-payload.png)

1. Click **Save**.

### Create a PagerDuty Incident Report via Webhook

Select the webhook connection in a [Scheduled Search](schedule-searches-webhook-connections.md) or [Monitor](/docs/alerts/monitors) to create a PagerDuty incident and use the following payload.

```json
{
    "service_key": "SERVICE KEY",
    "event_type": "trigger",
    "description": "SAMPLE DESCRIPTION",
    "client": "Sumo Logic",
    "client_url": "{{QueryURL}}",
    "details": {
        "name": "{{Name}}",
        "time": "{{TriggerTimeRange}}--{{TriggerTime}}",
        "num": "{{NumQueryResults}}",
        "query": "{{Query}}",
        "results": "{{ResultsJson}}"
    }
}
```