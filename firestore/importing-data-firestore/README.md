## Importing Data to a Firestore Database

### Overview
Pet Theory, a chain of veterinary clinics, is transitioning to a cloud-based scheduling system using Firestore to handle increased customer data. In this lab, you'll help set up a Firestore database, generate test data, and import data into Firestore using Node.js. 

### Objectives
- Set up Firestore in Google Cloud.
- Write Node.js code to import data into Firestore.
- Generate test data using a pseudo-random data generator.
- Import test customer data into Firestore.
- Inspect the imported data in Firestore.

### Prerequisites
- Basic familiarity with the Google Cloud Console and shell environments.
- Basic knowledge of Node.js.
- Experience with Firebase or Firestore is helpful but not required.

### Setup
- Access the lab in an incognito browser window.
- Use the provided temporary credentials to log in to the Google Cloud Console.

### Instructions

#### Task 1: Set up Firestore in Google Cloud
1. **Open Firestore**:
   - Go to the Google Cloud Console, and from the Navigation menu, select **Firestore**.
   
2. **Create a Firestore Database**:
   - Click **+Create Database**.
   - Choose **Native mode** and click **Continue**.
   - Select a region in the **Region** dropdown, then click **Create Database**.

#### Task 2: Write Database Import Code
1. **Clone the Pet Theory Repository**:
   ```bash
   git clone https://github.com/rosera/pet-theory
   ```
   
2. **Change to the Lab Directory**:
   ```bash
   cd pet-theory/lab01
   ```

3. **Install Required Node.js Packages**:
   - To interact with Firestore:
     ```bash
     npm install @google-cloud/firestore
     ```
   - To log information to Cloud Logging:
     ```bash
     npm install @google-cloud/logging
     ```

4. **Edit the Import Script**:
   - Open `importTestData.js` using the code editor.

5. **Add Firestore and Logging Dependencies**:
   - At the top of the file, add:
     ```javascript
     const { Firestore } = require("@google-cloud/firestore");
     const { Logging } = require('@google-cloud/logging');
     ```

6. **Add Write to Firestore Function**:
   - Add the following code to write data to Firestore:
     ```javascript
     async function writeToFirestore(records) {
       const db = new Firestore();
       const batch = db.batch();

       records.forEach((record) => {
         console.log(`Write: ${record}`);
         const docRef = db.collection("customers").doc(record.email);
         batch.set(docRef, record, { merge: true });
       });

       batch.commit()
         .then(() => {
           console.log('Batch executed');
         })
         .catch(err => {
           console.log(`Batch error: ${err}`);
         });
     }
     ```

7. **Update the `importCsv` Function**:
   - Modify the function to call `writeToFirestore` instead of `writeToDatabase`:
     ```javascript
     async function importCsv(csvFilename) {
       const parser = csv.parse({ columns: true, delimiter: ',' }, async function (err, records) {
         if (err) {
           console.error('Error parsing CSV:', err);
           return;
         }
         try {
           console.log(`Call write to Firestore`);
           await writeToFirestore(records);
           console.log(`Wrote ${records.length} records`);
         } catch (e) {
           console.error(e);
           process.exit(1);
         }
       });

       await fs.createReadStream(csvFilename).pipe(parser);
     }
     ```

8. **Add Logging for the Application**:
   - Initialize the Logging client at the top of the file:
     ```javascript
     const logName = "pet-theory-logs-importTestData";
     const logging = new Logging();
     const log = logging.log(logName);
     const resource = { type: "global" };
     ```
   - Add logging to the `importCsv` function:
     ```javascript
     success_message = `Success: importTestData - Wrote ${records.length} records`;
     const entry = log.entry({ resource: resource }, { message: `${success_message}` });
     log.write([entry]);
     ```

#### Task 3: Create Test Data
1. **Install Faker Library**:
   ```bash
   npm install faker@5.5.3
   ```

2. **Create Test Data Script**:
   - Open `createTestData.js` and add the Logging API module:
     ```javascript
     const { Logging } = require("@google-cloud/logging");
     ```
   - Initialize the Logging client:
     ```javascript
     const logName = "pet-theory-logs-createTestData";
     const logging = new Logging();
     const log = logging.log(logName);
     const resource = { type: "global" };
     ```
   - Add logging to the `createTestData` function:
     ```javascript
     const success_message = `Success: createTestData - Created file ${fileName} containing ${recordCount} records.`;
     const entry = log.entry({ resource: resource }, { message: `${success_message}` });
     log.write([entry]);
     ```
   
3. **Generate Test Data**:
   - Run the following command to create a CSV file with 1000 records:
     ```bash
     node createTestData 1000
     ```
   - Verify that the file `customers_1000.csv` is created successfully.

#### Task 4: Import the Test Customer Data
1. **Run the Import Script**:
   - Use the following command to import the generated data into Firestore:
     ```bash
     node importTestData customers_1000.csv
     ```

2. **Troubleshooting**:
   - If you encounter an error related to `csv-parse`, install it using:
     ```bash
     npm install csv-parse
     ```
   - Rerun the import command if necessary.

#### Task 5: Inspect the Data in Firestore
1. **Open Firestore in the Cloud Console**:
   - In the Navigation menu, select **Firestore**.
   - Click the pencil icon and type `/customers` to view the imported customer data.

### Conclusion
In this lab, you:
- Set up a Firestore database in Google Cloud.
- Developed a Node.js script to import data into Firestore.
- Created pseudo-random test data using the Faker library.
- Imported the test data into Firestore and verified the data in the Cloud Console.
