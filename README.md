# 'Madness' box writeup
## Madness is a CTF box created by optional and available on the [TryHackMe platform](https://tryhackme.com).
## Read about [JPG Signature Format: Documentation & Recovery Example](https://www.file-recovery.com/jpg-signature-format.htm), 
# ![bg](images/5iW7kC8.jpg_Min.png?raw=true "Title")

## Foothold
+ **We deploy the machine and start with a nmap scan for open ports**

``nmap -sV -sC -oN scan1 10.10.94.80``

+ **We observe 2 open ports, ssh and an http, the last one with an Apache service**

# ![1](images/nmap_mad.jpg?raw=true "mad")

+ **Opening the web-site, we can view an Apache2 Default Page, but looking into the source-code of the page, we observe something interesting**

# ![2](images/page.jpg?raw=true "page")

**There's a comment through the lines** ``They will never find me`` **and an image above, so it's obvious that the image it's a path to go on. Let's get that image into our machine**

``wget http://10.10.94.80/thm.jpg``

+ **Opening the image, we have a quite little problem:**

``The image “file://x/Madness/thm.jpg” cannot be displayed because it contains errors.``

**Let's look into the content of our image. It looks like our output says that our image is in PNG format. Maybe someone changed the image format.**

# ![3](images/head.jpg?raw=true "head")

+ **Furthermore, reading the [JPEG Signature](https://www.file-recovery.com/jpg-signature-format.htm) article, we can spot this: *JPEG files (compressed images) start with an image marker that always contains the tag code hex values FF D8 FF.* Let's take a look into our image header in hex format, using hexdump**

``hexdump thm.jpg | head -1``

# ![4](images/hexo.jpg?raw=true "hex")

**We can clearly see the first bytes are modified, but we can change them using the hexedit tool. Let's modify the first 2 hex sequences values with:**

``FF D8 FF E0   00 10 4A 46``

``hexedit thm.jpg``

# ![5](images/hexedit.jpg?raw=true "hexed")

**Opening our image now, we can see a font which tells us a hidden dir exists on our web server**

``xdg-open thm.jpg``

# ![6](images/hiddendir.jpg?raw=true "hidden")

+ **Let's take a look into that directory and see what's going on**

# ![7](images/webhid.jpg?raw=true "webbed")

**It seems like our mad friend wants to guess his secret .. But since i'm not a good guesser, let's take a look into the source code of the page**

# ![8](images/guess.jpg?raw=true "secret")

**He left some unwanted text inside. We know now, that the secret is between 0 and 99, but in the same time, looking into the page we see that there was already a secret entered. Maybe it's a parameter of the page we could enter; let's go on:**

``http://10.10.94.80/th1s_1s_h1dd3n/?secret=2``

# ![9](images/secret.jpg?raw=true "secret")

**Our secret seems to be entered and read inside the page, so let's go ahead and create a mini brute-force script to check 100 possible variants.**

+ **I'm gonna use curl to get my requests and page data and i will write a shell script**

```
#!/bin/bash

for i in {0..99}
do
        # modify the ip address below
        curl --silent http://10.10.94.80/th1s_1s_h1dd3n/?secret=$i | grep right >> /dev/null
        
        if [ $? -eq 0 ]
        then
                echo "$i is our SECRET page"
                curl --silent http://10.10.94.80/th1s_1s_h1dd3n/?secret=$i
                break;
        else
                echo "Secret $i is wrong"
        fi
done

```

**Let's make executable too and let's run it**

``chmod a+x secret_guess.sh``

``./secret_guess.sh``

# ![10](images/secrfound.jpg?raw=true "secretf")


