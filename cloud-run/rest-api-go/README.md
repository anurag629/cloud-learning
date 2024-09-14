# Developing a REST API with Go and Cloud Run

## Overview
Lily, the founder of Pet Theory, needs a way for insurance companies to access customer treatment costs without exposing personal identifiable information (PII). In this lab, you'll build a secure REST API using Go and Cloud Run, allowing authorized third-party access to Pet Theory's customer data.

## Objectives
- Develop a REST API with Go.
- Import test customer data into Firestore.
- Connect the REST API to the Firestore database.
- Deploy the REST API to Cloud Run.

## Prerequisites
- Familiarity with the Cloud Console and Cloud Shell environments.
- Completion of the following labs in the series is helpful but not required:
  - Importing Data to a Serverless Database
  - Building a Serverless Web App with Firebase and Firestore
  - Building a Serverless App that Creates PDF Files

## Setup and Requirements
- **Browser**: Use Chrome in Incognito mode to avoid conflicts with personal Google Cloud accounts.
- **Time**: The lab duration is one hour.
- **Credentials**: Temporary Google Cloud credentials provided by the lab.

## Instructions

### Task 1: Enable Google APIs
- For this lab, the required APIs have been enabled for you:
  - **Cloud Build**: `cloudbuild.googleapis.com`
  - **Cloud Run**: `run.googleapis.com`

### Task 2: Developing the REST API
1. **Activate the Project**:
   ```bash
   gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')
   ```

2. **Clone the Pet Theory Repository**:
   ```bash
   git clone https://github.com/rosera/pet-theory.git && cd pet-theory/lab08
   ```

3. **Create the REST API**:
   - Create a file named `main.go` and add the following code:
     ```go
     package main

     import (
       "fmt"
       "log"
       "net/http"
       "os"
     )

     func main() {
       port := os.Getenv("PORT")
       if port == "" {
           port = "8080"
       }
       http.HandleFunc("/v1/", func(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintf(w, "{status: 'running'}")
       })
       log.Println("Pets REST API listening on port", port)
       if err := http.ListenAndServe(":"+port, nil); err != nil {
           log.Fatalf("Error launching Pets REST API server: %v", err)
       }
     }
     ```

4. **Create the Dockerfile**:
   - Create a file named `Dockerfile` with the following content:
     ```Dockerfile
     FROM gcr.io/distroless/base-debian12
     WORKDIR /usr/src/app
     COPY server .
     CMD [ "/usr/src/app/server" ]
     ```

5. **Build the Go Binary**:
   ```bash
   go build -o server
   ```

6. **Verify Files**:
   - Ensure the directory contains `Dockerfile`, `go.mod`, `go.sum`, `main.go`, and `server`.

7. **Build and Deploy the Container**:
   - Build the container image:
     ```bash
     gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1
     ```
   - Deploy the container to Cloud Run:
     ```bash
     gcloud run deploy rest-api \
       --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 \
       --platform managed \
       --region "REGION" \
       --allow-unauthenticated \
       --max-instances=2
     ```

8. **Test the API**:
   - Access the URL from the deployment output, append `/v1/` to the end, and ensure it returns:
     ```
     {"status": "running"}
     ```

### Task 3: Import Test Customer Data into Firestore
1. **Create Firestore Database**:
   - Go to the Cloud Console: Navigation Menu > Firestore > Create Database.
   - Select **Native Mode**, choose your region, and click **Create Database**.

2. **Import Test Data**:
   - Create a Cloud Storage bucket and copy test data:
     ```bash
     gsutil mb -c standard -l REGION gs://$GOOGLE_CLOUD_PROJECT-customer
     gsutil cp -r gs://spls/gsp645/2019-10-06T20:10:37_43617 gs://$GOOGLE_CLOUD_PROJECT-customer
     ```
   - Import the data into Firestore:
     ```bash
     gcloud beta firestore import gs://$GOOGLE_CLOUD_PROJECT-customer/2019-10-06T20:10:37_43617/
     ```
   - Verify the data import by checking Firestore in the Cloud Console.

