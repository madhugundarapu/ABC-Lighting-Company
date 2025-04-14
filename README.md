# ABC-Lighting-Company


## 1. Overview and Requirements

My goal is to build a conversational AI interface that:
- Uses fabricated data (no live internet queries) to respond to questions about the company and three products.
- Offers product photos (images stored as URLs or documents) when requested.
- Maintains a conversation flow by asking, “Is there anything else I can help you with?” after each response and, when the user says “no,” collecting their contact info.
- Supports a parent dataset (company-level information) and three child datasets (each describing one product) along with associated assets.

To achieve this, we will combine:
AWS (for serverless back end, data storage, file hosting, and possibly admin panel hosting)
Postman (to simulate/test API calls and monitor the conversation flows)
ChatGPT (as the conversational engine integrated into your API)


## 2. System Architecture

### AWS Services Selection
-  AWS Lambda: Host and execute the API logic for conversation management. Lambda functions can access your stored datasets and business logic.
-  API Gateway: Expose REST API endpoints for your bot, allowing Postman (or any client) to send queries to your backend.
-  Amazon DynamoDB: Store your data in a NoSQL format:
- Parent Dataset Table**: Contains company information such as locations, business hours, and generic product details.
- Child Dataset Table: Contains detailed information for each product.
- Amazon S3: Store the product images (or other assets) that the bot will share when requested.
- AWS Amplify or a Custom Admin Dashboard (e.g., using a React app hosted on S3/CloudFront): Allow administrators to update datasets and images dynamically.

### Data Flow and Interaction Diagram
1. User/Client (via Postman or Front-end Interface)  
   sends a REST API call to  
2. API Gateway 
   routes the request to  
3. AWS Lambda Function (which implements conversation flow)  
   queries  
4. DynamoDB Tables (parent and child datasets) and retrieves asset URLs from  
5. Amazon S3
   then returns a conversational response through the API Gateway back to the client.  
6. Admin Panel  
   hosted via AWS Amplify or a custom dashboard allows dynamic updates to the DynamoDB tables and S3 assets.

---

## 3. Data Modeling and Datasets

### Parent Dataset (Company Information)
This dataset holds general information about ABC Lighting Company. Example JSON structure:

```json
{
  "companyName": "ABC Lighting Company",
  "description": "We are a customer service representative dedicated to providing the best solar powered lighting solutions.",
  "locations": [
    {"city": "Hyderabad", "address": "123 5th Ave, Hyd"},
    {"city": "Bangalore", "address": "456 Sunset Blvd, Bangalore"}
  ],
  "businessHours": {
    "Monday-Friday": "9:00 AM - 6:00 PM",
    "Saturday": "10:00 AM - 4:00 PM",
    "Sunday": "Closed"
  },
  "generalProducts": "We offer a range of solar-powered lighting products including street lights, driveway lights, and outside wall lights."
}
```

### Child Datasets (Product Specific Information)
Each product will have its own dataset. Here are three datasets:

#### 3.1 Solar Powered Street Light
```json
{
  "productID": "SPST-001",
  "name": "Solar Powered Street Light",
  "specs": {
    "height": "12 ft",
    "weight": "35 lbs",
    "power": "Solar panel 100W; battery backup 12V",
    "operationTime": "8-10 hours per night",
    "illumination": "800 lumens"
  },
  "description": "A robust street light designed to provide bright, energy-saving illumination for city roads and pedestrian areas.",
  "assets": {
    "imageUrl": "https://example.com/images/solar_street_light.jpg"
  }
}
```

#### 3.2 Solar Powered Driveway Light
*You can fabricate realistic data based on an actual datasheet structure:*
```json
{
  "productID": "SPDL-002",
  "name": "Solar Powered Driveway Light",
  "specs": {
    "height": "6 ft",
    "width": "2 ft",
    "power": "Solar panel 50W; battery backup 12V",
    "operationTime": "6-8 hours per night",
    "illumination": "600 lumens"
  },
  "description": "Ideal for driveways and private drive areas with a focus on energy efficiency and durability.",
  "assets": {
    "imageUrl": "https://example.com/images/solar_driveway_light.jpg"
  }
}
```

#### 3.3 Solar Powered Outside Wall Light
```json
{
  "productID": "SPWL-003",
  "name": "Solar Powered Outside Wall Light",
  "specs": {
    "height": "4 ft",
    "weight": "15 lbs",
    "power": "Solar panel 30W; battery backup 12V",
    "operationTime": "5-7 hours per night",
    "illumination": "400 lumens"
  },
  "description": "Perfectly suited for residential or commercial buildings’ exterior walls, combining aesthetics with functionality.",
  "assets": {
    "imageUrl": "https://example.com/images/solar_wall_light.jpg"
  }
}
```

