---
layout: post
title: FIWARE Orion
author: Andy Robinson
---

FIWARE's Orion Context Broker has grown to be a key part of our architecture.  Since we're trying to encourage open source developers to collaborate with OpenTrack, I'm going to try and explain what it is, why we chose it, and what it can do.

In brief, it's a readymade ReST interface to MongoDB, with publish-and-subscribe features. It gives us a readymade data collection mechanism for scores and results  

## What is FIWARE?

FIWARE is a large European Union-funded project to help entrepreneurs build new solutions on a cloud computing platform.  This is the <a href="https://www.fiware.org/developers-entrepreneurs/">least confusing introduction</a> I have found:
<blockquote>
FIWARE provides an enhanced OpenStack-based cloud environment plus a rich set of open standard APIs that make it easier to connect to the Internet of Things, process and analyse Big data and real-time media or incorporate advanced features for user interaction.
</blockquote>

To some extent, it has invented its own jargon which is very confusing to outsiders.  Several years ago, they surveyed and tried to find technical gaps and to work out what sort of services would be needed to enable and accelerate economic growth.  Several years R&amp;D followed. 

## Who is using it?
In the last 2 years, hundreds of small businesses have received funding to launch innovative ideas, but only if they use some FIWARE technologies and/or Open Data.  We are one of them.  

## What is Orion?

Here's the intro paragraph from the documentation.  Once you get into the details, it's excellent - correct and comprehensive with good getting-started guides - but in my opinion, it gives an average developer NO CLUE what it's there for.  What do you think?

<blockquote>
Orion is a C++ implementation of the NGSI9/10 REST API binding developed as a part of the FIWARE platform.<br>

Orion Context Broker allows you to manage all the whole lifecycle of context information including updates, queries, registrations and subscriptions. Using the Orion Context Broker, you are able to register context elements and manage them through updates and queries. In addition, you can subscribe to context information so when some condition occurs (e.g. a context element has changed) you receive a notification. These usage scenarios and the Orion Context Broker features are described in this document.
</p>

</blockquote>

Disclaimer:  I still haven't a clue what "context information" means in this, er, context; and after a quick glance decided that life is too shart to learn about NGSI9/10. 

This was a slight problem entering the FIWARE programme, because one has to select from a large catalogue of technologies, and it could take a good hour or two of drilling into the docs before finding out what they actually did.  It's a bit different to how people promote SAAS solutions, where one usually starts with a good use case, screen shots if possible, and something to suck developers in.


## Plain English, please
Here's my try:
<blockquote>
Orion is designed for collection Internet-of-Things data on a large scale.  It also allows web applications to subscribe and be notified of changes.
</blockquote>
At a technical level, it's a web server implementing a ReST API in front of MongoDB.   You can make calls to create documents, search and query in various ways (including geographically), and update individual attributes of documents.   
It also implements a publish/subscribe model.  So you can register your app to be notified every time some document changes, and for the next 30 days, Orion will send a POST request to your server with the newest version of the document whenever it changes  

## How does this work for us?
Our use case is collecting real-time results and scores from sporting events. Potentially this could happen on a wide scale.  Sporting events might be happening all over Europe on a busy Saturday or Sunday morning, with teachers and volunteers uploading data almost simultaneously.   

We figured we could trust Orion, because it's already collecting huge amounts of data from temperature sensors in cities all over Spain, millions of cows and beehives, and other things that would never occur to you until you attend a FIWARE meetup.   It's written in C++, and behind the scenes MongoDb is probably one of the most scalable collection mechanisms.  

But the icing on the cake was the Publish/Subscribe mechanism, which had an unexpected social benefit for the project.

Chris Dack is a national-standard Shot Putter in the UK, who's also pretty handy with Javascript and Node.js. He was the first person outside the team to start contributing to the code, and he got the ball rolling with our <a href="/2016/04/08/live-field-events.html">field events display code</a>. 

Chris found Orion useful right away, because once we'd written some data capture apps and were uploading to Orion, he could access it right away, and work independently on his own node.js implementation. He just subscribed to the competition as we set it up, and thereafter, he had all the data he needed to play with. We didn't need to write docs, sit side-by-side for a few hours, grant passwords or anything.

