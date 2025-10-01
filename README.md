import os
from flask import Flask, request, redirect, url_for, render_template_string, send_from_directory
from werkzeug.utils import secure_filename

app = Flask(__name__)

UPLOAD_FOLDER = "uploads"
ALLOWED_EXTENSIONS = {"png", "jpg", "jpeg", "gif"}
app.config["UPLOAD_FOLDER"] = UPLOAD_FOLDER

if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Image Gallery</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background: #f8f9fa; padding: 30px; }
        .gallery { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; }
        .gallery img { width: 100%; height: auto; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .upload-box { max-width: 600px; margin: auto; }
    </style>
</head>
<body>
    <div class="upload-box card shadow p-4 mb-5">
        <h2 class="text-center mb-3">📸 Image Gallery</h2>
        <form method="post" enctype="multipart/form-data">
            <div class="input-group">
                <input type="file" class="form-control" name="file" accept="image/*" required>
                <button type="submit" class="btn btn-primary">Upload</button>
            </div>
        </form>
    </div>
    <div class="gallery">
        {% for image in images %}
            <div>
                <img src="{{ url_for('uploaded_file', filename=image) }}" alt="Image">
            </div>
        {% endfor %}
    </div>
</body>
</html>
"""

def allowed_file(filename):
    return "." in filename and filename.rsplit(".", 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        if "file" not in request.files:
            return redirect(request.url)
        file = request.files["file"]
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config["UPLOAD_FOLDER"], filename))
            return redirect(url_for("index"))
    images = os.listdir(app.config["UPLOAD_FOLDER"])
    return render_template_string(HTML_TEMPLATE, images=images)

@app.route("/uploads/<filename>")
def uploaded_file(filename):
    return send_from_directory(app.config["UPLOAD_FOLDER"], filename)

if __name__ == "__main__":
    app.run(debug=True)
