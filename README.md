# 'Madness' box writeup
## Madness is a CTF box created by optional and available on the [TryHackMe platform](https://tryhackme.com).
## Read about [JPG Signature Format: Documentation & Recovery Example](https://www.file-recovery.com/jpg-signature-format.htm), 
# ![bg](images/mad_bg.png?raw=true "Title")

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



