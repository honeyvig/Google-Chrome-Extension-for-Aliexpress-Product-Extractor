# Google-Chrome-Extension-for-Aliexpress-Product-Extractor

The goal is to create a Chrome extension that:
1. Can extract product information from an open AliExpress product page
2. Show it to customer to edit and confirm
3.  send the details to a REST API.
2. Obtain the response from API, show it to customer
3. If customer confirm, send the new data to a different API endpoint
4. If not, start from (2)

The extension must support Auth0 authentication for secure API access.
-------------
To create a Chrome extension that meets your requirements, the following steps will be involved:

    Extract Product Information from AliExpress Product Page: This involves parsing the DOM of the AliExpress product page to get details like product name, price, description, and images.

    Show Extracted Information to the Customer: Display the extracted product details in an editable format (a popup in the extension) for the customer to confirm and edit.

    Send Data to REST API: Send the edited or unedited data to a REST API for processing.

    Get Response from API and Display It: Display the response from the REST API to the customer for confirmation.

    Send Final Data to Another API: If the customer confirms the data, send the finalized product data to another API endpoint.

    Auth0 Authentication: Ensure that API requests are secured using Auth0 for authentication and authorization.

Chrome Extension Breakdown

Let's break down the code structure for each part:
1. Manifest File (manifest.json)

This file is the configuration of the extension, indicating the permissions and scripts involved.

{
  "manifest_version": 3,
  "name": "AliExpress Product Extractor",
  "description": "Extract and edit product data from AliExpress.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "identity"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "assets/icon.png",
      "48": "assets/icon.png",
      "128": "assets/icon.png"
    }
  },
  "content_scripts": [
    {
      "matches": ["https://www.aliexpress.com/item/*"],
      "js": ["content.js"]
    }
  ],
  "options_page": "options.html",
  "icons": {
    "16": "assets/icon.png",
    "48": "assets/icon.png",
    "128": "assets/icon.png"
  }
}

2. Background Script (background.js)

The background script is used for authentication and handling communication between different parts of the extension.

// background.js
chrome.runtime.onInstalled.addListener(() => {
  console.log("AliExpress Product Extractor Extension Installed");
});

// Use chrome identity API for OAuth authentication with Auth0
chrome.identity.getAuthToken({ interactive: true }, function (token) {
  if (chrome.runtime.lastError) {
    console.error(chrome.runtime.lastError);
  } else {
    console.log("Auth Token:", token);
  }
});

3. Content Script (content.js)

This script will run on the AliExpress product page and extract product details like title, price, and description.

// content.js
function extractProductData() {
  const title = document.querySelector('h1.product-title')?.innerText;
  const price = document.querySelector('.product-price-value')?.innerText;
  const description = document.querySelector('.product-description')?.innerText;
  const imageUrl = document.querySelector('.magnifier-image img')?.src;

  return {
    title,
    price,
    description,
    imageUrl
  };
}

// Send extracted data to popup
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "extractProductInfo") {
    const productData = extractProductData();
    sendResponse(productData);
  }
});

4. Popup (popup.html)

This will display the extracted product details in an editable format for the customer to confirm or modify.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Edit Product</title>
</head>
<body>
  <h1>Edit Product Information</h1>
  <div>
    <label for="title">Title</label>
    <input type="text" id="title" />
  </div>
  <div>
    <label for="price">Price</label>
    <input type="text" id="price" />
  </div>
  <div>
    <label for="description">Description</label>
    <textarea id="description"></textarea>
  </div>
  <div>
    <label for="imageUrl">Image URL</label>
    <input type="text" id="imageUrl" />
  </div>
  <button id="submit">Submit</button>
  <button id="cancel">Cancel</button>
  
  <script src="popup.js"></script>
</body>
</html>

5. Popup Script (popup.js)

This script handles sending the extracted product data from the content script to the popup for display and allows the user to edit the information.

// popup.js

// Extract product data from the current tab
chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
  chrome.scripting.executeScript({
    target: { tabId: tabs[0].id },
    function: extractProductInfo
  }, function (result) {
    const productData = result[0].result;
    document.getElementById("title").value = productData.title;
    document.getElementById("price").value = productData.price;
    document.getElementById("description").value = productData.description;
    document.getElementById("imageUrl").value = productData.imageUrl;
  });
});

function extractProductInfo() {
  return window.extractProductData();
}

// Submit the form data to the API
document.getElementById("submit").addEventListener("click", () => {
  const productData = {
    title: document.getElementById("title").value,
    price: document.getElementById("price").value,
    description: document.getElementById("description").value,
    imageUrl: document.getElementById("imageUrl").value
  };

  // Send data to the first API endpoint
  fetch("https://your-api.com/submit-data", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${token}`  // Auth0 token
    },
    body: JSON.stringify(productData)
  })
  .then(response => response.json())
  .then(responseData => {
    console.log("API Response:", responseData);

    // Show response to customer and ask for confirmation
    if (confirm("Do you confirm this product data?")) {
      // Send final data to second API endpoint
      fetch("https://your-api.com/confirm-data", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${token}`
        },
        body: JSON.stringify(responseData)
      })
      .then(response => response.json())
      .then(finalResponse => {
        console.log("Final Response:", finalResponse);
      });
    }
  });
});

6. Auth0 Authentication

For secure API access, you will need to set up Auth0 authentication. Here’s how you can handle the authentication:

    Set up Auth0: Create an Auth0 account and configure an application. After that, you will get client_id, client_secret, and domain which you will use for authentication.

    Get Token: Use the chrome.identity API to obtain the Auth0 authentication token as shown in the background.js.

7. Styling and UI Improvements

You can add CSS to style the popup for better presentation. The HTML structure can be expanded for additional fields or customization based on the response data from the APIs.
Conclusion

This Chrome extension works by:

    Extracting product details from AliExpress.
    Displaying them in a popup for the user to edit.
    Sending the edited product information to a REST API.
    Receiving the response and confirming the data before sending it to another API endpoint.
    Securing API access using Auth0 for authentication.
