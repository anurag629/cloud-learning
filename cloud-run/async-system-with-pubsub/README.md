# Build a Resilient, Asynchronous System with Cloud Run and Pub/Sub

## Overview
Pet Theory aims to automate the process of sharing medical test results with clients efficiently. The solution involves multiple services for handling HTTP requests, sending emails, and SMS notifications, with communication managed via Pub/Sub to ensure resilience. This lab uses **Cloud Run** to deploy services and **Pub/Sub** to facilitate inter-service communication.

## Architecture
The system comprises the following components:
- **Lab Report Service**: Receives HTTP POST requests and publishes messages to Pub/Sub.
- **Email Service**: Consumes Pub/Sub messages to send emails.
- **SMS Service**: Consumes Pub/Sub messages to send SMS notifications.
- **Pub/Sub**: Provides asynchronous communication between services.
- **Cloud Run**: Deploys each service in a serverless environment.

## Objectives
- Create a Pub/Sub topic and subscription.
- Build a Cloud Run service to receive HTTP requests and publish messages to Pub/Sub.
- Build services for sending emails and SMS notifications, triggered by Pub/Sub messages.
- Test the system's resilience to failures.

## Prerequisites
- Familiarity with Google Cloud Console and shell environments.
- Completion of the previous labs in the series: "Importing Data to a Firestore Database," "Build a Serverless Web App with Firebase," and "Build a Serverless App with Cloud Run that Creates PDF Files."

## Setup and Requirements
- **Browser**: Use Chrome in Incognito mode to avoid conflicts with personal Google Cloud accounts.
- **Time**: The lab duration is one hour.
- **Credentials**: Temporary Google Cloud credentials provided by the lab.

## Instructions

### Task 1: Create a Pub/Sub Topic
1. **Create a Pub/Sub Topic**:
   ```bash
   gcloud pubsub topics create new-lab-report
   ```

2. **Enable Cloud Run**:
   ```bash
   gcloud services enable run.googleapis.com
   ```