We realised that if we stored the live data in Orion - and why not? It's all supposed to be Open Data - it would open the door to a lot of collaborators doing their own thing in terms of display.

Last month, a few of us were in Eindhoven, NL, recording the throws at a competition.  Each time our volunteer tapped a throw into a page on her phone, the throw distance was sent up to Orion.   Orion then relayed the state of that competition - a smallish chunk of JSON - onto our main server, but also onto various experimental ones as well.  I didn't even realise that Chris was around that day, but he had registered and was working on checking that his lightweight node server could handle the live feed correctly.

So, FIWARE's Orion allows an Englishman in South London to spy on fit Dutch girls as they throw things around, in real time. 

## What other choices were there?

We're funded by FIWARE so had to choose some of their technology to use.  Otherwise, we could have created our own athletics-specific APIs in our own Django server.  But I don't think the Pub/Sub mechanism would have occurred to us for quite a long time, and we'd have probably spent a long time debugging things.  And we would no doubt have resolved to produce documentation eventually, but failed to get around to it, or undergone a lot of churn, so it would have been a long time before external developers could join in.   Using a mature product helped - when you get past the first couple of pages, Orion is now very well-documented.   


## Using Orion from Python

We found that we 


## Using Orion from Javascript

Orion was designed for collecting Internet-of-Things data.  We started off with our Python application server posting information into Orion, but pretty quickly we realised that a mobile device or browser would be the ideal client.  Somebody will be at the side of a track, or in a field, tapping numbers into a web page.  We just want them to be uploaded and stored securely.  So we cut our own server out of the picture and can talk straight to Orion.

As a simple ReST API, it's possible to just point a browser at Orion and see the data within it (if you're running your own Orion).  It's no trouble to fetch an object, in a way very similar to Firebase.

But when it's time to update, at this point we ran into a small bug.  When browsers perform certain verbs (PUT and POST), they typically send a "pre-flight" OPTIONS request first to ask the server what it can respond to.  Orion doesn't yet handle this correctly.  

Our workaround was to produce a small Python WSGI server - probably about 40 lines of code - which we ran in front of Orion.  This implemented the missing methods, and at this point we could do everything from a browser that we could from a server.


## Security

Orion itself has no security. It's designed to be run behind a proxy server which provides security and access control.   Used within the FIWARE Lab, they run another service build on node.js, "PEP Proxy Wilma", in front of it.  Wilma checks that you have obtained a token from the FIWARE lab and put it in the headers.  At that point, it lets you through.  However, there is no fine-grained security in the FIWARE Lab instance; if you're careless with your object IDs, you can overwrite or delete some other user's data. Fair enough - it's just for training.

We're Python people, not Node people, so rather than learning about another third party product and how to configure it, we just wrote a few more lines of Python and extended our WSGI server.   We can serve up the tokens so that a browser can make a preliminary login call, get a token, and embed it on future requests.  In our context, it makes sense to control which sports officials can upload results, but perhaps to let anyone browse the results or register to be notified.  We think it's easier to write a little server class with wrappers around certain methods than to have a general-purpose configurable system and a declaration language.




# How we're running it

This is by no means the best way of doing things, but it didn't take long to set up and seems to work.

We set Orion up on our own Ubuntu servers using a Docker image.  We already use MongoDB, so we configured it to talk to a real, persistent MongoDB instance.  Docker was new to us, but works well.  The detup details



