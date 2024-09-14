# Build a Serverless Web App with Firebase

## Overview
Pet Theory, a veterinary clinic chain, is transitioning to a serverless web app to allow clients to schedule appointments in real-time. In this lab, you will create a Firebase-based web application that facilitates user authentication, customer information management, and real-time appointment scheduling.

## Objectives
- Configure Firestore Security for automated authentication and authorization.
- Add Google Sign-in to your web app.
- Enable users to add and update their contact information in the Firestore database.
- Deploy code that supports real-time appointment scheduling.
- Utilize Firebase's real-time updates in the web app.

## Prerequisites
- Basic familiarity with Google Cloud Console and shell environments.
- Experience with Firebase is helpful but not necessary.
- Completion of the "Importing Data to a Firestore Database" lab is recommended.
- Comfort with editing files using text editors like nano, vi, or Cloud Shell's built-in editor.

## Instructions

### Task 1: Register a Firebase Application
1. **Open Firebase Console**:
   - Open an incognito window and navigate to the [Firebase Console](https://console.firebase.google.com/).
   - Log in using the provided credentials (username and password).

2. **Select or Register the Firebase App**:
   - Choose the provisioned project with the label `PROJECT_ID`.
   - Click the web icon in the "Get started by adding Firebase to your app" section.
   - Provide an "App nickname": `Pet Theory`.
   - Check the box to **"Also set up Firebase hosting for this app."**
   - Click **Register app**, then proceed with **Next > Next > Continue to console**.

### Task 2: Enable Firebase Authentication
1. **Enable Google Sign-In**:
   - In the Firebase Console, navigate to **Build > Authentication** and click **Get Started**.
   - Select the **Sign-in method** tab, then click **Google**.
   - Toggle the **Enable** switch and select the **Support email** (your lab account) from the dropdown.
   - Click **Save**.

### Task 3: Install the Firebase CLI
1. **Open Cloud Code IDE**:
   - Copy the IDE link from the lab details and paste it into a new incognito browser tab.
   - Open a terminal in the Cloud Code environment: **Application menu > Terminal > New terminal**.

2. **Clone the Repository**:
   ```bash
   git clone https://github.com/rosera/pet-theory.git
   ```
   
3. **Open the Project Folder**:
   - In the left panel, click the Explorer icon and open `pet-theory/lab02`.

4. **Install Node Packages**:
   ```bash
   npm i && npm audit fix --force
   ```

### Task 4: Authorize Firebase Access
1. **Login to Firebase**:
   - Run the following command to authorize Firebase:
     ```bash
     firebase login --no-localhost
     ```
   - Copy the generated URL into a new incognito browser tab, select your lab account, and allow access.
   - Copy the access code and paste it into the Cloud Shell prompt.

### Task 5: Initialize Firebase Products
1. **Initialize Firebase**:
   - Run the following command in the terminal:
     ```bash
     firebase init
     ```
   - Use the arrow keys to select **Firestore** and **Hosting** using the spacebar, then press **Enter**.
   - Choose **Use an existing project** and select your `PROJECT_ID`.
   - Follow the prompts:
     - Keep the `firestore.rules` and `firestore.indexes.json` files.
     - Keep the `public` directory and do not rewrite to `/index.html`.
     - Do not set up automatic builds and deploys with GitHub.
     - Do not overwrite the `404.html` and `index.html` files.

### Task 6: Deploying to Firebase
1. **Deploy the Application**:
   - In the `pet-theory/lab02` folder, run:
     ```bash
     firebase deploy
     ```
   - Copy the hosting URL from the output and open it in a new incognito tab.
   - Click on **Sign in with Google** and use the lab-provided username.

### Task 7: Add a Customer Page to Your Web App
1. **Edit `customer.js`**:
   - In the `public` folder, open `customer.js` and paste the following code:
     ```javascript
     let user;

     firebase.auth().onAuthStateChanged(function(newUser) {
       user = newUser;
       if (user) {
         const db = firebase.firestore();
         db.collection("customers").doc(user.email).onSnapshot(function(doc) {
           const cust = doc.data();
           if (cust) {
             document.getElementById('customerName').setAttribute('value', cust.name);
             document.getElementById('customerPhone').setAttribute('value', cust.phone);
           }
           document.getElementById('customerEmail').innerText = user.email;
         });
       }
     });

     document.getElementById('saveProfile').addEventListener('click', function(ev) {
       const db = firebase.firestore();
       var docRef = db.collection('customers').doc(user.email);
       docRef.set({
         name: document.getElementById('customerName').value,
         email: user.email,
         phone: document.getElementById('customerPhone').value,
       });
     });
     ```

2. **Edit `styles.css`**:
   - In the `public` folder, open `styles.css` and paste the following code:
     ```css
     body { background: #ECEFF1; color: rgba(0,0,0,0.87); font-family: Roboto, Helvetica, Arial, sans-serif; margin: 0; padding: 0; }
     #message { background: white; max-width: 360px; margin: 100px auto 16px; padding: 32px 24px 16px; border-radius: 3px; }
     #message h3 { color: #888; font-weight: normal; font-size: 16px; margin: 16px 0 12px; }
     #message h2 { color: #ffa100; font-weight: bold; font-size: 16px; margin: 0 0 8px; }
     #message h1 { font-size: 22px; font-weight: 300; color: rgba(0,0,0,0.6); margin: 0 0 16px;}
     #message p { line-height: 140%; margin: 16px 0 24px; font-size: 14px; }
     #message a { display: block; text-align: center; background: #039be5; text-transform: uppercase; text-decoration: none; color: white; padding: 16px; border-radius: 4px; }
     #message, #message a { box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24); }
     #load { color: rgba(0,0,0,0.4); text-align: center; font-size: 13px; }
     @media (max-width: 600px) {
       body, #message { margin-top: 0; background: white; box-shadow: none; }
       body { border-top: 16px solid #ffa100; }
     }
     ```

3. **Deploy the Changes**:
   - Run the following command to deploy the updates:
     ```bash
     firebase deploy
     ```

4. **Test the Application**:
   - Refresh the web app using a **hard refresh** (`CTRL+SHIFT+R` on Windows, `CMD+SHIFT+R` on Mac).
   - Enter some test customer information and click **Save profile**.
   - Go to **Firebase Console > Firestore Database** to view the saved profile information.

## Conclusion
In this lab, you:
- Set up Firebase Authentication and integrated Google Sign-In.
- Created and deployed a customer page that stores and retrieves data in Firestore.
- Learned how to use Firebase's real-time updates in a serverless web application.
