# 3. Binance Pay API

This section provides comprehensive API documentation for integrating LeadsCalendar with Binance Pay API.

**Integration Purpose:** Enable users to pay the 1 USD equivalent in cryptocurrency using Binance Pay within LeadsCalendar.

**Authentication:**

* Acquire Binance API credentials (API Key and Secret) from your Binance Developer Platform account.

**API Calls:**

* **Create Order (POST):** Creates a payment order for the user using the `/orders` endpoint. Details are provided in the request body.

**Parameters:**

* **transactionType (string) - Required:** Set to `PAY` for a payment transaction.
* **fiat (string) - Required:** Set to `USD` for the desired settlement currency.
* **price (number) - Required:** Specify the amount in USD (e.g., 1.00).
* **callbackUrl (string) - Required:** Define a URL in your LeadsCalendar application to receive Binance Pay confirmation.


**Error Handling:**

Binance Pay API uses standard HTTP status codes along with JSON error responses that provide details about the encountered problem. Here's a reference table listing potential error codes, their meanings, and recommendations for handling them:

| Code | Meaning | Recommendation |
|---|---|---|
| 400 (Bad Request) | The request is invalid due to missing or malformed parameters. |  1. Parse the error message body for details. It will likely contain an `error_code` and a specific error message describing the problem. <br> 2. Inspect your request parameters and identify the one causing the error based on the error code and message. <br> 3. Correct the parameter value according to the Binance Pay API documentation. <br> 4. Retry the API call with the corrected parameter. |
| 401 (Unauthorized) | Invalid or expired API credentials. | 1. Verify that your API Key and Secret used in the request are correct. <br> 2. Check if the credentials have expired. If so, refresh them using the Binance API renewal process. <br> 3. Retry the API call with the valid credentials. |
| 403 (Forbidden) | Insufficient permissions for the requested operation. | 1. Ensure your Binance API account has the necessary permissions to create payment orders (e.g., "ORDER_CREATE").  <br> 2. If unsure about permissions, contact Binance support for clarification. |
| 429 (Too Many Requests) | Rate limit exceeded for API calls. | 1. Implement an exponential backoff strategy. This involves retrying the request after a waiting period that doubles with each attempt. <br> 2. This approach helps avoid overwhelming the Binance Pay API with requests. |
| 404 (Not Found) | (Less likely for order creation) Resource not found (e.g., specific order ID). |  1. Double-check any IDs used in the request (if applicable).<br> 2. Verify you're using the correct API endpoint. |
| 500 (Internal Server Error) | An issue on Binance Pay's side. | 1. Retry the API call after a reasonable delay (e.g., a few minutes) as the server issue might be temporary.  <br> 2. If the error persists, consider contacting Binance support for assistance. |

**Code Snippet (Python):**

Here's an example code snippet (using the `requests` library) demonstrating error handling based on status code and parsing the error response for Binance Pay API:

```python
import requests

def create_binance_pay_order(api_key, secret, price, callback_url):
  headers = {"X-MBX-APIKEY": api_key}
  data = {
      "transactionType": "PAY",
      "fiat": "USD",
      "price": price,
      "callbackUrl": callback_url
  }

  url = "https://api.binance.com/orders"

  try:
    response = requests.post(url, headers=headers, json=data)
    response.raise_for_status()  # Raise exception for non-200 status codes
    return response.json()
  except requests.exceptions.HTTPError as error:
    # Check the status code
    if error.response.status_code in (400, 401, 403):
      # Parse the error JSON for details
      error_data = error.response.json()
      error_code = error_data.get('code')
      error_msg = error_data.get('msg')
      print(f"Error creating order: {error_code} - {error_msg}")
      # Handle the error based on the specific code (e.g., refresh credentials, inform user)
    elif error.response.status_code == 429:
      print(f"Rate limit exceeded. Retrying after {delay} seconds...")
      # Implement exponential backoff logic here
      # Retry the request after a calculated delay
    else:
      raise  # Re-raise unexpected errors

# Example usage with error handling
try:
  order_response = create_binance_pay_order(your_api_key, your_secret, 1.00, "https://your-callback-url")
  # Process successful response
except Exception as e:
  print(f"An error occurred: {e}")
  # Handle any errors during order creation
```

**Remember:**

* Replace `your_api_key`, `your_secret`, `1.00`, and `"https://your-callback-url"` with your actual values.
