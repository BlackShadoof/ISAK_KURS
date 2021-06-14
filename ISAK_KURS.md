
# Курсовая работа по дисциплине информационные системы аэрокосмических комплексов.
## Журавлев А.А М3О312Б18

# 1) Установим репозиторий EPEL
sudo yum install epel-release
# 2) Установим диспетчер пакетов Python
sudo yum install python-pip python-devel gcc nginx  
# 3)Создаем виртуальное окружение
    1)sudo pip install virtualenv  
    2)virtualenv projectvenv 
    3)source projectvenv/bin/activate 
# 4) Cоздаем приложение FLASK
    1)pip install uwsgi flask  
    2)vi ~/project/application.py
    3)
```python
import os, io
import base64
from PIL import Image
import cv2
import numpy as np
from flask import Flask, render_template, request, abort
from werkzeug.utils import secure_filename
import imghdr
from image_processing.ndvi import get_ndvi, apply_gradient

if (__name__=='__main__'):
	app = Flask(__name__)
	app.config['TEMPLATES_AUTO_RELOAD'] = True
	app.config['UPLOAD_EXTENSIONS'] = ['.tif']
	app.config['UPLOAD_PATH'] = 'static/process_images'

@app.route('/')
def index():
	return render_template('index.html')

def validate_image(stream):
	header = stream.read(512)
	stream.seek(0)
	format = imghdr.what(None, header)
	if not format:
		return None
	return format

def get_uri(img):
	rawBytes = io.BytesIO()
	img.save(rawBytes, "JPEG")
	rawBytes.seek(0)
	img_base64 = base64.b64encode(rawBytes.getvalue()).decode('ascii')
	mime = "image/jpeg"
	return "data:%s;base64,%s"%(mime, img_base64)

@app.route('/', methods=['POST'])
def upload_files():
	files = request.files.getlist('file')
	filenames = []
	if (request.files['file'].filename == ''): # Нет файлов
		return ('', 204)
	for uploaded_file in files:
		filename = secure_filename(uploaded_file.filename)
		if filename != '':
			filenames.append(filename)
			file_ext = os.path.splitext(filename)[1]
			if file_ext not in app.config['UPLOAD_EXTENSIONS'] or \
					file_ext != validate_image(uploaded_file.stream):
				abort(400)
```
# 5)Создание точки входа WSGI.
    1)vi ~/project/wsgi.py  
    2)
    
```from project import app  
if __name__ == "__main__":  
  app.run()  
```
# 6)Настройка конфигурации uWSGI.
    1)vi ~/project/project.ini 
```
2)
[uwsgi]  
module = wsgi  
master = true  
processes = 3  
socket = project.sock  
chmod-socket = 660  
vacuum = true
```
# 7) Создание файла модуля systemd.
    1)sudo vi /etc/systemd/system/project.service 
```
2)
[Unit]  
Description=uWSGI for project  
After=network.target  
[Service]  
User=vladislav  
Group=nginx  
WorkingDirectory=/home/vladislav/project  
Environment="PATH=/home/vladislav/project/projectvenv/bin"  
ExecStart=/home/vladislav/project/projectvenv/bin/uwsgi --ini project.ini  
[Install]  
WantedBy=multi-user.target 
```
 
# 8)Результат.
1)![2021-06-13_18-05-05](https://user-images.githubusercontent.com/67752728/121814936-56d8ec80-cc7c-11eb-8996-30465a6879b5.png)
2)![2021-06-13_18-08-46](https://user-images.githubusercontent.com/67752728/121814942-648e7200-cc7c-11eb-9b5b-aed91196dbc7.png)

