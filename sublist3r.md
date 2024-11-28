

# Sublist3r Web App Deployment with Flask on Render

This guide walks you through how to deploy the **Sublist3r** subdomain enumeration tool with a **Flask web app** on **Render**, enabling non-technical users to interact with the tool through a simple web interface.

## Project Structure

```
my-sublist3r-web-app/
    ├── app.py            # Main Flask app
    ├── sublist3r.py      # Script to run Sublist3r
    ├── requirements.txt  # Dependencies
    └── Procfile          # Render deployment config
```

## Step 1: Set Up the Web Server (Flask)

Create the Flask application to handle HTTP requests, run Sublist3r, and return the results.

### 1.1 `app.py` (Flask App)

```python
from flask import Flask, request, jsonify
import subprocess

app = Flask(__name__)

@app.route('/')
def home():
    return """
    <html><body>
    <h1>Sublist3r Web Interface</h1>
    <form action="/sublist" method="POST">
        <input type="text" name="domain" placeholder="Enter domain" required>
        <button type="submit">Run Sublist3r</button>
    </form>
    </body></html>
    """

@app.route('/sublist', methods=['POST'])
def sublist():
    domain = request.form['domain']
    result = run_sublist3r(domain)
    return jsonify({"subdomains": result})

def run_sublist3r(domain):
    # Run Sublist3r as a subprocess and capture output
    try:
        result = subprocess.check_output(['python3', 'sublist3r.py', '-d', domain], stderr=subprocess.STDOUT)
        return result.decode('utf-8').split('\n')  # Return list of subdomains
    except subprocess.CalledProcessError as e:
        return str(e.output.decode('utf-8'))

if __name__ == '__main__':
    app.run(debug=True, host="0.0.0.0")
```

### 1.2 `sublist3r.py`

Clone **Sublist3r** from GitHub and add it to your project folder:

```bash
git clone https://github.com/aboul3la/Sublist3r.git
```

Copy the **`sublist3r.py`** script to the project folder.

### 1.3 `requirements.txt`

List the required dependencies:

```txt
Flask
requests
```

### 1.4 `Procfile` (Render Deployment)

The `Procfile` is used to tell Render how to start your app:

```txt
web: python app.py
```

## Step 2: Deploy to Render

### 2.1 Sign up for Render

Create an account on [Render](https://render.com/).

### 2.2 Create a New Web Service

1. Go to your **Render dashboard**.
2. Click on **"New"** and select **"Web Service"**.
3. **Connect your GitHub repository** or choose **"Deploy from Git"**.
4. Select the repository that contains your project.
5. Set the **Environment** to **Python**.
6. In **Build Command**, leave it empty or set it to `pip install -r requirements.txt`.
7. In **Start Command**, set it to `python app.py`.

Render will automatically build and deploy your app.

### 2.3 Access Your Web App

Once deployed, Render will provide a public URL, such as `https://my-sublist3r-web-app.onrender.com`. You can now access your app and interact with Sublist3r via the web interface.

## Step 3: Improve the User Interface

To improve the form’s appearance, you can add **CSS** and **JavaScript**. Here’s an example of improving the form with **Bootstrap**:

```html
<form action="/sublist" method="POST" class="form-inline">
    <input type="text" name="domain" class="form-control" placeholder="Enter domain" required>
    <button type="submit" class="btn btn-primary">Run Sublist3r</button>
</form>
```

You can further style the result with tables or other formatting.

## Step 4: Limitations and Free Tier Considerations

Render's free tier has some limitations, such as:
- Limited CPU and memory.
- Maximum of **100 hours per month** of usage.
  
If your app grows or requires more resources, you can upgrade to a paid plan.

## Summary

1. Set up Sublist3r and wrap it in a Flask web application.
2. Deploy the app to Render using their free tier.
3. Create a simple web interface for non-technical users to interact with Sublist3r.
4. Improve the UI with CSS and make the tool more user-friendly.

---

### Notes:

- You can further customize this app to enhance the user experience, such as providing downloadable CSV results or displaying the subdomains in a table.
- Consider securing your app with basic authentication or rate limiting if it becomes publicly accessible.
