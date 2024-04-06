# 2. PayPal REST API

This section provides comprehensive API documentation for integrating LeadsCalendar with PayPal REST API.

**Integration Purpose:** Process secure payments of 1 USD (or equivalent in cryptocurrency) through PayPal upon event creation in LeadsCalendar.

**Authentication:**

* Obtain PayPal REST API credentials from your PayPal Developer account. These include Client ID and Secret.

**API Calls:**

* **Create Payment (POST):** Initiates a payment using the `/payments` endpoint. You'll need to specify payment details in the request body.

**Parameters:**

* **intent (string) - Required:** Set to `sale` for a one-time payment.
* **payer (object) - Required:** Defines the payment source.
    * **payment_method (string):** Set to `paypal` for payments using user's PayPal account.
* **transactions (list of objects) - Required:** Specifies the transaction details.
    * **amount (object):**
        * **total (number):** Required. Total amount to be paid.
        * **currency (string):** Required. Currency code (e.g., USD).
    * **description (string):** Required. Brief description of the payment purpose (e.g., "LeadsCalendar Event Fee").

**Code Sample (using Python and requests library):**

```python
import requests

def create_paypal_payment(access_token, amount, description):
  headers = {"Authorization": f"Bearer {access_token}"}
  data = {
      "intent": "sale",
      "payer": {
          "payment_method": "paypal"
      },
      "transactions": [
          {
              "amount": {
                  "total": amount,
                  "currency": "USD"
              },
              "description": description
          }
      ]
  }

  response = requests.post("https://api.paypal.com/v1/payments/payment", headers=headers, json=data)
  response.raise_for_status()  # Raise exception for non-200 status codes

  return response.json()
```

**Error Handling:**

PayPal REST API uses standard HTTP status codes and provides detailed error messages in the JSON response body. Here are some common codes and recommendations:

| Code | Meaning | Recommendation |
|---|---|---|
| 400 (Bad Request) | The request is invalid due to missing or malformed parameters. | -  Parse the error message to identify the specific parameter causing the issue. - Correct the parameter value and retry the API call. |
| 401 (Unauthorized) | Invalid or expired client credentials. | - Verify your PayPal REST API credentials are correct. |
| 403 (Forbidden) | Insufficient permissions for the requested operation. | - Ensure your account has permission to create payments. |
| 404 (Not Found) | The requested resource (e.g., specific payment) doesn't exist. | - Double-check the payment ID used in the request (if applicable). |
| 429 (Too Many Requests) | Rate limit exceeded for API calls. | - Implement exponential backoff to retry the request after a delay. |

A code snippet (Python with requests library) demonstrating error handling based on status code and parsing the error response:

```python
  try:
    response = requests.post("https://api.paypal.com/v1/payments/payment", headers=headers, json=data)
    response.raise_for_status()  # Raise exception for non-200 status codes
    return response.json()
  except requests.exceptions.HTTPError as error:
    # Check the status code
    if error.response.status_code in (400, 401, 403):
      # Parse the error JSON for details
      error_data = error.response.json()
      error_code = error_data.get('error_code')
      error_message = error_data.get('message')
      print(f"Error creating payment: {error_code} - {error_message}")
      # Handle the error based on the specific code (e.g., refresh token, inform user)
    else:
      raise  # Re-raise unexpected errors
```

**Important Notes:**

* Ensure secure storage and transmission of access tokens.
* Refer to PayPal REST API documentation for a complete list of parameters and response structures: [https://developer.paypal.com/api/rest/](https://developer.paypal.com/api/rest/)
