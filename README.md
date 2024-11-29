# Case-Study-Portal
Searching For AI (SFAI) provides case studies on financial institutions live with an artificial intelligence app.

The SFAI research portal is part of the overall SFAI AI Decision Intelligence Platform (AIDIP).  The AIDIP will be delivered in phases as follows:

• Phase 1 – Research Portal.  Ability to view case studies in aggregate and individually and perform research on the CS archive.
• Phase 2 – AI Platform Builder.  Ability to put together the AI platform and build out the various projects and stages of an organizations AI roadmap.
• Phase 3 – AI ROI Calculator. Ability to track ROI for all AI projects and report on the success of the projects.  Consolidated AI metrics.
• Phase 4 – Vendor Monitoring.  Ability to monitor the various AI vendors who are used to deliver AI features throughout a financial institution.

This project is for PHASE 1.  For phase 1, SFAI will build an online portal for end-users to access the case studies and perform general research and run basic reports.

The phase 1 portal will provide two (2) primary ways of navigating the case study database; a visual user interface allowing screen engagement and a chatbot user interface allowing an end-user to speak to the SFAI portal to perform research.

The portal will be secure. Customer end-users will be required to log in to gain access to the portal.  Admin end-users will be able to log in and perform end-user maintenance tasks.

The portal will allow an end-user to set up alerts to see case studies as they are entered into the SFAI case study archives.

Currently using Zoho Creator for housing the SFAI database and considering Zoho Portal app to deliver the above project.  

Willing to consider other toolkits and low-code environments if easier to implement this phase of the project.
===============
To build Phase 1 of the SFAI Research Portal, we can break down the system into the following main components:

    User Authentication (Login/Access Control): Users should log in to access the portal. Admin users should have the ability to perform administrative tasks such as managing case studies and end-user accounts.

    Case Study Database: The ability to view and perform research on case studies, which are stored in the SFAI database.

    UI Navigation: Two modes of navigation — a visual user interface (UI) and a chatbot interface for searching the case studies.

    Alert System: End-users should be able to set up alerts to receive notifications when new case studies are added to the archive.

    Data Security: The portal should be secure, ensuring that only authorized users can access case studies and other sensitive data.

Here's an example Python code to create a web portal for Phase 1 using Flask (Python-based web framework) and SQLAlchemy (for database management). You can adapt this to Zoho Creator, Zoho Portal, or other toolkits later, but this code will give you a foundation for setting up the portal.
Step 1: Install Required Libraries

pip install flask flask_sqlalchemy flask_login

Step 2: Flask App Setup (app.py)

import os
from flask import Flask, render_template, redirect, url_for, request, flash, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_login import UserMixin, LoginManager, login_user, login_required, logout_user, current_user

# Initialize the app
app = Flask(__name__)
app.secret_key = os.urandom(24)  # Secret key for sessions
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///sfai.db'  # Local database
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Initialize database
db = SQLAlchemy(app)

# Initialize login manager
login_manager = LoginManager(app)
login_manager.login_view = 'login'

# Database Models
class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(150), unique=True, nullable=False)
    password = db.Column(db.String(150), nullable=False)
    is_admin = db.Column(db.Boolean, default=False)

class CaseStudy(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(150), nullable=False)
    content = db.Column(db.Text, nullable=False)
    date_added = db.Column(db.DateTime, default=db.func.current_timestamp())

class Alert(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    case_study_id = db.Column(db.Integer, db.ForeignKey('case_study.id'), nullable=False)

# Load user function for Flask Login
@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

# Routes

@app.route('/')
@login_required
def index():
    # Display all case studies
    case_studies = CaseStudy.query.all()
    return render_template('index.html', case_studies=case_studies)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = User.query.filter_by(username=username).first()
        
        if user and user.password == password:  # In practice, hash passwords for security
            login_user(user)
            return redirect(url_for('index'))
        else:
            flash("Invalid credentials!", "danger")
            
    return render_template('login.html')

@app.route('/logout')
@login_required
def logout():
    logout_user()
    return redirect(url_for('login'))

@app.route('/case_study/<int:id>')
@login_required
def case_study(id):
    case_study = CaseStudy.query.get(id)
    return render_template('case_study.html', case_study=case_study)

@app.route('/add_case_study', methods=['GET', 'POST'])
@login_required
def add_case_study():
    if not current_user.is_admin:
        flash("You do not have permission to add case studies.", "danger")
        return redirect(url_for('index'))

    if request.method == 'POST':
        title = request.form['title']
        content = request.form['content']
        new_case_study = CaseStudy(title=title, content=content)
        db.session.add(new_case_study)
        db.session.commit()
        flash("Case Study added successfully!", "success")
        return redirect(url_for('index'))

    return render_template('add_case_study.html')

@app.route('/set_alert/<int:case_study_id>', methods=['POST'])
@login_required
def set_alert(case_study_id):
    alert = Alert(user_id=current_user.id, case_study_id=case_study_id)
    db.session.add(alert)
    db.session.commit()
    flash("Alert set for this case study!", "success")
    return redirect(url_for('case_study', id=case_study_id))

@app.route('/chatbot', methods=['POST'])
@login_required
def chatbot():
    user_input = request.form['user_input']
    # Integrate an AI API for the chatbot to respond here (e.g., OpenAI GPT-3)
    response = generate_chatbot_response(user_input)
    return jsonify({'response': response})

def generate_chatbot_response(user_input):
    # Placeholder for chatbot logic (could integrate OpenAI GPT or another service)
    return "This is a placeholder response to: " + user_input

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)

Step 3: HTML Templates
index.html (Display case studies)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SFAI Research Portal</title>
</head>
<body>
    <h1>Welcome to the SFAI Research Portal</h1>
    <p>Logged in as {{ current_user.username }}</p>
    <a href="{{ url_for('logout') }}">Logout</a>
    <h2>Case Studies</h2>
    <ul>
        {% for case_study in case_studies %}
            <li>
                <a href="{{ url_for('case_study', id=case_study.id) }}">{{ case_study.title }}</a>
            </li>
        {% endfor %}
    </ul>
    <a href="{{ url_for('add_case_study') }}">Add a New Case Study</a>
</body>
</html>

login.html (Login page)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
</head>
<body>
    <h2>Login to SFAI Portal</h2>
    <form method="POST">
        <label for="username">Username:</label>
        <input type="text" name="username" id="username" required>
        <label for="password">Password:</label>
        <input type="password" name="password" id="password" required>
        <button type="submit">Login</button>
    </form>
</body>
</html>

case_study.html (View case study)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ case_study.title }}</title>
</head>
<body>
    <h1>{{ case_study.title }}</h1>
    <p>{{ case_study.content }}</p>
    <form method="POST" action="{{ url_for('set_alert', case_study_id=case_study.id) }}">
        <button type="submit">Set Alert for New Case Studies</button>
    </form>
</body>
</html>

Step 4: Run the Flask App

python app.py

Now you can access the portal at http://localhost:5000/. You can:

    Log in and view the case studies.
    Set alerts for case studies.
    Use the chatbot (though the chatbot's functionality needs integration with an AI model like GPT-3 or another service).

Next Steps:

    For Phase 1: Integrate an AI model for the chatbot using OpenAI or another NLP model.
    Consider integrating Zoho Creator or Portal if needed, which would provide built-in solutions for authentication, database management, and UI design.

This approach provides a solid foundation for SFAI's Phase 1 Research Portal, and you can expand it further based on additional requirements.