## Detailed setup notes
(from Robin Becker, Chief Engineer, working on Ubuntu 14.04)

    0) install mongodb

    1) Install docker


    2) create a working area to contain our own Docker file.

    rptlab@denali:~/devel/orion
    $ cat Dockerfile-80
    FROM fiware/orion
    WORKDIR /
    ENTRYPOINT ["/usr/bin/contextBroker","-fg", "-multiservice", "-corsOrigin", "__ALL"]
    EXPOSE 1026

    we added the -corsOrigin __ALL to the command line to allow cross site connection.


    3) build a docker image called rl/orion

    docker build -f  Dockerfile-80 -t rl/orion .

    our image requires no resources so . just refers to the path containg the Dockerfile (Dockerfile-80).

    4) Check that our image now exists with docker images
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ubuntu              trusty              90d5884b1ee0        8 weeks ago         188 MB
    fiware/orion        latest              fe1ffdce3f49        8 weeks ago         277.9 MB
    rl/orion-http       latest              b5b472b0c13d        8 weeks ago         277.9 MB

    the ubuntu & fiware/orion images are constructed by the build process which results in our rl/orion image.

    5) Run the docker image to create an executing server. We needed to use the local mongo so we have net == host

    docker run -d --net=host -p1026:1026 rl/orion-http orion-http

    6) test local connection on port 1026.
    rptlab@denali:~/devel/orion
    $ curl localhost:1026/version
    {
      "orion" : {
      "version" : "1.0.0-next",
      "uptime" : "13 d, 18 h, 46 m, 2 s",
      "git_hash" : "58a6b5e250cc5ced103f2ae7413a22ff73d602f6",
      "compile_time" : "Wed Apr 27 06:15:32 UTC 2016",
      "compiled_by" : "root",
      "compiled_in" : "838a42ae8431"
    }
    }

    7) Set up a backend proxy using traditional python virtualenv; our server which is uwsgi based is a simple wsgi app and is started thusly
            $ENVDIR/bin/uwsgi \
                --daemonize=$ENVDIR/logs/uwsgi.log \
                --home=$ENVDIR \
                --chdir=$ENVDIR \
                --pidfile=$ENVDIR/tmp/uwsgi.pid \
                --uwsgi-socket=0:13080 \
                --workers=20 \
                --master \
                --cheaper-algo=spare \
                --cheaper=2 \
                --cheaper-initial=2 \
                --cheaper-step=1 \
                --logdate='%Y/%m/%d %T' \
                --log-format-strftime \
                --disable-logging \
                --module=proxy.wsgi:application

    here --uwsgi-socket=0:13080 says we listen to port 13080 on all interfaces.

    The proxy code may be found at  https://hg.reportlab.com/hg/orion-proxy/.

    The latest proxy does allow for fiware style tokens to be used for validation our tokens are generated from a one time list of random letters. This 'feature' is turned on by having an environment variable NEED_TOKENS=1 at server start. The passwords are probably not that secure, but the is all internal behind the https transport layer.

    ============================nginx server
    8) We now  have our orion http server on port 13080. We connect it as a backend to an https/http nginx running on a different machine. So on that machine we have an nginx server running with a sitefile corresponding to our listening server the site file looks like this

    $ cat orion1.reportlab.com
    server {
            server_name orion1.reportlab.com;
            listen 80;
            charset utf-8;
            error_page 404 /404.html;
            root    /home/rptlab/website/orion1.reportlab.com/live/htdocs;
            location /404.html {
                    root /home/rptlab/etc/nginx/html/;
                    }
            error_page 500 502 503 504 /50x.html;
            location /50x.html {
                    root /home/rptlab/etc/nginx/html/;
                    }
            location /favicon.ico {
                    alias /home/rptlab/website/orion1.reportlab.com/live/htdocs/favicon.ico;
                    }
            location / {
                    uwsgi_pass 192.168.0.251:13080;
                    include /etc/nginx/uwsgi_params;
                    }
            }
    server {
            listen 443 ssl;
            include /home/rptlab/etc/certs/reportlab.com/reportlab-ssl.nginx;
            server_name orion1.reportlab.com;
            charset utf-8;
            error_page 404 /404.html;
            root    /home/rptlab/website/orion1.reportlab.com/live/htdocs;
            location /404.html {
                    root /home/rptlab/etc/nginx/html/;
                    }
            error_page 500 502 503 504 /50x.html;
            location /50x.html {
                    root /home/rptlab/etc/nginx/html/;
                    }
            location /favicon.ico {
                    alias /home/rptlab/website/orion1.reportlab.com/live/htdocs/favicon.ico;
                    }
            location / {
                    uwsgi_pass 192.168.0.251:13080;
                    include /etc/nginx/uwsgi_params;
                    }
            }


    obviously we could just remove the http access and be more secure. We test the setup with traditional sudo nginx -t and if OK sudo service nginx restart.






