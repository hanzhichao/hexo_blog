---
layout: post
title: Flask+Wrk+Nmon+pyNmonAnalyzer开发性能测试平台
comments: true
toc: true
date: 2018-02-01 19:41:21
updated: 2018-02-01 19:41:21
categories:
tags: Flask
---
# perf.py
# coding:utf-8
from flask import Flask, render_template, request, redirect, url_for
import subprocess
from time import sleep
app = Flask(__name__)
@ app.route("/")
def index():
    return render_template('index.html')
@ app.route("/wrk", methods=["GET", "POST"])
def perf():
    global result
    result = ''
    if request.method == "POST":
        method = request.form["methods"]
        connection = request.form["connection"]
        url = request.form["url"]
        c = request.form["c"]
        t = request.form["t"]
        d = request.form["d"]
        subprocess.Popen("nmon -s 1 -c %s -F report.nmon" % str(int(d)+2),shell=True, stdout=subprocess.PIPE)
        subprocess.Popen("wrk -c %s -t %s -d %s -H '%s'--latency %s://%s > wrk.log" % (c, t, d, connection, method, url),
                         shell=True, stdout=subprocess.PIPE)
        sleep(int(d)+2)
    with open("wrk.log",'r') as f:
        result = f.read().replace('\n','<br/>')
    subprocess.Popen("pyNmonAnalyzer -b -t static -x -o static/report -i report.nmon", shell=True,stdout=subprocess.PIPE)
    
    return render_template("wrk.html", result=result)
@ app.route("/nmon")
def nmon():
    return render_template("nmon.html")
@ app.route("/report")
def report():
    return render_template("report.html")
if __name__ == '__main__':
    app.run("0.0.0.0")



# index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Performance Test</title>
    <link rel="stylesheet" type="text/css" href="../static/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <nav class="navbar navbar-default">
        <div class="navbar-heading">
            <div class="navbar-brand"><a href="#">PerfTestFlat</a></div>
        </div>
        <ul class="nav navbar-nav">
            <li class='active'><a href="./">Home</a></li>
            <li><a href="./wrk">Wrk</a></li>
            <li><a href="./nmon">Nmon</a></li>
            <li><a href="./report">Report</a></li>
        </ul>
    </nav>
    <div class="page-header"><h1>Welcome</h1></div>
    </div>
</div>
</body>
</html>

#nmon.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nmon</title>
    <link rel="stylesheet" type="text/css" href="../static/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <nav class="navbar navbar-default">
        <div class="navbar-heading">
            <div class="navbar-brand"><a href="#">PerfTestFlat</a></div>
        </div>
        <ul class="nav navbar-nav">
            <li><a href="./">Home</a></li>
            <li><a href="./wrk">Wrk</a></li>
            <li class="active"><a href="./Nmon">Nmon</a></li>
            <li><a href="./report">Report</a></li>
        </ul>
    </nav>
</div>
</body>
</html>

# report.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Report</title>
    <link rel="stylesheet" type="text/css" href="../static/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <nav class="navbar navbar-default">
        <div class="navbar-heading">
            <div class="navbar-brand"><a href="#">PerfTestFlat</a></div>
        </div>
        <ul class="nav navbar-nav">
            <li><a href="./">Home</a></li>
            <li><a href="./wrk">Wrk</a></li>
            <li><a href="./nmon">Nmon</a></li>
            <li class="active"><a href="./report">Report</a></li>
        </ul>
    </nav>
    <div class="page-header"><h1>Report</h1></div>
    <div class="panel"><img src="&#123;&#123; url_for('static', filename='report/img/CPU_vs_Time.png') &#125;&#125;"></div>
    <div class="panel"><img src="&#123;&#123; url_for('static', filename='report/img/Disk_Busy_vs_Time.png') &#125;&#125;"></div>
    <div class="panel"><img src="&#123;&#123; url_for('static', filename='report/img/Memory_vs_Time.png') &#125;&#125;"></div>
</div>
</body>
</html>

#result.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Result</title>
    <link rel="stylesheet" type="text/css" href="../static/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <nav class="navbar navbar-default">
        <div class="navbar-heading">
            <div class="navbar-brand"><a href="#">PerfTestFlat</a></div>
        </div>
        <ul class="nav navbar-nav">
            <li class="active"><a href="#">Home</a></li>
            <li><a href="#">Test</a></li>
            <li><a href="#">Report</a></li>
            <li><a href="#">Nmon</a></li>
        </ul>
    </nav>
    <div class="page-header"><h1>Result</h1></div>
</div>
</body>
</html>
#wrk.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Performance Test</title>
    <link rel="stylesheet" type="text/css" href="../static/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <nav class="navbar navbar-default">
        <div class="navbar-heading">
            <div class="navbar-brand"><a href="#">PerfTestFlat</a></div>
        </div>
        <ul class="nav navbar-nav">
            <li><a href="./">Home</a></li>
            <li class="active"><a href="./wrk">Wrk</a></li>
            <li><a href="./nmon">Nmon</a></li>
            <li><a href="./report">Report</a></li>
        </ul>
    </nav>
    <div class="page-header"><h1>Performance Test</h1></div>
    <div class="col-md-4 col-sm-6" style="padding-left: 0px;">
        <div class="panel panel-primary">
            <div class="panel-heading">
                <div class="panel-title">Wrk</div>
            </div>
            <div class="panel-body">
                <form class="form" action="" method="post">
                    <div class="form-group">
                        <label>Method:</label>
                        <select name="methods" class="form-control">
                            <option label="http" value="http">http</option>
                            <option label="https" value="https">https</option>
                        </select></div>
                    <div class="form-group">
                        <label>Connection:</label>
                        <select name="connection" class="form-control">
                            <option value="keep-alive">keep-alive</option>
                            <option value="close">close</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <div class="input-group">
                            <span class="input-group-addon">Url:</span>
                            <input type="text" class="form-control" name="url">
                        </div>
                    </div>
                    <div class="form-group">
                        <div class="input-group">
                            <span class="input-group-addon">C:</span>
                            <input type="text" class="form-control" name="c">
                        </div>
                    </div>
                    <div class="form-group">
                        <div class="input-group">
                            <span class="input-group-addon">T:</span>
                            <input type="text" class="form-control" name="t">
                        </div>
                    </div>
                    <div class="form-group">
                        <div class="input-group">
                            <span class="input-group-addon">D:</span>
                            <input type="text" class="form-control" name="d">
                        </div>
                    </div>
                    <div class="form-group">
                        <button type="submit" class="btn btn-primary">submit</button>
                    </div>
                </form>
            </div>
            <div class="panel-footer">by Hanzhichao</div>
        </div>
    </div>
    <div class="col-md-8 col-sm-6" style="padding-right: 0px">
        <div class="panel panel-primary">
            <div class="panel-heading"><div class="panel-title">Wrk Result</div></div>
            <div class="panel-body">&#123;&#123; result|safe  &#125;&#125;</div>
            <div class="panel-footer">by Hanzhichao</div>
        </div>
    </div>
</div>
</body>
</html>