### Task 4: Connect the REST API to Firestore
1. **Update the REST API**:
   - Open `main.go` and update it to use Firestore. Replace the content with the following:
     ```go
     package main

     import (
       "context"
       "encoding/json"
       "fmt"
       "log"
       "net/http"
       "os"
       "cloud.google.com/go/firestore"
       "github.com/gorilla/handlers"
       "github.com/gorilla/mux"
       "google.golang.org/api/iterator"
     )

     var client *firestore.Client

     func main() {
       var err error
       ctx := context.Background()
       client, err = firestore.NewClient(ctx, "YOUR_PROJECT_ID")
       if err != nil {
           log.Fatalf("Error initializing Cloud Firestore client: %v", err)
       }

       port := os.Getenv("PORT")
       if port == "" {
           port = "8080"
       }

       r := mux.NewRouter()
       r.HandleFunc("/v1/", rootHandler)
       r.HandleFunc("/v1/customer/{id}", customerHandler)

       log.Println("Pets REST API listening on port", port)
       cors := handlers.CORS(
           handlers.AllowedHeaders([]string{"X-Requested-With", "Authorization", "Origin"}),
           handlers.AllowedOrigins([]string{"https://storage.googleapis.com"}),
           handlers.AllowedMethods([]string{"GET", "HEAD", "POST", "OPTIONS", "PATCH", "CONNECT"}),
       )

       if err := http.ListenAndServe(":"+port, cors(r)); err != nil {
           log.Fatalf("Error launching Pets REST API server: %v", err)
       }
     }
     ```

2. **Add Handler Functions**:
   - Append these functions to `main.go` to handle API requests and interact with Firestore:
     ```go
     func rootHandler(w http.ResponseWriter, r *http.Request) {
       fmt.Fprintf(w, "{status: 'running'}")
     }

     func customerHandler(w http.ResponseWriter, r *http.Request) {
       id := mux.Vars(r)["id"]
       ctx := context.Background()
       customer, err := getCustomer(ctx, id)
       if err != nil {
           w.WriteHeader(http.StatusInternalServerError)
           fmt.Fprintf(w, `{"status": "fail", "data": '%s'}`, err)
           return
       }
       if customer == nil {
           w.WriteHeader(http.StatusNotFound)
           msg := fmt.Sprintf("Customer \"%s\" not found", id)
           fmt.Fprintf(w, fmt.Sprintf(`{"status": "fail", "data": {"title": %s}}`, msg))
           return
       }
       amount, err := getAmounts(ctx, customer)
       if err != nil {
           w.WriteHeader(http.StatusInternalServerError)
           fmt.Fprintf(w, `{"status": "fail", "data": "Unable to fetch amounts: %s"}`, err)
           return
       }
       data, err := json.Marshal(amount)
       if err != nil {
           w.WriteHeader(http.StatusInternalServerError)
           fmt.Fprintf(w, `{"status": "fail", "data": "Unable to fetch amounts: %s"}`, err)
           return
       }
       fmt.Fprintf(w, fmt.Sprintf(`{"status": "success", "data": %s}`, data))
     }
     ```

3. **Add Customer Support**:
   - Append these functions to handle customer data:
     ```go
     type Customer struct {
       Email string `firestore:"email"`
       ID    string `firestore:"id"`
       Name  string `firestore:"name"`
       Phone string `firestore:"phone"`
     }

     func getCustomer(ctx context.Context, id string) (*Customer, error) {
       query := client.Collection("customers").Where("id", "==", id)
       iter := query.Documents(ctx)

       var c Customer
       for {
           doc, err := iter.Next()
           if err == iterator.Done {
               break
           }
           if err != nil {
               return nil, err
           }
           err = doc.DataTo(&c)
           if err != nil {
               return nil, err
           }
       }
       return &c, nil
     }

     func getAmounts(ctx context.Context, c *Customer) (map[string]int64, error) {
       if c == nil {
           return map[string]int64{}, fmt.Errorf("Customer should be non-nil: %v", c)
       }
       result := map[string]int64{
           "proposed": 0,
          

 "approved": 0,
           "rejected": 0,
       }
       query := client.Collection(fmt.Sprintf("customers/%s/treatments", c.Email))
       if query == nil {
           return map[string]int64{}, fmt.Errorf("Query is nil: %v", c)
       }
       iter := query.Documents(ctx)
       for {
           doc, err := iter.Next()
           if err == iterator.Done {
               break
           }
           if err != nil {
               return nil, err
           }
           treatment := doc.Data()
           result[treatment["status"].(string)] += treatment["cost"].(int64)
       }
       return result, nil
     }
     ```

### Task 5: Deploy the Updated REST API
1. **Rebuild the Source Code**:
   ```bash
   go build -o server
   ```

2. **Build and Deploy the New Image**:
   ```bash
   gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2
   gcloud run deploy rest-api \
     --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 \
     --platform managed \
     --region "REGION" \
     --allow-unauthenticated \
     --max-instances=2
   ```

3. **Test the API**:
   - Append `/customer/22530` to the service URL and ensure it returns:
     ```
     {"status" : "success", "data" :{"proposed" :1602, "approved" :585, "rejected" :489}}
     ```

## Conclusion
You have successfully built and deployed a scalable, serverless REST API that interacts with Firestore. This API now allows insurance companies to securely access customer data without exposing PII.
