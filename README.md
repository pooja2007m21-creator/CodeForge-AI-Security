# CoreForge-AI-Security
AI powered privacy- first cybersecurity system
# CodeForge AI Security Prototype
# Single-file demo: AI-based Phishing URL Detector
# Works locally, no cloud, privacy-preserving

from flask import Flask, request, render_template_string
import pandas as pd
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

app = Flask(__name__)

# -------------------------
# 1️⃣ Sample dataset
# -------------------------
# Columns: length, dots, has_at, https, label (0 = safe, 1 = phishing)
data = pd.DataFrame([
    [20, 2, 0, 1, 0],
    [55, 5, 1, 0, 1],
    [30, 3, 0, 1, 0],
    [60, 6, 1, 0, 1],
    [25, 2, 0, 1, 0],
    [70, 7, 1, 0, 1]
], columns=["length", "dots", "has_at", "https", "label"])

X = data.drop("label", axis=1)
y = data["label"]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# -------------------------
# 2️⃣ Train AI model
# -------------------------
model = LogisticRegression()
model.fit(X_train, y_train)

# -------------------------
# 3️⃣ Feature extraction
# -------------------------
def extract_features(url):
    features = []
    features.append(len(url))              # URL length
    features.append(url.count('.'))        # number of dots
    features.append(1 if '@' in url else 0)  # @ symbol
    features.append(1 if 'https' in url else 0) # https presence
    return [features]

# -------------------------
# 4️⃣ Web app (Flask)
# -------------------------
html_template = """
<!DOCTYPE html>
<html>
<head>
    <title>CodeForge AI Security</title>
</head>
<body>
    <h2>CodeForge AI Phishing URL Detector</h2>
    <form method="POST">
        <input type="text" name="url" placeholder="Enter URL" required style="width:300px">
        <button type="submit">Analyze</button>
    </form>
    <h3>{{ prediction_text }}</h3>
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def index():
    prediction_text = ""
    if request.method == "POST":
        url = request.form["url"]
        features = extract_features(url)
        prediction = model.predict(features)
        if prediction[0] == 1:
            prediction_text = "⚠️ Warning: This URL is likely PHISHING!"
        else:
            prediction_text = "✅ Safe: This URL appears legitimate."
    return render_template_string(html_template, prediction_text=prediction_text)

# -------------------------
# 5️⃣ Run the app
# -------------------------
if __name__ == "__main__":
    app.run(debug=True)
