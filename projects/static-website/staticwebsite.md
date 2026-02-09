## 🚀 Steps to Launch a Static Website on AWS (S3)

This section outlines the steps I followed to deploy a static website using **Amazon S3**.

---

### 1️⃣ Create an S3 Bucket
- Logged into the AWS Management Console
- Navigated to **Amazon S3**
- Created a new bucket with a **globally unique name**
- Selected the appropriate AWS region

![Static Website](../static-website/step%202.jpeg)
---

### 2️⃣ Configure Bucket Settings
- Disabled **Block all public access** (required for public website hosting)
- Acknowledged the public access warning
- Enabled **Static website hosting**
- Set:
  - Index document: `index.html`
  - Error document: `error.html` (optional)

![Static Website](../static-website/step%205.jpeg)
---

### 3️⃣ Upload Website Files
- Uploaded all static files (HTML, CSS, images)
- Verified that files were correctly uploaded and visible in the bucket

![Static Website](../static-website/step%203.jpeg)


---

### 4️⃣ Update Bucket Policy for Public Access
- Added a bucket policy to allow public read access to objects
- Ensured the policy referenced the correct bucket name

![Static Website](../static-website/step%204.jpeg)

---

### 5️⃣ Verify Website Endpoint
- Opened the **S3 static website endpoint URL**
- Confirmed that the website loaded successfully in the browser

![Static Website](../static-website/Screenshot%202026-02-06%20at%2008.08.25.png)

---

### 6️⃣ Test and Update Content
- Tested page navigation and styling
- Re-uploaded files when updates were needed
- Confirmed changes reflected immediately via the S3 endpoint

![Static Website](../static-website/Screenshot%202026-02-06%20at%2008.08.30.png)

---

## ✅ Outcome
The static website was successfully deployed and made publicly accessible using **Amazon S3**, demonstrating a simple, cost-effective method for hosting static content on AWS.


[def]: ../static-website/step%201.jpeg