# Auth server metrics events

The auth server emits two types of event
that are imported to redshift
and made available to
metrics queries in redash:

* [Flow events](#flow-events),
  which represent something happening
  during the course of
  a sign-in or sign-up flow.

* [Activity events](#activity-events),
  which represent significant actions
  or state changes
  at the account level.

Flow events are used
to plot charts for
sign-in and sign-up user funnels.
They enable us to
follow individual user journeys
for the length of each flow,
even when those journeys
span over multiple devices or browsers.
They also power the chart for our
"time taken for device connection" KPI.

Activity events are used
for analysing user behaviour
in a more general way.
They are behind the charts for our
"engagement ratio" and "multi-device usage" KPIs.

## Flow events

The following flow events
represent a successful step
in a sign-in or sign-up flow:

|Name|Description|
|----|-----------|
|`flow.${viewName}.begin`|A user has landed on a page that allows them to sign in/up.|
|`flow.${viewName}.engage`|A user has interacted with the the form on a page that allows them to sign in/up`.|
|`flow.${viewName}.submit`|A user has submitted the signup form on a page that allows them to sign in/up.|
|`flow.${viewName}.have-account`|A user has clicked on the 'Already have an account?' link.|
|`flow.${viewName}.create-account`|A user has clicked on the 'Create an account' link.|
|`flow.${viewName}.forgot-password`|A user has clicked on the 'Forgot password?' link.|
|`flow.${action}.attempt`|The content server has sent a sign-in/up request to the auth server.|
|`account.login`|An existing account has been signed in to.|
|`account.created`|A new account has been created.|
|`email.confirmation.sent`|A sign-in confirmation email has been sent to a user.|
|`email.verification.sent`|A sign-up verification email has been sent to a user.|
|`email.confirmation.resent`|A sign-in confirmation email has been re-sent to a user.|
|`email.verification.resent`|A sign-up verification email has been re-sent to a user.|
|`email.verify_code.clicked`|A user has clicked on the link in a confirmation/verification email.|
|`account.confirmed`|Sign-in to an existing account has been confirmed via email.|
|`account.reminder`|A new account has been verified via a reminder email.|
|`account.verified`|A new account has been verified via email.|
|`account.keyfetch`|Sync encryption keys have been fetched.|
|`account.signed`|A certificate has been signed.|
|`account.login.confirmedUnblockCode`|A user has successfully unblocked their account.|
|`account.login.sentUnblockCode`|A sign-in unblock email has been sent to the user.|
|`password.forgot.send_code.start`|A user has initiated the password reset flow.|
|`password.forgot.send_code.completed`|A password reset email has been sent to the user.|
|`password.forgot.resend_code.start`|A user has requested that the password reset email be re-sent.|
|`password.forgot.resend_code.completed`|A password reset email has been re-sent to the user.|
|`password.forgot.verify_code.start`|A user has clicked on the link in a password reset email.|
|`password.forgot.verify_code.completed`|A password reset has been successfully completed on the server.|
|`flow.completed`|A user has successfully completed a sign-in or sign-up flow.|

The following flow events
represent error conditions,
which may or may not be terminal
to a flow:

|Name|Description|
|----|-----------|
|`customs.blocked`|A request was blocked by the customs server.|

In redshift,
these events are stored
in two tables:

* `flow_metadata`,
  containing all of the data
  relating to a flow
  as a single entity.

* `flow_events`,
  containing data
  for the individual events
  within each flow.

The `flow_metadata` table
contains the following fields:

|Name|Description|
|----|-----------|
|`flow_id`|The flow identifier.|
|`begin_time`|The time at which the `flow.begin` event occurred.|
|`duration`|The length of time from the `flow.begin` event until the last event of the flow.|
|`completed`|Boolean indicating whether the flow was successfully completed.|
|`new_account`|Boolean indicating whether the flow was a sign-up.|
|`ua_browser`|The user's web browser.|
|`ua_version`|The user's browser version.|
|`ua_os`|The user's operating system.|
|`context`|FxA auth broker context.|
|`entrypoint`|The entrypoint for the flow.|
|`migration`|Sync migration.|
|`service`|The service identifier. For Sync it may be empty or `sync`. For OAuth reliers it is their hex id.|
|`utm_campaign`|Marketing campaign identifier. Not stored if the `DNT` request header was `1`.|
|`utm_content`|Marketing campaign content identifier. Not stored if the `DNT` request header was `1`.|
|`utm_medium`|Marketing campaign medium. Not stored if the `DNT` request header was `1`.|
|`utm_source`|Marketing campaign source. Not stored if the `DNT` request header was `1`.|
|`utm_term`|Marketing campaign search term. Not stored if the `DNT` request header was `1`.|

The `flow_events` table
contains the following fields:

|Name|Description|
|----|-----------|
|`timestamp`|The time at which the event occurred.|
|`flow_time`|The time since the beginning of the flow.|
|`flow_id`|The flow identifier.|
|`type`|The event name.|

## Activity events

The following activity events
are emitted:

|Name|Description|
|----|-----------|
|`account.created`|A new account has been created.|
|`account.login`|An existing account has been signed in to.|
|`account.verified`|A new account has been verified via email.|
|`account.confirmed`|Sign-in to an existing account has been confirmed via email.|
|`account.keyfetch`|Sync encryption keys have been fetched.|
|`account.signed`|A certificate has been signed.|
|`account.reset`|An account has been reset.|
|`account.deleted`|An account has been deleted.|
|`device.created`|A device record has been created for a Sync account.|
|`device.updated`|Device record is updated on a Sync account.|
|`device.deleted`|Device record has been deleted from a Sync account.|

In redshift,
these events are stored
in the `activity_events` table
with the following fields:

|Name|Description|
|----|-----------|
|`timestamp`|The time at which the event occurred.|
|`type`|The name of the event.|
|`uid`|The user id, HMACed so as not to leak PII.|
|`device_id`|Optional. The id of the device record.|
|`service`|Optional. The id of the requesting service. For Sync this may be `'sync'` or the empty string.|
|`ua_browser`|The user's web browser.|
|`ua_version`|The user's browser version.|
|`ua_os`|The user's operating system.|

Two further tables,
summarising device usage,
are populated
based on the activity event data.

The table `daily_activity_per_device`
contains the following fields:

|Name|Description|
|----|-----------|
|`day`|The date of the activity.|
|`uid`|The HMACed user id.|
|`device_id`|The id of the active device.|
|`service`|The id of the requesting service.|
|`ua_browser`|The user's web browser.|
|`ua_version`|The user's browser version.|
|`ua_os`|The user's operating system.|

The table `daily_multi_device_users`
contains the following fields:

|Name|Description|
|----|-----------|
|`day`|The date of the activity.|
|`uid`|The HMACed user id.|

For this table,
a multi-device user is defined as
somebody who was also active
on a different device
in the preceding five days.

## Significant changes

### Train 75

* The correct `service` parameter
  was passed to `/certificate/sign`
  for OAuth reliers,
  stopping those requests from
  being identified as originating from
  the content server.

### Train 74

* Flow event data validation
  was implemented.

* The `${viewName}` part of
  `flow.${viewName}.begin`,
  `flow.${viewName}.engage` and
  `flow.${viewName}.submit`
  was fixed,
  rendering earlier data incorrect.

* The `flow.have-account` event
  was changed to `flow.${viewName}.have-account`.

* The `flow.${viewName}.create-account` event
  was implemented.

* The `flow.${viewName}.forgot-password` event
  was implemented.

* The `flow.${action}.attempt` event
  was implemented.

### Train 73

* Expiry time
  for metrics context data in memcached
  was increased from 30 minutes
  to 2 hours.

* The `flow.complete` event
  was implemented.

### Train 72

* A change to the memcached key
  used when stashing metrics context data
  introduced a 30-minute partial blip
  in flow event data.

### Train 71

* The `flow.begin` event
  was changed to `flow.${viewName}.begin`.

* Timestamps were fixed
  on the begin, engage and submit events.

* Metrics context data was added
  to the `flow.${viewName}.begin` event.

* Erroneous `"none"` values were removed
  from empty metrics context properties
  in the content server.

* Expiry time for flow ids
  was increased from 30 minutes
  to two hours.

