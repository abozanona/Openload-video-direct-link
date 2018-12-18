# Openload-video-direct-link
I was working on developing android/ios mobile applications that displays movies from openload.co server on mobile.

The problem with openload is that they don't provide direct links for videos in the page source code, instead, they generate the video url from js by making calculations on some sort of keys.

Openload embed video links format is `https://openload.co/embed/{VIDEO_ID}`. By viewing the source code of this page we can find 22 `script` tags and a lot of other HTML tags.

After trying to remove random elements from the page and keeping an eye on the console and making sure that the video loads normally I found that I only need the `video`, `p`, `JQuery` tags and the last `script` tag to load the video url.
In addition to the above tags, we also need this script to display the video:

    <script>
      $.ready();
      //pId is the ID of the second paragraph tag in the page
      document.getElementById('olvideo').src = 'https://openload.co/stream/' + $('#" + pId + "').text() + '?mime=true';"
    </script>
    
The above works like a charm on normal devices.

Running the above on a server and sending it back to the mobile application was not a good idea, since openload blocks ips that sends a lot of request to the website, So we have to run the code locally.

Even though, I tried to run the code on a server using PhantomJS but it didn't work, and that's because if the js code detects phantom js it doesn't execute.

The remaining code contains two paragraph tags:

    <p style="" id="OddEftOyxU">53e3581b36e8a9b13f4f71df2ae9dc1df329ff5715f4b60b8fc65da3267cccf7c4f5335b49546f4066014f4e405d3158534f62156675705d49016d68626e5f027b51464873015f6f78566103476661794201697c5e484b035c487d4e346c6653444601</p>
    <p style="" class="" id="DtsBlkVFQx">640K ought to be enough for anybody</p>

The first paragraph contains some sort of keys that is used to generate the video url, and the next one is where the link is placed after generating it.

The `video#olvideo` tag is where the video link placed after executing our new add `JQuery` script.

And the last script tag is where all the magic happens! and it's the same for all openload embedded links! It consists of three parts; the first and third parts are not readable at all, while the second part is better written.

I tried to remove the last part and nothing bad happens, it's useless regarding generating the video link.

After prettifying the first part and executing it line by line, I found that it just declares a variable `window.ffff` and assignes it's value to the id of the first `P` id. The variable `ffff` is used later in the second part.

Open load also tried to make the second part as fuzzy as possible. It contains a lot of hex chars `\xXX`, which you can easily read them by replacing them by their ASCII representation while you're fetching the page content.

The second part's job is mainly to generate the video url and to generate a false url if the page is not running on a web browser.

It has a lot of definitions of a normal web browser: 

    if(('toString' in sin&&sin.toString().indexOf('[native code')!=-1&&document.getElementById.toString().indexOf('[native code')==-1)||window.callPhantom||/Phantom/.test(navigator.userAgent)||window.__phantomas||pt()||window.domAutomation||window.webdriver||document.documentElement.getAttribute('webdriver'))

So replacing this line with `if(false)` will make it possible to run the js code everywhere.

The second part also uses JQuery to add listner to page onload function access and modify innerText of elements. So You can remove the JQuery CDN script and define your own simole JQuery function. Just make sure that you define both `$` and `JQuery` functions, since the code checks if they both exist and point to the same reference.

-------

At first I was asked to create an Android application to display videos, so I used JSoup to parse the page, make changes in it's DOM and then render it in a webview. and by evaluatin `document.body.innerHTML` I was able to get the renderd DOM, pass it to JSoup again and egt the video link(I could simply evaluate `document.getElementById('olvideo').src` but I didn't :3 ).

I was required later to build an IOS application, so I switched to xamarin and used the same technique only using `AngularSharp`.

The performance in xamarin forms was disastrous, so I had to switch to nativescript. 

Nativescript doesn't have a library to parse HTML, so I used regex instead.
