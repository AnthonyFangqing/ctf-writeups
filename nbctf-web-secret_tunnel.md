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
This file sets up two routes on the server for the public to access. The / route (which can default to just the base url, with no slash) sends back the webpage `index.html`. Looking in the `index.html` file in the source, this is just the html for the site we see [https://secret-tunnel.chal.nbctf.com/]
