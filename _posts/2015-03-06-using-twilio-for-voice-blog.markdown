---
layout: post
title: "Use Twilio and your phone to make audioblog posts"
date: 2015-03-06 10:00
tags: python, flask, bootstrap, twilio
slug: twilio-voice-blog
subtitle: "Make yourself a simple audioblog and publish blog posts just by calling a specific number."
author: techstonia
permalink: twilio-voice-blog.html
---

Recently I played with [Twilio](https://www.twilio.com/) API. Their service is mind-blowingly simple - it took me about 50 lines of Python code to set up a MVP. No, I'm not going to start a Twitter-meets-audio kind of startup - you can [steal the idea](http://bubbly.net/). Basically I just have to call a specific number, tell my message and the audio goes to my website. You can find the humblingly minimal audioblogging platform here: [https://twog.herokuapp.com/](https://twog.herokuapp.com/).

##Okay, how can I get this to work? 
The shortest way is to follow the instructions on my [Github repository](https://github.com/techstonia/twog), which I'm copying here as well. 

1. Create a free [Twilio](https://www.twilio.com) account and buy a [voice number](https://www.twilio.com/user/account/phone-numbers/incoming).
2. Clone [this repository](https://github.com/techstonia/twog).
3. Change `AUTH_TOKEN` and `ACCOUNT_SID` in `config.py`. You can find them under your [account settings](https://www.twilio.com/user/account/settings)
4. Create a [Heroku](https://heroku.com) app and push your code into the app. If you're not familiar with Heroku I suggest starting [here](https://devcenter.heroku.com/articles/getting-started-with-python#introduction).
5. Change the request url in your [manage number](https://www.twilio.com/user/account/phone-numbers/incoming) page. This should be in the form of yourappname.herokuapp.com/new.
6. Call the number in [manage number](https://www.twilio.com/user/account/phone-numbers/incoming) to record your first post.
7. Yay! You have a voice blog!

P.S. Twilio service is not free, but there is an initial 30$ credit, which should be enough for playing with their service. You can also delete posts from Twilio website.

## What's under the hood?
It's built with Flask and Bootstrap and all the logic is in `app.py`. When we call our magic number Twilio accesses the [yourappname.herokuapp.com/new](yourappname.herokuapp.com/new) and get's instructions in TwiML, which is a Twilio's markup language. Basically the view checks if the call is made from your number and after you have pressed 1, the [yourappname.herokuapp.com/handle-key](yourappname.herokuapp.com/handle-key) is accessed.
{% highlight python %}
@app.route("/new", methods=['GET', 'POST'])
def new_post():
    from_number = request.values.get('From', None)
    resp = twilio.twiml.Response()

    if from_number == MY_NR:
        resp.say("Hi!")
        with resp.gather(numDigits=1, action="/handle-key", method="POST") as g:
            g.say("To record a post, press 1.")
        return str(resp)
    else:
        resp.say("Go away intruder!")
        return str(resp)
{% endhighlight %}

In handle-key Twilio gets instructions of what to say to you and what to do after the call has finished.
{% highlight python %}
@app.route("/handle-key", methods=['GET', 'POST'])
def handle_key():
    digit_pressed = request.values.get('Digits', None)
    if digit_pressed == "1":
        resp = twilio.twiml.Response()
        resp.say("Record your post after the tone.")
        resp.record(maxLength="10", action="/handle-recording")
        return str(resp)
    else:
        return redirect("/new")
{% endhighlight %}
If your call reaches 10 seconds, you're redirected to `/handle-recording`, and informed that your post is now live.
{% highlight python %}
@app.route("/handle-recording", methods=['GET', 'POST'])
def handle_recording():
    resp = twilio.twiml.Response()
    resp.say("Your post is now live! Goodbye")
    return str(resp)
{% endhighlight %}

Now we only need a view for displaying the posts:
{% highlight python %}
@app.route('/')
def index():
    client = TwilioRestClient(ACCOUNT_SID, AUTH_TOKEN)
    recordings = client.recordings.list()
    return render_template('index.html', posts=recordings)
{% endhighlight %}
It's very straightforward - we fetch recordings and pass them into `render_template` function, which then produces the necessary html for displaying all the posts we have created. So that's that. Simple, huh? 

[https://twog.herokuapp.com/](https://twog.herokuapp.com/)

[https://github.com/techstonia/twog](https://github.com/techstonia/twog)