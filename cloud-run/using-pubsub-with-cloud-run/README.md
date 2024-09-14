## Using Cloud Pub/Sub with Cloud Run

### Overview
Pub/Sub enables applications to utilize efficient message queues. In this lab, you'll learn how to integrate Pub/Sub with Cloud Run to achieve resilient communication between microservices.

### Objectives
- Enable the Cloud Run API.
- Deploy microservices to Cloud Run.
- Create a Pub/Sub topic.
- Invoke a Cloud Run service from a Pub/Sub subscription.

### Prerequisites
- Intermediate knowledge of Google Cloud.
- Familiarity with Pub/Sub and Cloud Run is helpful.

### Setup and Requirements
- Use an incognito window to avoid conflicts with your personal Google Cloud account.
- Do not use your personal Google Cloud account to avoid incurring charges.

### Lab Instructions

#### Task 1: Ensure Pub/Sub API is Enabled
1. **Enable the Pub/Sub API**:
   - Navigate to **APIs & Services > Library**.
   - Search for **Cloud Pub/Sub API** and click **Manage**.
   - Disable and then re-enable the API.

#### Task 2: Developing a Minimal Viable Product (MVP)
The goal is to deploy a producer service (`store-service`) and a consumer service (`order-service`).

##### Deploy the Producer Service (`store-service`)
1. **Enable Cloud Run API** and set the environment:
   ```bash
   gcloud services enable run.googleapis.com
   LOCATION=REGION
   gcloud config set compute/region $LOCATION
   ```

2. **Deploy the Store Service**:
   ```bash
   gcloud run deploy store-service \
     --image gcr.io/qwiklabs-resources/gsp724-store-service \
     --region $LOCATION \
     --allow-unauthenticated
   ```

##### Deploy the Consumer Service (`order-service`)
1. **Deploy the Order Service**:
   ```bash
   gcloud run deploy order-service \
     --image gcr.io/qwiklabs-resources/gsp724-order-service \
     --region $LOCATION \
     --no-allow-unauthenticated
   ```

#### Task 3: Deploying Pub/Sub
##### Create a Pub/Sub Topic
1. **Create a Topic**:
   ```bash
   gcloud pubsub topics create ORDER_PLACED
   ```

#### Task 4: Creating a Service Account
To allow Pub/Sub to invoke the `order-service`, you need to create a service account and assign the necessary permissions.

1. **Create a Service Account**:
   ```bash
   gcloud iam service-accounts create pubsub-cloud-run-invoker \
     --display-name "Order Initiator"
   ```

2. **Bind the Service Account with the `Cloud Run Invoker` Role**:
   ```bash
   gcloud run services add-iam-policy-binding order-service --region $LOCATION \
     --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
     --role=roles/run.invoker --platform managed
   ```

3. **Enable the Project Service Account to Create Tokens**:
   ```bash
   PROJECT_NUMBER=$(gcloud projects list --filter="qwiklabs-gcp" --format='value(PROJECT_NUMBER)')
   gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
     --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
     --role=roles/iam.serviceAccountTokenCreator
   ```

#### Task 5: Create a Pub/Sub Subscription
1. **Store the Endpoint of `order-service`**:
   ```bash
   ORDER_SERVICE_URL=$(gcloud run services describe order-service --region $LOCATION --format="value(status.address.url)")
   ```

2. **Create a Subscription**:
   ```bash
   gcloud pubsub subscriptions create order-service-sub \
     --topic ORDER_PLACED \
     --push-endpoint=$ORDER_SERVICE_URL \
     --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
   ```

#### Task 6: Testing the Application
1. **Create a Test JSON File**:
   ```bash
   cat > test.json <<EOF
   {
     "billing_address": {
       "name": "Kylie Scull",
       "address": "6471 Front Street",
       "city": "Mountain View",
       "state_province": "CA",
       "postal_code": "94043",
       "country": "US"
     },
     "shipping_address": {
       "name": "Kylie Scull",
       "address": "9902 Cambridge Grove",
       "city": "Martinville",
       "state_province": "BC",
       "postal_code": "V1A",
       "country": "Canada"
     },
     "items": [
       {
         "id": "RW134",
         "quantity": 1,
         "sub-total": 12.95
       },
       {
         "id": "IB541",
         "quantity": 2,
         "sub-total": 24.5
       }
     ]
   }
   EOF
   ```

2. **Store the Endpoint of `store-service`**:
   ```bash
   STORE_SERVICE_URL=$(gcloud run services describe store-service --region $LOCATION --format="value(status.address.url)")
   ```

3. **Post a Message to `store-service`**:
   ```bash
   curl -X POST -H "Content-Type: application/json" -d @test.json $STORE_SERVICE_URL
   ```

4. **Verify the Logs**:
   - In the **Google Cloud Console**, navigate to **Cloud Run > store-service** to view logs and see the generated order ID.
   - Similarly, view the **order-service** logs to confirm the JSON data was successfully received.

### Conclusion
In this lab, you learned how to:
- Deploy services to Cloud Run.
- Create a Service Account with the appropriate role and permissions.
- Define a Pub/Sub Topic.
- Bind a Pub/Sub Subscription to a Service Account to facilitate communication between Cloud Run services.
