---
title: FastAPIでファイルアップロード
date: 2020-09-29
categories: ["Python"]
tags: ["fastapi","python"]
slug: "fileuploadwithfastapi"
adsenseTop: true
adsenseBottom: true
---

FastAPIでアップロードされたファイルを保存する処理をFlaskを参考に作成。

## Flaskでは

Flaskの[ファイルアップロード](https://flask.palletsprojects.com/en/1.1.x/patterns/fileuploads/)の項から引用

```py
import os
from flask import Flask, flash, request, redirect, url_for
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = '/path/to/the/uploads'
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit an empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return redirect(url_for('uploaded_file',
                                    filename=filename))
```

Flaskのドキュメントでは関数allowed_fileでALLOWED_EXTENSIONSの許可されたファイルタイプのみuploadsディレクトリにコピーするようになってます。  
これをFastAPIでも使うことにする。

## FastAPIでは

FastAPIでアップロードファイルを受け取るにはpython-multipartが必要なので、  
`pip install python-multipart`でインストールする。

```py
from fastapi import FastAPI, File, UploadFile
import os
import shutil

app = FastAPI()

ALLOWED_EXTENSIONS = set(['csv','jpg'])
UPLOAD_FOLDER = './uploads'


def allowed_file(filename):
    return '.' in filename and \
        filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.post("/upload/")
async def upload(file: UploadFile = File(...)):
    if file and allowed_file(file.filename):
        filename = file.filename
        fileobj = file.file
        upload_dir = open(os.path.join(UPLOAD_FOLDER, filename),'wb+')
        shutil.copyfileobj(fileobj, upload_dir)
        upload_dir.close()
        return {アップロードされたファイル": filename}
    if file and not allowed_file(file.filename):
        return {"warning": "許可されたファイルタイプではありません"}
```

Flaskはwerkzeugのsave()でアップロードされたファイルをコピーしていますが、  
FastAPIはsave()がないので、pythonの[shutil.copyfileobj](https://docs.python.org/ja/3/library/shutil.html#shutil.copyfileobj)を利用します。
