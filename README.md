# project
Creative an interactive AI platform for teacher and student

from flask import Flask, render_template, redirect, url_for, request, send_from_directory
import os
from werkzeug.utils import secure_filename
import uuid

import model

app = Flask(__name__)

uploads_dir = os.path.join(app.instance_path, 'uploads')
os.makedirs(uploads_dir, exist_ok=True)

csv_file = os.path.join(app.instance_path, 'uploads', 'Questions.csv')

@app.route('/')
@app.route('/home')
def home_page():
    return render_template('home.html')

@app.route('/uploads/<path:path>')
def uploads(path):
    return send_from_directory(uploads_dir, path)

@app.route('/student', methods=["GET", "POST"])
def student_page():
    data = {'answer_audio': 'error'}
    if request.method == 'POST':
        audio_file = request.files['questionAudio']
        # query_id = str(uuid.uuid4())
        query_id = 'temp'
        audio_file_path = os.path.join(uploads_dir, query_id + '.mp3')
        audio_file.save(audio_file_path)
        
        # process the data with the audio file as [audio_file_path]
        question_id, question = model.process(audio_file_path)
        answer_audio_file_remote_path = f'/uploads/{question_id}.mp3'
        data['answer_audio'] = answer_audio_file_remote_path
        data['question'] = question
    return render_template('student.html', data=data)

@app.route('/teacher', methods=["POST","GET"])
def teacher_page():
    data = {'status': 'unknown'}
    if request.method == "POST":
        question_id = str(uuid.uuid4())
        audio_file = request.files['answerAudio']
        question = request.form['question']
        # print(f"{question_id},{question},{audio_file_path}")
        print(f'csv_file : {csv_file}')
        with open(csv_file, 'a') as f:
            f.write(f'{question_id},{question}\n')
        audio_file_path = os.path.join(uploads_dir, question_id + '.mp3')
        audio_file.save(audio_file_path)
        data['status'] = 'success'
    return render_template('teacher.html', data=data)

if __name__ == "__main__":
    app.run(debug=True)
