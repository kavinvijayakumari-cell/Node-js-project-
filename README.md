from flask import Flask, render_template, request, redirect, url_for, send_from_directory
import os

app = Flask(__name__)

UPLOAD_FOLDER = 'static/uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

# 1️⃣ Home Page
@app.route('/')
def home():
    return render_template('home.html')

# 2️⃣ Upload Page
@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if request.method == 'POST':
        if 'file' not in request.files:
            return 'No file part!'
        file = request.files['file']
        if file.filename == '':
            return 'No selected file!'
        filepath = os.path.join(app.config['UPLOAD_FOLDER'], file.filename)
        file.save(filepath)
        return redirect(url_for('files'))
    return render_template('upload.html')

# 3️⃣ Files List Page
@app.route('/files')
def files():
    file_list = os.listdir(UPLOAD_FOLDER)
    return render_template('files.html', files=file_list)

# 4️⃣ View / Download Page
@app.route('/view/<filename>')
def view_file(filename):
    return render_template('view.html', filename=filename)

@app.route('/download/<filename>')
def download_file(filename):
    return send_from_directory(UPLOAD_FOLDER, filename, as_attachment=True)

# 5️⃣ Delete Confirmation Page
@app.route('/delete/<filename>', methods=['GET', 'POST'])
def delete_file(filename):
    filepath = os.path.join(UPLOAD_FOLDER, filename)
    if request.method == 'POST':
        if os.path.exists(filepath):
            os.remove(filepath)
        return redirect(url_for('files'))
    return render_template('delete.html', filename=filename)

if __name__ == '__main__':
    app.run(debug=True)
________________________________________
🧩 templates/base.html
<!DOCTYPE html>
<html>
<head>
  <title>File Upload Manager</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="bg-light">
<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">File Manager</a>
    <div>
      <a class="nav-link d-inline text-light" href="/upload">Upload</a>
      <a class="nav-link d-inline text-light" href="/files">Files</a>
    </div>
  </div>
</nav>
<div class="container">
  {% block content %}{% endblock %}
</div>
</body>
</html>
________________________________________
🧩 templates/home.html
{% extends "base.html" %}
{% block content %}
<div class="text-center mt-5">
  <h1>Welcome to File Upload Manager</h1>
  <p class="lead">Upload, view, and manage your files easily.</p>
  <a href="/upload" class="btn btn-primary">Go to Upload Page</a>
</div>
{% endblock %}
________________________________________
🧩 templates/upload.html
{% extends "base.html" %}
{% block content %}
<h2>Upload a New File</h2>
<form action="/upload" method="post" enctype="multipart/form-data" class="mt-3">
  <input type="file" name="file" class="form-control mb-3" required>
  <button class="btn btn-success">Upload</button>
</form>
{% endblock %}
________________________________________
🧩 templates/files.html
{% extends "base.html" %}
{% block content %}
<h2>Uploaded Files</h2>
{% if files %}
<ul class="list-group mt-3">
  {% for f in files %}
  <li class="list-group-item d-flex justify-content-between align-items-center">
    {{ f }}
    <span>
      <a href="/view/{{ f }}" class="btn btn-info btn-sm">View</a>
      <a href="/download/{{ f }}" class="btn btn-secondary btn-sm">Download</a>
      <a href="/delete/{{ f }}" class="btn btn-danger btn-sm">Delete</a>
    </span>
  </li>
  {% endfor %}
</ul>
{% else %}
<p>No files uploaded yet.</p>
{% endif %}
{% endblock %}
________________________________________
🧩 templates/view.html
{% extends "base.html" %}
{% block content %}
<h2>View File: {{ filename }}</h2>
<div class="mt-3">
  {% if filename.endswith('.jpg') or filename.endswith('.png') or filename.endswith('.gif') %}
    <img src="/static/uploads/{{ filename }}" class="img-fluid" alt="{{ filename }}">
  {% else %}
    <p>Preview not available for this file type.</p>
  {% endif %}
</div>
<a href="/download/{{ filename }}" class="btn btn-primary mt-3">Download</a>
<a href="/files" class="btn btn-secondary mt-3">Back to Files</a>
{% endblock %}
________________________________________
🧩 templates/delete.html
{% extends "base.html" %}
{% block content %}
<h2>Delete File</h2>
<p>Are you sure you want to delete <strong>{{ filename }}</strong>?</p>
<form method="post">
  <button type="submit" class="btn btn-danger">Yes, Delete</button>
  <a href="/files" class="btn btn-secondary">Cancel</a>
</form>
{% endblock %}




