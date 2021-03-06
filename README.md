# Slack

## Synopsis

This neuron allows you to 
- POST a message to slack (channel or IM).
- READ a number of messages from a channel.
- list USERS of a slack's team 

## Installation
```bash
kalliope install --git-url https://github.com/kalliope-project/kalliope_neuron_slack.git
```

## Specification

The Slack Neuron has multiple available actions : POST, READ, USERS.

Each of them requires specific options, return values and synapses example : 

#### POST 
##### Options


| parameter   | required | type   | default | choices           | comment                              |
|-------------|----------|--------|---------|-------------------|--------------------------------------|
| action      | YES      | String | None    | POST, READ, USERS | Defines the action type              |
| slack_token | YES      | String | None    |                   | The slack token                      |
| message     | YES      | String | None    |                   | The text to post                     |
| channel     | YES      | String | None    |                   | The channel name to post the message |


##### Return Values

| Name    | Description                                | Type   | sample      |
|---------|--------------------------------------------|--------|-------------|
| action  | the action USED                            | String | POST        |
| message | The text posted on Slack                   | String | Hi Kalliope |
| channel | The channel where the text has been posted | String | General     |

##### Synapses example

``` yml
- name: "post-slack"
  neurons:
    - slack:
        action: "POST"
        slack_token: "MY_SECRET_TOKEN"
        channel: "#general"
        message: "{{ message }}"
  signals:
    - order: "post on slack {{ message }}"
```


#### READ
##### Options


| parameter   | required | type   | default | choices           | comment                              |
|-------------|----------|--------|---------|-------------------|--------------------------------------|
| action      | YES      | String | None    | POST, READ, USERS | Defines the action type              |
| slack_token | YES      | String | None    |                   | The slack token                      |
| nb_messages | No       | int    | 10      |                   | number of messages to read           |
| channel     | YES      | String | None    |                   | The channel name to post the message |


##### Return Values

| Name     | Description                                | Type   | sample                                                                    |
|----------|--------------------------------------------|--------|---------------------------------------------------------------------------|
| action   | the action USED                            | String | READ                                                                      |
| messages | The list of dict username:message          | List   | ({"monf":"Hi There!"}, {"Kalliope":"Hi @monf"}, {"monf":"How are you ?"}) |
| channel  | The channel where the text has been read   | String | General                                                                   |

##### Synapses example

``` yml
- name: "read-slack"
  neurons:
    - slack:
        action: READ
        slack_token: "MY_SECRET_TOKEN"
        nb_messages: 3
        channel: "{{ channel }}"
  signals:
    - order: "Read Slack messages from {{ channel }}"
```

An example using the Jinja2 templates files:
``` yml
- name: "read-slack"
  neurons:
    - slack:
      slack_token: "MY_SECRET_SLACK_TOKEN"
      action: "READ"
      nb_messages: 3
      channel: "general"
      file_template: neurons/slack/slack_template.j2
 signals:
    - order: "Read Slack messages"
```

The template defined in the slack_template.j2
```jinja2
{% for message in messages %}
    {% for key, value in message.iteritems() %}
        {{ key }} said {{ value }}
    {% endfor %}
{% endfor %}

```

#### USERS
##### Options

| parameter      | required | type   | default | choices           | comment                              |
|----------------|----------|--------|---------|-------------------|--------------------------------------|
| action         | YES      | String | None    | POST, READ, USERS | Defines the action type              |
| only_connected | NO       | bool   | TRUE    |                   | Only connected users                 |


##### Return Values

| Name           | Description                                           | Type   | sample                   |
|----------------|-------------------------------------------------------|--------|--------------------------|
| action         | the action USED                                       | String | USERS                    |
| members        | The list of [users](https://api.slack.com/types/user) | List   |                          |
| only_connected | the value of only_connected parameters                | bool   | True                     |

##### Synapses example

``` yml
- name: "users-slack"
  neurons:
    - slack:
        action: USERS
        slack_token: "MY_SECRET_TOKEN"
        file_template: neurons/slack/slack_user_template.j2
  signals:
    - order: "List connected users to Slack"
```

The template defined in the slack_user_template.j2
``` yml
{% if members|length == 0 %}
    No users connected
{% else %}
    {{ members|length }} members connected.
    {% for member in members %}
        {{ member.name }}
    {% endfor %}
{% endif %}
```

##### 

## Notes

In order to be able to post on Slack, you need to get a slack token (currently only *Tokens for testing* are supported). 

### How to get your Slack token (associated with one team)

1. Sign in your [Slack account](https://slack.com/signin)
2. Create a [test token](https://api.slack.com/docs/oauth-test-tokens)
6. Get your token (Keep it secret !)
7. Post your first message with this neuron !

### Send a IM message
/!\ __TODO__ automatise this step 

You need to get the IM channel's ID.

 1. Go to [Slack user api test page](https://api.slack.com/methods/users.list/test) 
 2. Select the token for the team and click to the test method button
 3. Get the ID for the user  
 4. Go to [Slack im list api test page](https://api.slack.com/methods/im.list/test) 
 5. Select the token for the team and click to the test method button
 6. Get the IM channel's ID associated for this user (property user)