*Note:* Use placeholder URLs for images (or if possible, host real sample images on your S3 bucket). These URLs simulate access to the images when the bot needs to share them with users.

---

## 4. Conversation Flow with ChatGPT

The conversational flow is designed to:
- Answer any query using data from the parent dataset or delegate to child dataset details if specific product questions are asked.
- At the end of each answer, append:  
  **"Is there anything else I can help you with?"**
- If the user replies “no”, then initiate a follow-up to collect contact information.

### Flow Example:
1. Greeting/Initial Query:  
   Bot: “Welcome to ABC Lighting Company customer service. We offer solar powered street lights, driveway lights, and wall lights. Our locations are in New York and Los Angeles; our business hours are Monday-Friday from 9:00 AM to 6:00 PM. How can I help you today?”  
   *(Append: “Is there anything else I can help you with?”)*

2. **Product Inquiry:**  
   User: “Tell me more about the solar powered street light.”  
   Bot (lookup child dataset SPST-001): “Our Solar Powered Street Light is a 12 ft tall unit weighing 35 lbs. It features a 100W solar panel and 12V battery backup that delivers approximately 8-10 hours of illumination per night with an output of 800 lumens. Here’s an image of the product: [show image](https://example.com/images/solar_street_light.jpg). Is there anything else I can help you with?”

3. **Exit Conversation:**  
   User: “No.”  
   Bot: “Please provide your contact info (name, email, and/or phone) so that we can follow up.”

### Implementing with ChatGPT
Your ChatGPT integration will be configured with prompts that include instructions for:
- Leveraging the datasets.
- Appending the “anything else” query after every response.
- Checking for exit phrases (like “no”) to trigger a request for contact details.
- When the user requests an image, look up the `assets.imageUrl` from the corresponding dataset and include the link (or the actual image if integrated with an image display mechanism).

---

## 5. Integration Using AWS

### 5.1 Backend Setup with AWS Lambda and API Gateway
- **Create a Lambda Function**  
  - Write the business logic in your preferred language (Node.js, Python, etc.).  
  - The function will parse incoming POST requests, check the user query, and determine if the query is general (company info) or specific (product details).  
  - The function should call the appropriate logic:
    - **Parent Info Query:** Return general info from the parent dataset.
    - **Child Product Query:** Look up and return the respective child dataset details.
    - **Conversation Termination:** When receiving a “no” answer, trigger a prompt asking for contact details.
    
- **Configure API Gateway**  
  - Set up endpoints (e.g., `/chat`) that accept JSON requests.
  - Integrate with your Lambda backend.
  - Add authentication or CORS settings as needed.
  
*Sample pseudo-code snippet for Lambda (Python):*

```python
import json

# Sample in-memory datasets (replace with DynamoDB retrieval in production)
company_info = {
    "companyName": "ABC Lighting Company",
    "locations": ["Hyderabad: 123 5th Ave, Hyd", "Bangalore: 456 Sunset Blvd, Bangalore"],
    "businessHours": {
        "Monday-Friday": "9:00 AM - 6:00 PM",
        "Saturday": "10:00 AM - 4:00 PM",
        "Sunday": "Closed"
    },
    "generalProducts": "We offer solar powered street lights, driveway lights, and outside wall lights."
}

products = {
    "street_light": {
        "name": "Solar Powered Street Light",
        "specs": "12 ft tall, 35 lbs, 100W solar panel, 12V battery, 8-10 hours of operation, 800 lumens",
        "asset": "https://example.com/images/solar_street_light.jpg"
    },
    "driveway_light": {
        "name": "Solar Powered Driveway Light",
        "specs": "6 ft tall, durable, 50W solar panel, 12V battery, 6-8 hours of operation, 600 lumens",
        "asset": "https://example.com/images/solar_driveway_light.jpg"
    },
    "wall_light": {
        "name": "Solar Powered Outside Wall Light",
        "specs": "4 ft tall, 15 lbs, 30W solar panel, 12V battery, 5-7 hours of operation, 400 lumens",
        "asset": "https://example.com/images/solar_wall_light.jpg"
    }
}

def lambda_handler(event, context):
    body = json.loads(event.get("body", "{}"))
    user_input = body.get("message", "").lower()
    
    # Basic routing logic
    if "street" in user_input:
        response_text = (
            f"{products['street_light']['name']}: {products['street_light']['specs']}. "
            f"Here is the image: {products['street_light']['asset']}."
        )
    elif "driveway" in user_input:
        response_text = (
            f"{products['driveway_light']['name']}: {products['driveway_light']['specs']}. "
            f"Image: {products['driveway_light']['asset']}."
        )
    elif "wall" in user_input:
        response_text = (
            f"{products['wall_light']['name']}: {products['wall_light']['specs']}. "
            f"Image: {products['wall_light']['asset']}."
        )
    elif user_input.strip() == "no":
        response_text = "Please provide your contact info (name, email, and/or phone) so that we can follow up."
    else:
        response_text = (
            f"Welcome to {company_info['companyName']}! We have locations in {', '.join(company_info['locations'])} and "
            f"operational hours {company_info['businessHours']['Monday-Friday']}. "
            "We offer various solar powered lighting solutions. How can I help you today?"
        )
    
    # Append conversation continuance if not ending
    if user_input.strip() != "no" and "contact" not in response_text.lower():
        response_text += " Is there anything else I can help you with?"
    
    return {
        'statusCode': 200,
        'body': json.dumps({"response": response_text})
    }
```

*Note:* In production, replace the in-memory data with calls to DynamoDB to fetch the parent/child dataset entries.

### 5.2 Data Storage with DynamoDB and Assets in S3
- **DynamoDB Tables:**  
  - One table for the parent dataset (company info).  
  - One table for product details with a partition key (e.g., `productID`).  
- **S3 Bucket:**  
  - Create a bucket to host your images.  
  - Use meaningful file names matching your product datasets.  
- **Admin Panel Integration:**  
  - Build a simple web application (using React, Angular, or Vue) that accesses AWS Amplify or calls API Gateway endpoints to update records.
  - Use AWS Cognito for secure admin user authentication if needed.

---

## 6. Testing with Postman

Use Postman to simulate conversations with your API.

### 6.1 Creating API Requests in Postman
- **Endpoint URL:** Use the API Gateway endpoint (e.g., `https://<api_id>.execute-api.<region>.amazonaws.com/prod/chat`).
- **Method:** POST
- **Headers:**  
  - Content-Type: application/json
- **Request Body Example:**

  ```json
  {
    "message": "Tell me about your solar powered driveway light."
  }
  ```
  
- **Expected Response:**  
  The API should return a JSON response with details, such as:
  
  ```json
  {
    "response": "Solar Powered Driveway Light: Solar Powered Driveway Light: 6 ft tall, durable, 50W solar panel, 12V battery, 6-8 hours of operation, 600 lumens. Image: https://example.com/images/solar_driveway_light.jpg. Is there anything else I can help you with?"
  }
  ```

### 6.2 Testing Conversation Flow
- Chain a series of requests in Postman (using Postman’s Collections or automated scripts) to simulate:
  - An initial inquiry (company info).
  - A product inquiry (street, driveway, or wall light).
  - A “no” response to end the conversation and trigger contact info collection.

---

## 7. Building the Customizable Admin Panel

### 7.1 Objectives for the Admin Panel:
- **Dataset Management:**  
  Allow admin users to update company info, product details, and business hours.
- **Assets Management:**  
  Provide file upload functionality for product images.
- **User Interface:**  
  A clean, web-based interface that interacts with your backend API.

### 7.2 Possible Implementation Approaches:
- **AWS Amplify:**  
  Use Amplify’s tools to quickly create a full-stack application with authentication (Cognito), API integrations, and file storage.
- **Custom React Application:**  
  - Hosted in an S3 bucket (static website hosting) or via AWS Amplify.
  - Provide forms to edit JSON data stored in DynamoDB via API Gateway or directly using AWS SDKs.
  - For image uploads, integrate an S3 file uploader.
  
*High-Level Steps:*
1. **Set Up the Project:**  
   Initialize a React (or your framework of choice) project.
2. **Configure Amplify or AWS SDK:**  
   Connect the app to your backend services (API Gateway for data updates, S3 for image hosting, etc.).
3. **Create Admin Components:**  
   - **Form for Company Info:** Update text fields for locations, hours, etc.
   - **Product Management:** List existing products and provide options to update specs, descriptions, and asset URLs.
   - **Image Upload Component:** Allow admins to upload new images (set permissions on your S3 bucket accordingly).
4. **Authentication:**  
   Implement an authentication mechanism (AWS Cognito or a similar service) to restrict access to the admin panel.
5. **Deployment:**  
   Deploy your admin panel to an AWS service (S3 with CloudFront for static hosting, Amplify Hosting, etc.).

---

## 8. Final Remarks

By integrating these components:
- **Conversational AI:** Powered by ChatGPT and supported by AWS Lambda and API Gateway.
- **Data Management:** Handled through DynamoDB for datasets and S3 for product images.
- **Testing and API Simulation:** Conducted via Postman.
- **Dynamic Administration:** Enabled through a customizable admin panel (via AWS Amplify or a custom React app).
