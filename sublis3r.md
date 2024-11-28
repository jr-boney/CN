# connecting frotend and backend
No problem! I’ll walk you through the steps in detail so you can get this working, even as a beginner. We'll break this down into small, manageable parts.

### High-Level Overview:
1. **Backend Server**: We will set up a server that runs Sublist3r, processes requests, and returns results via an API.
2. **Next.js Frontend**: Your web application (Next.js) will allow users to enter a domain and then request subdomains from the backend.

Here’s how you can do it step by step:

---

### Step 1: Set Up Your Backend Server (Flask with Sublist3r)

#### 1.1 Install Python and Flask on Your Server

To run Sublist3r, you need to have **Python** installed. If you're running on your local machine or a cloud server, make sure Python is installed. If you're not sure, you can check by running:

```bash
python --version
```

or

```bash
python3 --version
```

If Python isn’t installed, you can download it from [here](https://www.python.org/downloads/).

Next, we need **Flask** (a Python web framework) to create the API.

**Install Flask**:
```bash
pip install flask
```

Now let’s download **Sublist3r** (a Python script) and set it up.

#### 1.2 Install Sublist3r

1. Go to the directory where you want to store Sublist3r and clone it from GitHub:

   ```bash
   git clone https://github.com/aboul3la/Sublist3r.git
   ```

2. Install the required libraries for Sublist3r:

   ```bash
   cd Sublist3r
   pip install -r requirements.txt
   ```

This ensures Sublist3r has the necessary dependencies.

#### 1.3 Create a Python API (Flask) to Run Sublist3r

We will now create an API endpoint to run Sublist3r whenever a request is made.

1. In the same directory as Sublist3r, create a new file called `sublist3r_api.py`.

2. In `sublist3r_api.py`, copy and paste the following code:

   ```python
   from flask import Flask, request, jsonify
   import subprocess

   app = Flask(__name__)

   @app.route('/get-subdomains', methods=['GET'])
   def get_subdomains():
       domain = request.args.get('domain')
       if not domain:
           return jsonify({"error": "Domain parameter is required"}), 400

       try:
           # Run Sublist3r using subprocess
           result = subprocess.run(
               ['python3', 'Sublist3r/sublist3r.py', '-d', domain],
               stdout=subprocess.PIPE, stderr=subprocess.PIPE
           )

           # Get the output from Sublist3r
           subdomains = result.stdout.decode('utf-8').split('\n')

           # Return the subdomains as a JSON response
           return jsonify({"subdomains": subdomains})

       except Exception as e:
           return jsonify({"error": str(e)}), 500

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000, debug=True)
   ```

**Explanation**:
- **Flask** creates a web API that listens for requests.
- The endpoint `/get-subdomains` will accept a `domain` query parameter, run Sublist3r with that domain, and return the subdomains found.
- **subprocess.run** runs the Sublist3r command in the background and captures the output.

#### 1.4 Run Your Flask API

Now, you can run the Flask API:

```bash
python sublist3r_api.py
```

Your Flask server should now be running on `http://0.0.0.0:5000`, and you can access the `/get-subdomains` endpoint.

To test it manually in a browser, go to:

```
http://<your-server-ip>:5000/get-subdomains?domain=example.com
```

Replace `<your-server-ip>` with the actual IP address of the machine where the server is running.

---

### Step 2: Set Up Your Frontend in Next.js

#### 2.1 Install Next.js

If you don’t already have a Next.js project, you can create one by running the following commands in your terminal:

1. **Install Node.js**: If you don't have Node.js installed, download it from [here](https://nodejs.org/).

2. **Create a Next.js App**:

   ```bash
   npx create-next-app@latest my-nextjs-app
   cd my-nextjs-app
   ```

3. **Install Axios** (for making HTTP requests from your Next.js app):

   ```bash
   npm install axios
   ```

#### 2.2 Create an API Route to Call Your Backend Server

Inside your Next.js project, you need to create an API route that will communicate with your Flask backend.

1. In the `pages/api` folder of your Next.js app, create a file called `get-subdomains.js`.

2. In `get-subdomains.js`, add the following code:

   ```javascript
   import axios from 'axios';

   export default async function handler(req, res) {
       const { domain } = req.query;

       if (!domain) {
           return res.status(400).json({ error: "Domain is required" });
       }

       try {
           // Make a GET request to your Flask API
           const response = await axios.get(`http://<backend-server-ip>:5000/get-subdomains?domain=${domain}`);

           // Return the subdomains to the Next.js frontend
           return res.status(200).json(response.data);
       } catch (error) {
           return res.status(500).json({ error: 'Failed to fetch subdomains' });
       }
   }
   ```

Replace `<backend-server-ip>` with the actual IP address or domain of the Flask backend server.

#### 2.3 Create the Frontend to Call the API

Now, create the user interface where users can input a domain and get subdomains.

1. Open `pages/index.js` and replace its contents with the following code:

   ```javascript
   import { useState } from 'react';

   export default function Home() {
       const [domain, setDomain] = useState('');
       const [subdomains, setSubdomains] = useState([]);
       const [error, setError] = useState(null);

       const fetchSubdomains = async () => {
           try {
               const res = await fetch(`/api/get-subdomains?domain=${domain}`);
               const data = await res.json();

               if (data.subdomains) {
                   setSubdomains(data.subdomains);
               } else {
                   setError(data.error || 'Unknown error');
               }
           } catch (error) {
               setError('Failed to fetch subdomains');
           }
       };

       return (
           <div>
               <h1>Sublist3r with Next.js</h1>
               <input
                   type="text"
                   value={domain}
                   onChange={(e) => setDomain(e.target.value)}
                   placeholder="Enter domain"
               />
               <button onClick={fetchSubdomains}>Get Subdomains</button>

               {error && <p style={{ color: 'red' }}>{error}</p>}

               <ul>
                   {subdomains.length > 0 &&
                       subdomains.map((sub, index) => <li key={index}>{sub}</li>)}
               </ul>
           </div>
       );
   }
   ```

This code creates a simple form where users can enter a domain, then press the "Get Subdomains" button to retrieve the results.

#### 2.4 Run Your Next.js Application

Now, you can run your Next.js app:

```bash
npm run dev
```

Open your browser and go to:

```
http://localhost:3000
```

You should see the form to input a domain. Once you enter a domain and click "Get Subdomains," the app will fetch the subdomains from the Flask API you created earlier.

---

### Step 3: Deploy Both Backend and Frontend

1. **Deploy Backend (Flask)**: You can deploy your Flask API on a server (e.g., AWS EC2, DigitalOcean, Heroku). Make sure it's publicly accessible.

2. **Deploy Frontend (Next.js)**: You can deploy your Next.js app to platforms like Vercel, Netlify, or any other hosting service.

3. **Update API URL in Next.js**: Once deployed, update the `axios.get` URL in your Next.js API route to point to the actual IP or domain of the backend server.

---

### Summary:

1. **Backend (Flask)**: Creates an API that runs Sublist3r and returns subdomains.
2. **Next.js Frontend**: Allows users to input a domain and fetch subdomains from the backend.
3. **Communication**: The Next.js app communicates with the Flask backend via HTTP requests (using Axios).

This setup should allow you to run Sublist3r as a service, and your Next.js app will make API requests to retrieve subdomains. Let me know if you need any further clarification!
