**This is simple mini cloud web app **

Overview (request → response)

Client requests route (GET/POST) → Flask receives HTTP request.
Flask matches route to view function (app.route decorators).
For protected routes, Flask-Login checks session cookie and current_user.
For POST forms, CSRFProtect validates CSRF token from form data.
View performs actions (DB via SQLAlchemy models, filesystem ops).
Templates render HTML (Jinja2) and include links to static assets (CSS) via url_for('static', ...).
Responses returned to client; file downloads use send_from_directory with ownership check.
Auth flow

Register: POST username/password → User model created → password hashed (Werkzeug) → db.session.commit().
Login: POST credentials → verify password → login_user(user) sets session cookie.
Logout: logout_user() clears session.
File flow (upload)

User submits multipart/form POST to /upload.
CSRF token validated.
allowed_file() checks extension; secure_filename() sanitizes name.
Server saves to uploads/<user_id>/<secure_filename>.
File size read (os.path.getsize); metadata inserted into File model (file_id, user_id, filename, filepath, size, upload_date).
Response redirects to dashboard.
File flow (download/delete)

/download/<file_id>: query File, check f.user_id == current_user.id, send_from_directory(directory, filename, as_attachment=True).
/delete/<file_id>: POST only, ownership check, remove file from FS, delete DB row.
Security measures

Passwords hashed via generate_password_hash / check_password_hash.
Session-based auth via Flask-Login.
CSRF protection via Flask-WTF CSRFProtect.
secure_filename() and allowed extensions + MAX_CONTENT_LENGTH limit.
Files stored outside static folder (uploads/) and per-user directories.
Ownership checks (403 abort) on download/delete.
CSRF fix (templates) — wrap token in hidden input

<form method="post">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
    <!-- ... -->
</form>
<form action="{{ url_for('upload') }}" method="post" enctype="multipart/form-data">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
    <input type="file" name="file" required>
    <input type="submit" value="Upload">
</form>

<form style="display:inline" action="{{ url_for('delete', file_id=f.id) }}" method="post">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
    <button type="submit">Delete</button>
</form>

CSS not applied — quick checks

Open browser devtools Network → request: GET style.css → expect 200.
Confirm base template link: <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">.
Confirm app.static_folder points to your static folder (you set it when creating Flask).
Directly open: http://127.0.0.1:5000/static/style.css — if 404, verify file path d:\CODE\WebDev\mini-cloud\static\style.css.
Clear cache / hard reload.
Debug commands

Check CSS URL (PowerShell):
curl http://127.0.0.1:5000/static/style.css -UseBasicParsing
Tail Flask logs for static 404s, CSRF errors.
If you want, I can patch the three templates now to add the hidden CSRF inputs and confirm base.html has correct <link>.

