## Develop an App with Vertex AI Gemini 1.0 Pro

### Overview
Gemini is a family of generative AI models designed for multimodal use cases. In this lab, you'll use the Vertex AI SDK for Python to interact with Gemini 1.0 Pro and Gemini 1.0 Pro Vision models and develop a Python app using the Streamlit framework. You will also containerize your application and deploy it on Cloud Run.

### Objectives
- Develop a Python app using the Streamlit framework.
- Install the Vertex AI SDK for Python.
- Develop code to interact with the Gemini 1.0 Pro model using the Vertex AI Gemini API.
- Develop code to interact with the Gemini 1.0 Pro Vision model using the Vertex AI Gemini API.
- Containerize the application and deploy it on Cloud Run.

### Prerequisites
- Basic understanding of Python, machine learning models, and Google Cloud services like Vertex AI and Cloud Run.

### Setup
- Access the lab using an incognito window.
- Note the lab's access time and plan to finish within that time.
- Use the provided temporary credentials to log in to the Google Cloud Console.

### Lab Instructions

#### Task 1: Configure the Environment and Project
1. **Set Project ID and Region**:
   ```bash
   PROJECT_ID=$(gcloud config get-value project)
   REGION=set at lab start
   echo "PROJECT_ID=${PROJECT_ID}"
   echo "REGION=${REGION}"
   ```

2. **Enable Required APIs**:
   ```bash
   gcloud services enable cloudbuild.googleapis.com cloudfunctions.googleapis.com run.googleapis.com logging.googleapis.com storage-component.googleapis.com aiplatform.googleapis.com
   ```

#### Task 2: Set Up the Application Environment
1. **Create App Directory**:
   ```bash
   mkdir ~/gemini-app
   cd ~/gemini-app
   ```

2. **Create a Python Virtual Environment**:
   ```bash
   python3 -m venv gemini-streamlit
   source gemini-streamlit/bin/activate
   ```

3. **Create the Requirements File**:
   ```bash
   cat > ~/gemini-app/requirements.txt <<EOF
   streamlit
   google-cloud-aiplatform==1.38.1
   google-cloud-logging==3.6.0
   EOF
   ```

4. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

#### Task 3: Develop the App
1. **Create the Main App Entry Point**:
   ```bash
   cat > ~/gemini-app/app.py <<EOF
   # Add your Python code for the app here
   EOF
   ```

2. **Develop the Story Tab**:
   ```bash
   cat > ~/gemini-app/app_tab1.py <<EOF
   # Add code to implement the story tab here
   EOF
   ```

3. **Develop Response Utility Functions**:
   ```bash
   cat > ~/gemini-app/response_utils.py <<EOF
   # Add response utility functions here
   EOF
   ```

#### Task 4: Run and Test the App Locally
1. **Run the App**:
   ```bash
   streamlit run app.py --browser.serverAddress=localhost --server.enableCORS=false --server.enableXsrfProtection=false --server.port 8080
   ```
2. **Access the App**: Use the provided URL or Cloud Shell Web Preview on port 8080 to access the app.

#### Task 5-13: Develop Additional Features
1. **Implement the Marketing Campaign Tab**:
   - Write code for `app_tab2.py` to generate a marketing campaign.

2. **Implement Image Playground**:
   - Develop multiple tabs like "Furniture recommendation," "Oven instructions," "ER diagrams," and "Math reasoning" in `app_tab3.py`.

3. **Implement Video Playground**:
   - Develop "Video description," "Video tags," "Video highlights," and "Video geolocation" tabs in `app_tab4.py`.

4. **Modify the Main App File to Include All Tabs**:
   ```bash
   cat >> ~/gemini-app/app.py <<EOF
   # Add code to include all developed tabs
   EOF
   ```

#### Task 14: Deploy the App to Cloud Run
1. **Set Up Environment Variables**:
   ```bash
   SERVICE_NAME='gemini-app-playground'
   AR_REPO='gemini-app-repo'
   ```

2. **Create Docker Repository**:
   ```bash
   gcloud artifacts repositories create "$AR_REPO" --location="$REGION" --repository-format=Docker
   gcloud auth configure-docker "$REGION-docker.pkg.dev"
   ```

3. **Create a Dockerfile**:
   ```bash
   cat > ~/gemini-app/Dockerfile <<EOF
   FROM python:3.8
   EXPOSE 8080
   WORKDIR /app
   COPY . ./
   RUN pip install -r requirements.txt
   ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8080", "--server.address=0.0.0.0"]
   EOF
   ```

4. **Build the Container Image**:
   ```bash
   gcloud builds submit --tag "$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME"
   ```

5. **Deploy the App to Cloud Run**:
   ```bash
   gcloud run deploy "$SERVICE_NAME" \
     --port=8080 \
     --image="$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME" \
     --allow-unauthenticated \
     --region=$REGION \
     --platform=managed \
     --set-env-vars=PROJECT_ID=$PROJECT_ID,REGION=$REGION
   ```

### Conclusion
In this lab, you:
- Developed a Python app using the Streamlit framework.
- Installed the Vertex AI SDK for Python.
- Interacted with the Gemini 1.0 Pro and Pro Vision models using the Vertex AI Gemini API.
- Used text, images, and videos to generate stories, marketing campaigns, recommendations, instructions, and more.
- Deployed and tested the app on Cloud Run.