### Task 2: Build the Lab Report Service
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/rosera/pet-theory.git
   cd pet-theory/lab05/lab-service
   ```

2. **Install Dependencies**:
   ```bash
   npm install express body-parser @google-cloud/pubsub
   ```

3. **Update `package.json`**:
   In the "scripts" section, add:
   ```json
   "start": "node index.js",
   ```

4. **Create `index.js`**:
   Add the following code to handle incoming HTTP requests and publish messages to Pub/Sub:
   ```javascript
   const {PubSub} = require('@google-cloud/pubsub');
   const pubsub = new PubSub();
   const express = require('express');
   const app = express();
   const bodyParser = require('body-parser');
   app.use(bodyParser.json());
   const port = process.env.PORT || 8080;

   app.listen(port, () => {
     console.log('Listening on port', port);
   });

   app.post('/', async (req, res) => {
     try {
       const labReport = req.body;
       await publishPubSubMessage(labReport);
       res.status(204).send();
     } catch (ex) {
       console.log(ex);
       res.status(500).send(ex);
     }
   });

   async function publishPubSubMessage(labReport) {
     const buffer = Buffer.from(JSON.stringify(labReport));
     await pubsub.topic('new-lab-report').publish(buffer);
   }
   ```

5. **Create `Dockerfile`**:
   ```Dockerfile
   FROM node:18
   WORKDIR /usr/src/app
   COPY package.json package*.json ./
   RUN npm install --only=production
   COPY . .
   CMD [ "npm", "start" ]
   ```

6. **Deploy the Service**:
   - Create a file `deploy.sh`:
     ```bash
     gcloud builds submit \
       --tag gcr.io/$GOOGLE_CLOUD_PROJECT/lab-report-service
     gcloud run deploy lab-report-service \
       --image gcr.io/$GOOGLE_CLOUD_PROJECT/lab-report-service \
       --platform managed \
       --region us-west1 \
       --allow-unauthenticated \
       --max-instances=1
     ```
   - Make it executable and run it:
     ```bash
     chmod u+x deploy.sh
     ./deploy.sh
     ```

### Task 3: Create and Deploy the Email Service
1. **Navigate to Email Service Directory**:
   ```bash
   cd ~/pet-theory/lab05/email-service
   ```

2. **Install Dependencies**:
   ```bash
   npm install express body-parser
   ```

3. **Update `package.json`**:
   Add the following to the "scripts" section:
   ```json
   "start": "node index.js",
   ```

4. **Create `index.js`**:
   Add the following code to handle Pub/Sub messages:
   ```javascript
   const express = require('express');
   const app = express();
   const bodyParser = require('body-parser');
   app.use(bodyParser.json());

   const port = process.env.PORT || 8080;
   app.listen(port, () => {
     console.log('Listening on port', port);
   });

   app.post('/', async (req, res) => {
     const labReport = decodeBase64Json(req.body.message.data);
     try {
       console.log(`Email Service: Report ${labReport.id} trying...`);
       sendEmail();
       console.log(`Email Service: Report ${labReport.id} success :-)`);
       res.status(204).send();
     } catch (ex) {
       console.log(`Email Service: Report ${labReport.id} failure: ${ex}`);
       res.status(500).send();
     }
   });

   function decodeBase64Json(data) {
     return JSON.parse(Buffer.from(data, 'base64').toString());
   }

   function sendEmail() {
     console.log('Sending email');
   }
   ```

5. **Create `Dockerfile`**:
   ```Dockerfile
   FROM node:18
   WORKDIR /usr/src/app
   COPY package.json package*.json ./
   RUN npm install --only=production
   COPY . .
   CMD [ "npm", "start" ]
   ```

6. **Deploy the Email Service**:
   - Create a `deploy.sh`:
     ```bash
     gcloud builds submit \
       --tag gcr.io/$GOOGLE_CLOUD_PROJECT/email-service
     gcloud run deploy email-service \
       --image gcr.io/$GOOGLE_CLOUD_PROJECT/email-service \
       --platform managed \
       --region us-west1 \
       --no-allow-unauthenticated \
       --max-instances=1
     ```
   - Make it executable and run it:
     ```bash
     chmod u+x deploy.sh
     ./deploy.sh
     ```

### Task 4: Configure Pub/Sub to Trigger the Email Service
1. **Create Service Account**:
   ```bash
   gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"
   ```

2. **Assign IAM Policy**:
   ```bash
   gcloud run services add-iam-policy-binding email-service \
     --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
     --role=roles/run.invoker \
     --region us-west1 \
     --platform managed
   ```

3. **Create Pub/Sub Subscription**:
   ```bash
   EMAIL_SERVICE_URL=$(gcloud run services describe email-service --platform managed --region us-west1 --format="value(status.address.url)")
   gcloud pubsub subscriptions create email-service-sub --topic new-lab-report --push-endpoint=$EMAIL_SERVICE_URL --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
   ```

4. **Test Services**:
   - Run:
     ```bash
     ~/pet-theory/lab05/lab-service/post-reports.sh
     ```
   - Check logs in Cloud Console > Cloud Run > email-service.

### Task 5: Build and Deploy the SMS Service
1. **Navigate to SMS Service Directory**:
   ```bash
   cd ~/pet-theory/lab05/sms-service
   ```

2. **Install Dependencies**:
   ```bash
   npm install express body-parser
   ```

3. **Update `package.json`**:
   Add the following to the "scripts" section:
   ```json
   "start": "node index.js",
   ```

4. **Create `index.js`**:
   Similar to the Email Service, add logic to handle Pub/Sub messages and simulate sending an SMS.

5. **Create and Deploy SMS Service**:
   - Similar to the Email Service, create `Dockerfile` and `deploy.sh`.
   - Deploy using `./deploy.sh`.

6. **Link Pub/Sub to SMS Service**:
   ```bash
   SMS_SERVICE_URL=$(gcloud run services describe sms-service --platform managed --region us-west1 --format="value(status.address.url)")
   gcloud pubsub subscriptions create sms-service-sub --topic new-lab-report --push-endpoint=$SMS_SERVICE_URL --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_C

LOUD_PROJECT.iam.gserviceaccount.com
   ```

### Task 6: Test System Resilience
1. **Simulate Failure**:
   Modify the `index.js` of the Email Service to throw an error in `sendEmail()` and redeploy.
   ```javascript
   function sendEmail() {
     throw 'Email server is down';
     console.log('Sending email');
   }
   ```

2. **Fix and Redeploy**:
   - Remove the error-throwing line and redeploy the Email Service.

## Takeaways
- **Resilience**: Pub/Sub retries failed message deliveries.
- **Loose Coupling**: Services communicate asynchronously via Pub/Sub, enhancing resilience and scalability.
- **Healing**: Services automatically recover and process messages once they're back online.
