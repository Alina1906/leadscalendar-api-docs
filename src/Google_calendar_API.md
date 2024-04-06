# 1. Google Calendar API

This section provides comprehensive API documentation for integrating LeadsCalendar with Google Calendar API.

**Integration Purpose:** Create events on Google Calendar after user confirmation and successful payment processing within LeadsCalendar.

**Authentication:**

LeadsCalendar requires user authorization to access their Google Calendar data and create events. Here's a simplified overview:

1. **Google OAuth 2.0:** Implement Google OAuth 2.0 flow to obtain user consent for accessing their Google Calendar. This involves:
    * Registering your LeadsCalendar application with Google Cloud Platform (GCP) and creating OAuth credentials.
    * Redirecting users to a Google authorization page where they grant access.
    * Upon consent, Google returns an authorization code that LeadsCalendar can exchange for an access token.
    * Store the access token securely to make authenticated requests to the Google Calendar API.

**API Calls:**

* **Events.insert:** This method creates a new event on the authorized user's Google Calendar. You'll need to provide details like event summary, description, date/time, and optionally attendees.

**Code Sample (using Python and Google APIs Client Library):**

```python
from googleapiclient.discovery import build

def create_google_calendar_event(event_data, access_token):
  service = build('calendar', 'v3', credentials=access_token)

  event = service.events().insert(calendarId='primary', body=event_data).execute()

  return event
```

**Parameters:**

* **summary (string):** Required. Brief summary of the event.
* **description (string):** Optional. Detailed description of the event.
* **start.dateTime (string):** Required. The datetime (as RFC 3339) of the event's start.
* **end.dateTime (string):** Required. The datetime (as RFC 3339) of the event's end.
* **attendees (list of objects):** Optional. List of attendees with email addresses.

**Error Handling:**

Google Calendar API uses standard HTTP status codes for error handling. Here are some common codes and recommendations:

| Code | Meaning | Recommendation |
|---|---|---|
| 400 (Bad Request) | The request is invalid due to missing or malformed parameters. |  - Check the request body for missing or invalid fields.  - Verify parameter values according to API documentation. |
| 401 (Unauthorized) | Access token is invalid, expired, or revoked. | - Refresh the access token if expired.  - Prompt the user to re-authorize access if the token is invalid. |
| 403 (Forbidden) | User doesn't have permission to create events on the specified calendar. | - Verify the user's permissions on the target calendar. |
| 404 (Not Found) | The requested calendar resource (e.g., specific calendar ID) doesn't exist. | - Double-check the calendar ID used in the request. |
| 429 (Too Many Requests) | Rate limit exceeded for API calls. | - Implement exponential backoff to retry the request after a delay. |

A code snippet (Python) demonstrating how to handle errors based on status code and parse the error message:

```python
def create_google_calendar_event(event_data, access_token):
  service = build('calendar', 'v3', credentials=access_token)

  try:
    event = service.events().insert(calendarId='primary', body=event_data).execute()
    return event
  except HTTPError as error:
    # Check the status code
    if error.resp.status in (400, 401, 403):
      # Parse the error message for details
      error_data = json.loads(error.resp.content)
      error_message = error_data.get('error', {}).get('message')
      print(f"Error creating event: {error_message}")
      # Handle the error based on the specific message (e.g., refresh token, inform user)
    else:
      raise  # Re-raise unexpected errors
```

**Important Notes:**

* Refer to Google Calendar API documentation for detailed information on available parameters and authentication flow: [https://developers.google.com/calendar/api/v3/reference](https://developers.google.com/calendar/api/v3/reference)

