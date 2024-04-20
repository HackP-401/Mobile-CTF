
# Mobile Hacking Labs: Strings Writeup 
Hello everyone! Hope you all are doing well. In this blog post I’ll walk you through my learnings from the Mobile Hacking Lab - [Strings](https://www.mobilehackinglab.com/course/lab-strings) and walk you through my journey to crack this challenge.  

---
# Introduction:
Mobile Hacking Lab gives you access of Corellium instance for 120 min Online mode (Browser), I decided to solve the challenge on my linux machine and download the base.apk from corellium instance. To download the same in your machine, follow the steps. 

<img align="right" src="https://github.com/HackP-401/Mobile-CTF/assets/73713378/7938fdaf-ce83-4b1a-9e54-feb63f32046f" alt="1" width="200" height="200" />
<br />
<br />
After downloading | installing the apk in android emulator, opening it display with a message saying “Hello From C++”. As we don’t have any fruitful     options to perform, thought to open the application on JADX to get more visibility on the code/logic prospective.
<br />
<br />
Usually, I start the source code review activity from AndroidManifest.xml file, and let’s see what juicy information we can gather. 
<br />
<br />
<p align="right" style="font-size:0.1;" >Fig 1: Application with greeted message</p>
<br />
<br/>

![2](https://github.com/HackP-401/Mobile-CTF/assets/73713378/4fea08e7-7795-4dcd-bc0b-cbc07ee32d55)

<p align="center" style="font-size:12;">Fig 2: Application Manifest file</p>
<br/>
<br/>

<p>We can see an exported activity Activity2 is mentioned with certain intent-filter indicating accessibility from 
other applications., if we want to trigger this activity, we need to call the intent using adb command with
scheme mhl and with host value labs. The format to call using activity manager is as follows: 
“mhl://labs/payload”</p>

<p>adb shell am start -d "mhl://labs" -n com.mobilehackinglab.challenge/.Activity2</p>

![3](https://github.com/HackP-401/Mobile-CTF/assets/73713378/b0c6993f-88dc-4445-807d-39ab3108b6b6)

<p align="center" style="font-size:12">Fig 3: Trigger the exported activity using adb command</p>
<br/>
<br/>

<p> :thinking: hmmm……. Calling the exported activity make no changes, nothing happened… make me more curious 
what actually the Activity 2 is doing? </p>

<p>To understand this let’s jump into the source code of Activity 2 and analyse.</p>

![4](https://github.com/HackP-401/Mobile-CTF/assets/73713378/37275aea-ce77-497f-9aaf-9c8bddd0fe35)

![5](https://github.com/HackP-401/Mobile-CTF/assets/73713378/62b08efd-c651-4984-9cb5-c18dd56c8390)

<p align="center" style="font-size:12">Fig 4(a): Source code of Activity 2</p>

<br/>
<br/>

* In line number 30 & 31, we can see this application code try to read **DAD4** file, from shared preferences
and try to read the string associated with it “**UUU0133**”. 
* In line number 34, the application code comparing the action with “**android.intent.action.VIEW**” to retrive 
the intent.
* In line number 35, the application code tries to compare the values coming from shared preferences with 
the value return from the function m26cd.

<p>Let’s try to analyse the code for the **m26cd** function:</p>

* In line number 86, we can find **m26cd** function, hmmm…. It’s contained several processes, discussing one 
after another. 
  - The function returns today’s **date in dd/mm/yyyy** in string format. 
![6](https://github.com/HackP-401/Mobile-CTF/assets/73713378/4a17385a-e1c4-495a-b29f-01d7b30a1eac)
<p align="center" style="font-size:12">Fig 4(b): Source code of Activity 2</p>

<br/>
<br/>

It will be very difficult to get the same value for **u_1** and **m26cd** at the same time, or for isActionView
and **isU1Matching**. So, the better option is to bypass **isActionView** && **isU1Matching** checks. For that 
we need to create shared preferences file and send the intent with VIEW action.

<p style="margin-left:10">
  🤔 In this moment I’m thinking which function is creating the existing shared preferences file?
Hmmm….. So I try to find the existing function, which can help us to generate a new shared 
preferences file.
</p>

And guess what I found that in MainActivity class, named as **KLOW()** function.

![7](https://github.com/HackP-401/Mobile-CTF/assets/73713378/bf77ad4e-3cca-4e42-b6ef-62726c7216c2)

<p align="center" style="font-size:12">Fig 5: Source code of MainActivity</p>

<br/>
<br/>

<p align="center" >Now what we need to do just to call the **KLOW()** function using Frida script. </p>

But before this we have couple of other findings in the **m26cd** function.

* In line number 39 & 40, we can see application code checks Uri and Scheme, provided in manifest file and 
fetch base64 value using **uri.getLastPathSegment()**. 
* In line number 47, we are comparing base64 decoded value with “**str**” variable which contain decrypted 
value of “**bqGrDKdQ8z026HfIRsGWA==**” string.

We can decrypt the above mention string with the hardcoded key, as we already get the idea about the 
encryption algorithm is AES with the following mode CBC:

* Encrypted String: “bqGrDKdQ8z026HfIRsGWA==”
* Key: “your_secret_key_1234567890123456”
* Mode: CBC
* IV: “1234567890123456”

<p align="center">
  <img src="https://github.com/HackP-401/Mobile-CTF/assets/73713378/e590e089-a3b5-438e-8440-4c877244dba0" alt="1" />
</p>
<p align="center" style="font-size:12">Fig 6: IV value comes from Activity2Kt.fixedIV variable</p>

<br/>
<br/>

Applying the values in [CyberChef](https://gchq.github.io/CyberChef/), and we gat the below result. 

![9](https://github.com/HackP-401/Mobile-CTF/assets/73713378/1432204b-5ac9-4351-b865-144b6e7929be)
<p align="center" style="font-size:12">Fig 7: Decrypted values of encrypted strings</p>

<br/>
<br/>

So, finally we get the secret value “**mhl_secret_1337**”. Now we need to convert this string to base64 and 
pass this at the end of the uri last path segment using adb command. 

![10](https://github.com/HackP-401/Mobile-CTF/assets/73713378/472204a1-4c04-4acd-84c3-68bde528e46b)
<p align="center" style="font-size:12">Fig 8: Encoding the secret value</p>

<br/>
<br/>

This will eventually pass all the checks and load “**flag**” library and call the **getflag()** function. 
Calling the exported activity and injecting the Frida script to call the *KLOW()* function is the consecutive 
task, displayed below. To call the *KLOW()* function, I used this Frida script.

![11](https://github.com/HackP-401/Mobile-CTF/assets/73713378/fd846731-0bf0-4908-a276-a8b617d0d944)

adb shell am start -a android.intent.action.VIEW -W -d "mhl://labs/bWhsX3NlY3JldF8xMzM3" -n
com.mobilehackinglab.challenge/.Activity

![12](https://github.com/HackP-401/Mobile-CTF/assets/73713378/caf56c46-7c7f-4a03-add3-24f046dc1511)
<p align="center" style="font-size:12">Fig 8: Calling the *KLOW()* function using Strings.js and calling the exported activity using adb activity manager</p>

<br/>
<br/>

Oh, great. We het success message, but what about the flag? 🤔

As getflag() function is from libflag.so library, I thought to check that using Ghidra. It contains several 
functions and several calls of memcpy (*note: Memcpy is a function in the C standard library that copies a block of memory from 
one location to another. It is defined in the string.h header file.). From there we can get a idea about dumping and analyse the 
application memory.

![13](https://github.com/HackP-401/Mobile-CTF/assets/73713378/a3759c79-b8b8-441f-ac7c-233e4551f094)

<p align="center" style="font-size:12">Fig 9: Using Ghidra we are analyzing memcpy calls</p>

<br/>
<br/>

For dumping and analysing the application memory we can use Fridump. Lets use this…
Using the command: **python3 fridump3.py -u Strings**

![14](https://github.com/HackP-401/Mobile-CTF/assets/73713378/da05a9f9-a06f-453d-86aa-29fa69c09483)

<p align="center" style="font-size:12">Fig 10: Using Fridump we are analyzing the memory and finding the secret in the memory.</p>

<br/>
<br/>

# Conclusion:

This lab offers a comprehensive exploration of Frida scripting, covering everything from analysing function 
results to memory scanning for secret retrieval.

Happy Learning 😄

