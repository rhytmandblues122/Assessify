from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)

quiz_data = [
    {
        'question': 'What is the capital of France?',
        'options': ['Paris', 'Berlin', 'London', 'Madrid'],
        'correct_answer': 'Paris'
    },
    {
        'question': 'Which programming language is this quiz written in?',
        'options': ['Python', 'Java', 'C++', 'JavaScript'],
        'correct_answer': 'Python'
    },
    # Add more questions as needed
]
user_score = 0

@app.route('/')
def index():
    return render_template('index.html', quiz_data=quiz_data)

@app.route('/submit', methods=['POST'])
def submit():
    global user_score
    user_answers = request.form.to_dict()
    
    for question, answer in user_answers.items():
        for q in quiz_data:
            if q['question'] == question and q['correct_answer'] == answer:
                user_score += 1

    return redirect(url_for('results'))

@app.route('/results')
def results():
    global user_score
    score = user_score
    user_score = 0  # Reset the user's score for the next attempt
    return render_template('results.html', score=score)

if __name__ == '__main__':
    app.run(debug=True)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Flask Quiz App{% endblock %}</title>
</head>
<body>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>

{% extends 'base.html' %}

{% block title %}Quiz{% endblock %}

{% block content %}
    <h1>Quiz</h1>
    <form action="{{ url_for('submit') }}" method="post">
        {% for question in quiz_data %}
            <fieldset>
                <legend>{{ question['question'] }}</legend>
                {% for option in question['options'] %}
                    <label>
                        <input type="radio" name="{{ question['question'] }}" value="{{ option }}">
                        {{ option }}
                    </label><br>
                {% endfor %}
            </fieldset>
        {% endfor %}
        <input type="submit" value="Submit">
    </form>
{% endblock %}

{% extends 'base.html' %}

{% block title %}Results{% endblock %}

{% block content %}
    <h1>Results</h1>
    <p>Your score is: {{ score }} out of {{ quiz_data|length }}</p>
{% endblock %}

for pdf
from flask import Flask, render_template, request, redirect, url_for
from io import BytesIO
from pdfminer.high_level import extract_text
import re

app = Flask(__name__)

def clean_text(text):
    # remove extra white spaces
    text = re.sub(' +', ' ', text)
    # remove extra newlines
    text = re.sub('\n+', '\n', text)
    return text

@app.route('/', methods=['GET', 'POST'])
def upload_pdf():
    if request.method == 'POST':
        if 'pdf_file' not in request.files:
            return redirect(request.url)
        pdf_file = request.files['pdf_file']
        if pdf_file.filename == '':
            return redirect(request.url)
        if pdf_file:
            pdf_text = extract_text(BytesIO(pdf_file.read()))
            cleaned_text = clean_text(pdf_text)
            return render_template('pdf_content.html', content=cleaned_text)
    return render_template('upload_pdf.html')

if __name__ == '__main__':
    app.run(debug=True)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Upload PDF</title>
</head>
<body>
    <h1>Upload PDF</h1>
    <form action="{{ url_for('upload_pdf') }}" method="post" enctype="multipart/form-data">
        <label for="pdf_file">PDF File:</label>
        <input type="file" id="pdf_file" name="pdf_file" required>
        <input type="submit" value="Upload">
    </form>
</body>
</html>

for camera access 
from flask import Flask, render_template, Response
import cv2

app = Flask(__name__)

def generate_frames():
    cap = cv2.VideoCapture(0)
    while True:
        success, frame = cap.read()
        if not success:
            break
        else:
            ret, buffer = cv2.imencode('.jpg', frame)
            frame = buffer.tobytes()
            yield (b'--frame\r\n'
                   b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/video_feed')
def video_feed():
    return Response(generate_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

if __name__ == '__main__':
    app.run(debug=True)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Online Quiz</title>
</head>
<body>
    <h1>Welcome to Online Quiz</h1>
    <form action="{{ url_for('home') }}" method="post">
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required>
        <label for="quiz_id">Quiz ID:</label>
        <input type="number" id="quiz_id" name="quiz_id" required>
        <input type="submit" value="Start Quiz">
    </form>
</body>
</html>
