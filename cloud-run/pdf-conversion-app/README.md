# Build a Serverless App with Cloud Run that Creates PDF Files

## Overview
Pet Theory, a veterinary clinic chain, seeks to improve customer satisfaction by converting invoices from DOCX to PDF format automatically. This serverless app uses **Google Cloud Run** to handle the conversion, leveraging **LibreOffice** for the DOCX-to-PDF conversion, **Cloud Storage** for file storage, and **Cloud Pub/Sub** for event processing. 

The goal is to build a cost-efficient, serverless solution that scales to zero when not in use, minimizing operational costs and overhead.

## Architecture
The app leverages the following services:
- **Cloud Run**: To run the conversion service.
- **Cloud Build**: For building Docker images.
- **Cloud Storage**: For uploading and storing files (both original DOCX and converted PDF).
- **Cloud Pub/Sub**: For triggering file conversion on file upload events.

## Objectives
- Convert a Node.js application into a container.
- Build containers using Google Cloud Build.
- Deploy a Cloud Run service to convert files to PDF.
- Set up Cloud Storage buckets to handle file uploads and processed files.
- Trigger the PDF conversion using Cloud Storage events via Cloud Pub/Sub.

## Prerequisites
- Familiarity with Google Cloud Console and shell environments.
- Experience with Firebase and serverless products.
- Completion of the "Importing Data to a Firestore Database" and "Build a Serverless Web App with Firebase" labs.

## Instructions

### Task 1: Enable the Cloud Run API
1. Open **APIs & Services > Library** in the Google Cloud Console.
2. Search for **Cloud Run** and enable the **Cloud Run Admin API**.

### Task 2: Deploy a Simple Cloud Run Service
1. **Clone the Pet Theory Repository**:
   ```bash
   git clone https://github.com/rosera/pet-theory.git
   cd pet-theory/lab03
   ```

2. **Update `package.json`**:
   Edit the file and add the following under `"scripts"`:
   ```json
   "start": "node index.js",
   ```

3. **Install Required Packages**:
   ```bash
   npm install express body-parser child_process @google-cloud/storage
   ```

4. **Build the Docker Container**:
   Review the `Dockerfile` and then run the following command to build the container:
   ```bash
   gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter
   ```

5. **Deploy the Cloud Run Service**:
   ```bash
   gcloud run deploy pdf-converter \
     --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
     --platform managed \
     --region us-west1 \
     --no-allow-unauthenticated \
     --max-instances=1
   ```

6. **Set Environment Variables**:
   ```bash
   SERVICE_URL=$(gcloud beta run services describe pdf-converter --platform managed --region us-west1 --format="value(status.url)")
   echo $SERVICE_URL
   ```

7. **Test the Deployment**:
   - Make an authorized POST request:
     ```bash
     curl -X POST -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
     ```
   If you see an "OK" response, the service is deployed correctly.

### Task 3: Set Up Cloud Storage and Pub/Sub
1. **Create Cloud Storage Buckets**:
   ```bash
   gsutil mb gs://$GOOGLE_CLOUD_PROJECT-upload
   gsutil mb gs://$GOOGLE_CLOUD_PROJECT-processed
   ```

2. **Create a Pub/Sub Topic**:
   ```bash
   gsutil notification create -t new-doc -f json -e OBJECT_FINALIZE gs://$GOOGLE_CLOUD_PROJECT-upload
   ```

3. **Create a Pub/Sub Service Account**:
   ```bash
   gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"
   ```

4. **Grant Permissions to the Service Account**:
   ```bash
   gcloud beta run services add-iam-policy-binding pdf-converter \
     --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
     --role=roles/run.invoker --platform managed --region us-west1
   ```

5. **Enable Cloud Pub/Sub Authentication**:
   ```bash
   gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
     --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
     --role=roles/iam.serviceAccountTokenCreator
   ```

6. **Create a Pub/Sub Subscription**:
   ```bash
   gcloud beta pubsub subscriptions create pdf-conv-sub \
     --topic new-doc \
     --push-endpoint=$SERVICE_URL \
     --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
   ```

### Task 4: Update the Application
1. **Install LibreOffice in Docker**:
   Add the following line in the Dockerfile:
   ```Dockerfile
   RUN apt-get update -y && apt-get install -y libreoffice && apt-get clean
   ```

2. **Update the `index.js` File**:
   Add the logic to handle file conversion using LibreOffice. The main operations are:
   - Download the file from Cloud Storage.
   - Convert the file to PDF using LibreOffice.
   - Upload the PDF back to Cloud Storage.
   - Delete the original file.

3. **Rebuild and Deploy**:
   ```bash
   gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter
   gcloud run deploy pdf-converter \
     --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
     --platform managed --region us-west1 \
     --memory=2Gi --no-allow-unauthenticated --max-instances=1 \
     --set-env-vars PDF_BUCKET=$GOOGLE_CLOUD_PROJECT-processed
   ```

### Task 5: Test the Service
1. **Upload Test Files**:
   ```bash
   gsutil -m cp gs://spls/gsp644/* gs://$GOOGLE_CLOUD_PROJECT-upload
   ```

2. **Verify the Conversion**:
   Check Cloud Logging for logs related to the uploaded files, and confirm that PDFs are created in the `-processed` bucket.

---

## Conclusion
Congratulations! You have successfully built and deployed a serverless Cloud Run service that converts DOCX files to PDFs. By leveraging Google Cloud's serverless technologies, Pet Theory now has a scalable, cost-effective solution for managing document conversions.
