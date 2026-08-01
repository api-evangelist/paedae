---
name: paedae-run-communication
description: >-
  Create, target, publish, and stop a Gimbal proximity communication
  (notification campaign) via the Gimbal Manager REST API.
api: Gimbal REST API
base_url: https://manager.gimbal.com/api
auth: 'Authorization: Token token=<organization_server_api_key>'
operations:
  - createCommunication
  - createCommunicationNotification
  - createCommunicationGeofenceTargets
  - createCommunicationGeofenceTrigger
  - publishCommunication
  - stopCommunication
---

# Run a Gimbal Proximity Communication

Use this skill to stand up a location-triggered message campaign. All requests
are `application/json` and authenticate with
`Authorization: Token token=<organization_server_api_key>`.

## Steps

1. **Create the communication** — `POST /communications` (`createCommunication`).
2. **Attach a notification** — `POST /communications/{id}/notification`
   (`createCommunicationNotification`) with the message payload.
3. **Set geofence targets** — `POST /communications/{id}/geofence_targets`
   (`createCommunicationGeofenceTargets`) to choose where it fires.
4. **Add a geofence trigger** — `POST /communications/{id}/geofence_trigger`
   (`createCommunicationGeofenceTrigger`) for entry/exit behavior; optionally a
   time filter under it.
5. **Publish** — `POST /communications/{id}/publish` (`publishCommunication`).
6. **Stop** — `POST /communications/{id}/stop` (`stopCommunication`) when done.

## Notes

- Runtime sighting events (Arrived/Departed/Sighted) are delivered to your
  configured action URL via the Sightings Callback — see
  `asyncapi/paedae-webhooks.yml`.
- Validation errors return HTTP 422 with Gimbal error code 6001.
