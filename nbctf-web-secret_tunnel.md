# web/secret tunnel

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/daf49757-0990-4750-b353-6ef41eec2c36)

We're given a link to the website and what is hopefully the source of the website
Let's try the website first.

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/474ef7a7-02d1-450c-a26a-90e1f4fec360)

It lets us enter in a url, and says that it will display the first 20 characters of any website. Let's try it out

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/5b027549-6adc-495b-8aee-80c57a9a3a43)

And it does actually work! The first 20 characters of https://google.com are indeed `<!doctype html><html itemscope="`

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/2ca557b9-1a70-4676-a21e-54039df4e976)

This is also our first obstacle. The first 20 characters of basically any website are not going to be very meaningful. Let's take a look at the source then.

## The source
The source is a docker container with 4 important files: 

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/0ce2c44b-5914-4155-bf61-d8faa0b33773)

- main.py 
- flag.py
- index.html
- run.sh

After feeling my way around for a bit, I realized that I didn't really know how Flask works, having never used it before. So, I asked for help from a fellow ctfer about how the program worked.
From there, I was able to use my existing knowledge to figure it out from there. Remeber, don't be afraid to ask for help! No one on a ctf team can be expected to know about every possible topic that could appear in a ctf problem.

#### With that out of the way, what do we need to know?
Flask is a web framework written in Python. The contents of this zip file are probably what the challenge authors are using to create the challenge website you see on the screen.

Either from reading the dockerfile, or just guessing, we can see that `run.sh` is run to start website.
![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/de4c3877-8dcb-40e1-953f-e8c2cbd772ab)

Two python proccesses are started. The first tells Flask to run the flag.py file to start a server on port 1337. The second tells Flask to run the main.py file to start another server.
So there are two servers running. Why is there only one website then? Well 
...

### main.py
![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/961b5837-5685-4366-a3ef-798ebfe39c96)
This file sets up two routes on the server for the public to access. The / route (which can default to just the base url, with no slash), when it receives a GET request, sends back the webpage `index.html`. Looking in the `index.html` file in the source, this is just the html for the site we see [https://secret-tunnel.chal.nbctf.com/]

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/6c2b5747-7e54-4d22-9e25-7b2e84723ece)
There's not much to see in the html, but we do notice that when the form is submitted, the data in the form is sent to the `/fetchdata` endpoint (ex. https://secret-tunnel.chal.nbctf.com) through a POST request

Speaking of which ...

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/cd7f0fbe-5f51-4f2c-b102-d124d4756b74)
The `/fetchdata` route specifies that the url should be read in from the request, checked, and then used to get the first 20 characters of the website linked to by the url.

If the url does not pass the checks, or the request fails, then something other than the first 20 characters are displayed.
All these checks do seem bothersome. However, given how even the website tells us to use this "secret tunnel", it's probably our best chance. With main.py exhausted, lets look into the other file.

### flag.py
![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/44a00c5e-6d43-4f4b-ab16-3cb27ff3ab8b)
Meanwhile, when flag.py is run to create the server, it reads in what is presumably the flag from flag.txt, then returns it when it receives a GET request. _NOTE_: we don't have flag.txt. This is probably a file on their computer, with the flag in it, that we have to figure out the contents of. 

## Experimentation
![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/c8ae810f-edca-48c2-b24b-efbc225b013b)
Using the network tab in devtools, we can see that clicking the button sends a request to https://secret-tunnel.chal.nbctf.com/fetchdata. Makes sense, since the base url is https://secret-tunnel.chal.nbctf.com/.
Let's try https://secret-tunnel.chal.nbctf.com/flag then, since we know that visiting a url sends a get request to the that url's server.
![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/a69eb90b-2309-4cc8-9126-f9b8da3a7d23)
Not that easy. While flag.py does tell /flag to send us the flag, it's a separate process the one that's running https://secret-tunnel.chal.nbctf.com. The contest organizers didn't make it _that_ easy.
So how do we get to /flag? Well, the unindentified teammate came in clutch, giving me the hint that local servers tend to use 127.0.0.1 for the ip address, as it's equivalent to localhost.

Then remember that `run.sh` specifies that the flag.py process should create a server using `port=1337`. Thus, the url we really want to target is 127.0.0.1:1337/flag

However, if we try to go to this address, we will go to a site hosted on our local development server, which probably doesn't have a /flag. We need the secret tunnel's server to go to 127.0.0.1:1337/flag, and tell us what it sees.

How convenient it is that this site happens to have a form to submit a url that the server will then try to get content from. 

We put it in, and... ![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/f6f67a50-79e6-4613-839e-0e40fec0b37b)

oh right the checks.

![image](https://github.com/AnthonyFangqing/ctf-writeups/assets/77250066/12f0a063-f626-4a4b-98fd-9a346acf11b6)

We use 127, mutliple periods, and flag in our response. So, we need a url that's identical to http://127.0.0.1:1337/flag, but that doesn't use some of the restricted characters.

## Bypassing the checks
It was at this moment that I tried to remember that link shortening site that creates a url that automatically redirects to another url. You know, so we could change the url we're submitting without actually changing the final destination. I also couldn't remember if this site did any shennanigans to the shortened url that might cause the /fetchdata function to get data from the wrong site, so I moved on.

You could also host your own website that automatically redirects to the specified url, but I didn't have a website handy to do that.

Then, I remembered that Tom Scott video where he explains how %20 in a url actually means the space character, and that websites treated urls with them as if there were spaces instead of %20s.
I then asked ChatGPT what the proper name of that function was, and it turns out it was "url encoding" (https://www.w3schools.com/tags/ref_urlencode.ASP)
And, it works for characters that aren't just space! It even works for letters and numbers!
To test it out, I sent in https://%67oogle.com, and it correctly got data from google.com! I wasn't sure if it was perhaps decoding it first, and then checking, but it was worth a shot.

Armed with the the conversion table, I turned http://127.0.0.1:1337/flag into http://%31%32%37%2E0%2E0%2E1:1337/%66%6C%61%67
Truth be told, I didn't need to completely convert flag into that mess at the back, and I didn't need to convert every period, but oh well.

Sending that url in, the secret tunnel kindly gave me the flag nbctf{s3cr3t_7uNN3lllllllllll!}

PS: I definitely learned to ask for help in this problem. Without a clear knowledge of how Flask was working and how the source functioned, I could not have hoped to solve this.

PPS: localhost works too, instead of 127.0.0.1

