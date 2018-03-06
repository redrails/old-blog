---
title: "Getting started with Twilio"
categories:
  - article
tags:
  - java
  - automata
  - theory
  - computer science
header:
  overlay_image: http://www.cloudways.com/blog/wp-content/uploads/Twilio-SMS-on-Cloud-Banner.jpg
  overlay_filter: 0.4
  caption: ""
---

A few weeks ago I visited BRUMHACK16 and we decided to use Twilio to build something cool and unique, I thought it would be a good idea to get some more people introduced to the whole [Twilio API](https://www.twilio.com/docs/api/rest) and how useful it is.

# What is Twilio?

[Twilio](https://www.twilio.com/) is very broad but for the most part, it provides you with the capability to composing and receiving calls and SMS messages. Why is this useful? Well, ever used 2FA or needed to confirm your number for a website or application? Well, those applications essentially use the Twilio API to send you codes or other messages as such. This is all done automatically and the cost of Twilio is very cheap so it's a very good option.

You might be wondering why I am promoting Twilio and it's for the simple fact that they have amazing support, people, documentation, help and prices. They also have amazing products that they are rolling out very frequently which makes it a very viable option to get yourself to start using. I wouldn't recommend any other service in competition to Twilio, personally.

# How to setup Twilio

Setting up Twilio is fairly simple, beware that you will need to link either your PayPal or bank account so you can top up your account with credit. Firstly, you will need to sign up to Twilio [using their website](https://www.twilio.com/try-twilio).

Once you've signed up and have some money in your account, you can "buy a number" which costs $1/month usually and can make calls, send texts or do both. I will only be focusing on basic text messaging in this guide and may come back to show some more features in the future. So for now, just get any number in your country which supports text-messaging.

To buy a number you can access [this](https://www.twilio.com/console/phone-numbers/search) page, and just click the search button to get any number supporting SMS. Once you have purchased a number, you can see it [here](https://www.twilio.com/console/phone-numbers/incoming).

Upon clicking on the number you will be presented with several options.

We will focus on the `Messaging` part of this page, we have several ways of configuring this messaging app.

1. Webhooks/TwiML - You simply create an application to deploy to the web, your app will provide endpoints for Twilio to use and you will delegate requests using that application.
2. TwiML App - This feature allows you create generic Twilio apps that you can use with multiple phone numbers, more information on this is available on [their blog](https://www.twilio.com/blog/2011/06/introducing-twilio-applications-an-easier-way-to-manage-phone-numbers.html).
3. Messaging service - This is for using 3rd party messaging services that provide many useful features for different purposes.

# What is TwiML?

At this point you should be wondering *what is this TwiML you speak of???*. Good question, TwiML is a very basic language that Twilio uses to recognise commands and parse information that we receive. TwiML looks like XML and thus uses `<tags>` for interpreting callbacks.

A very simple TwiML script may look like this ([source](https://www.twilio.com/docs/api/twiml/sms/message#examples-2)):

```
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <Message>
        <Body>Store Location: 123 Easy St.</Body>
        <Media>https://demo.twilio.com/owl.png</Media>
    </Message>
</Response>
```

We start a `<Response>` block which waits for a message to be received, and within that response block we define other commands we want to execute. Let's say that we want to respond with a message, we can simply insert a `<Message>` block which will create a message. Then, within that block we can write a `<Body>` text which will be the message we respond with. There are some other features we can also use like `<Media>` which is essentially equivalent to an MMS where you can attach media. For more information about TwiML please read [this page](https://www.twilio.com/docs/api/twiml/sms/message).

# Let's create something

So, what we will be doing is writing a Python web application that provides endpoints for Twilio to use. It's important to note that creating applications in other languages will essentially generate a TwiML script dynamically so that we don't have to write out XML. Let's see an example of this, using the Twilio API for python I'll write a simple response script in Python and I'll be using the Flask Framework for this guide but any web framework should work.

In this scenario let's say we want to make a response with a message, and we want to tell the sender what they wrote.

So our conversation should look like this:

```
Tom            : Hello Twilio
Twilio Response: You wrote: "Hello Twilio"
```

We will use the [Twilio Python Library](https://www.twilio.com/docs/libraries/python) to make this response, we can do the following:

```
from flask import Flask, request, redirect
import twilio.twiml


app = Flask(__name__)

@app.route("/message", methods=['GET', 'POST'])
def makeMessage():
	resp = twilio.twiml.Response()
	resp.message("Hello World!")
	return str(resp) 

if __name__ == "__main__":
    app.run(debug=True)
```

Now, we can run this script by:

```
python yourapp.py
```

When this app runs, we will simply be able to load the page `http://localhost:5000/message` and you'll see "Hello World!" being displayed. However, if you view-source on this page we will see the following:

```
<?xml version="1.0" encoding="UTF-8"?><Response><Message><Body>Hello World!</Body></Message></Response>
```

Which is basically the TwiML we wrote before. So the library is just generating what we want in the TwiML form. The reason to use a web-server to do this is because we will later customise the response message. You will also notice that there is no API keys being used here, and that is because we don't need them. We are not composing a message or a call, we are simply providing Twilio with a response TwiML to use that we are generating on the server.

Let's dive deeper into this and connect it to Twilio's website so we can see some messages on our mobile phone.

For this I'll use ngrok which just provides a tunnel between private and public API; it just tunnels requests from the web to my machine instead of me using a public API. Learn about ngrok and how to install it [here](https://ngrok.com/).

On the Twilio website under the option of Messaging we will provide the endpoint that we want Twilio to use when a message comes in:

![](https://i.imgur.com/bEcyCdL.png)

Now that we have this, let's send a message to our number and see what we get.

![](https://i.imgur.com/rNwofou.png)

So we get a response like we had hoped. Now we can do other cool things, let's try and respond with the text that we receive. For this we can use Flask's `request` [context](http://flask.pocoo.org/docs/0.11/reqcontext/). 

Let's change our function to this:

```
@app.route("/message", methods=['GET', 'POST'])
def makeMessage():
	resp = twilio.twiml.Response()

	received_body = request.form['Body']

	resp.message("You wrote: {}".format(received_body))

	return str(resp) 
```

And when we text our number we get the following:
![](https://i.imgur.com/mbVS2cK.png)

# Going Forward

So as we can see Twilio can be very useful for text messaging, what me and my team created was an application that would give you information over the web when you didn't have internet. So whatever the reason for you not having internet access, you can simply use SMS messages to retrieve information from the web. The connection between you and Twilio is simply by text, but all the searching and internet usage is done by us. Here is a little screenshot of it working:

![](https://i.imgur.com/wB4DQP3.png)

As you can see, we can send the application commands which it can then deduce to what information is required to send back. We use many different APIs including Wolfram Alpha, Google Search and Google Maps with a bunch of other libraries that can provide information as needed. 

I hope that this guide has been useful and encourages you guys to try it out!
