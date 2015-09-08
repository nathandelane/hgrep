# Basic HGrep Usage #
### This page describes how to use HGrep in general ###

The simplest way to use HGrep is to get a list of headers back from a server. For example when you type something like:
<pre>
C:\>hgrep uri=http://nathandelane.com<br>
</pre>

you will see the program return the server headers like this:
<pre>
Response Headers:<br>
Keep-Alive                              timeout=15, max=100<br>
Connection                              Keep-Alive<br>
Content-Length                          215<br>
Content-Type                            text/html<br>
Date                                    Wed, 05 Aug 2009 20:28:52 GMT<br>
Server                                  Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch<br>
X-Powered-By                            PHP/5.2.4-2ubuntu5.6<br>
</pre>

As you can see this can sometimes prove to be very useful. From this information you can already derive that I, on my server, use Apache HTTP server 2.2.8, I am running an Ubuntu Linux server, and I am using PHP 5.2.4-2.

Let's take a look at somebody else's server now, for example HackThisSite.org:
<pre>
C:\>hgrep uri=http://www.hackthissite.org<br>
</pre>

The result is:
<pre>
Response Headers:<br>
Pragma                                  no-cache<br>
Keep-Alive                              timeout=5, max=100<br>
Connection                              Keep-Alive<br>
Transfer-Encoding                       chunked<br>
Cache-Control                           no-store, no-cache, must-revalidate, post-check=0, pre-check=0<br>
Content-Type                            text/html<br>
Date                                    Wed, 05 Aug 2009 20:44:06 GMT<br>
Expires                                 Thu, 19 Nov 1981 08:52:00 GMT<br>
Set-Cookie                              PHPSESSID=8pallmnfmabg42s8ebc3frdjt1; path=/<br>
Server                                  Apache<br>
</pre>

Here you can see that they are setting a cookie. It also looks like they custom compiled their Apache server so that it wouldn't include the version. They must also be using PHP because of the PHPSESSID cookie. Not much else though.

Although one of the main reasons I wrote HGrep was in order to get server headers back in order to verify that I was pointing to the server that I thought I should be, it is also useful for retrieving actual data from the server in the form of HTML. There are two selection protocols currently in place, one for XPath, and one for Regular Expressions. The Regular Expression selection protocol is currently in an experimental stage, which is explained in the output when you use it, for example:
<pre>
C:\>hgrep url=http://www.google.com "find-regexp=<img"<br>
Matches Found (Experimental):<br>
/<img/<br>
0 <br>
<br>
Unknown end tag for </style><br>
<br>
<div id="tba"><a href="#" onclick="tbc();return!1" id="tbe"><img border="0" src="/images/close_sm.gif" />...<br>
</pre>

Which can go on and on forever. This is because the Regular Expression evaluator is a line evaluator, so for each line that the regular expression matches, you get a pysical line back from the file.

The XPath selection protocol is much more useful, because it selects and returns individual elements. XPath is a W3C standard for selecting elements in XML-based documents. Since for all intents and purposes moden HTML is XML based, XPath turns out to work nicely with it. More on XPath syntax can be found at <a href='http://www.w3schools.com/XPath/xpath_syntax.asp'>W3C</a>. I will only discuss a limited amount of XPath here.

Probably the most common XPath, at least that I use is relative XPath syntax. Relative XPath syntax requires that you formulate your expression as:
```
//<tagName>[@attributeName='value'...']
```

You can use the xpath like this on the Google homepage:
<pre>
C:\>hgrep uri=http://www.google.com find=//img<br>
</pre>

The result looks something like this:
<pre>
Nodes Found:<br>
img [[[border='0']][[src='/images/close_sm.gif']]]<br>
img [[[src='/images/toolbar_sm.png']]]<br>
img [[[src='/images/dl_btn_left.gif']]]<br>
img [[[src='/images/dl_btn_right.gif']]]<br>
img [[[height='110']][[alt='Google']][[width='276']][[src='/intl/en_ALL/images/logo.gif']][[onload='window.lol&amp;&amp;lol()']][[id='logo']]]<br>
</pre>

What you see is a representation of every <img> tag on the Google homepage. There are five tags here. The square brackets contain all of the attributes. And if there were an innerHtml or innerText value, then it would appear after an equals symbol on the right side. Now that you have this information, you can use it to get more specific:<br>
<pre>
C:\>hgrep uri=http://www.google.com find=//img[[@alt='Google']]<br>
Nodes Found:<br>
img [[[height='110']][[alt='Google']][[width='276']][[src='/intl/en_ALL/images/logo.gif']][[onload='window.lol&amp;&amp;lol()']][[id='logo']]]<br>
</pre>

The nice thing about this is that it does not rely on the physical format of the document, rather it only relies on the syntactic format of the XML, XHTML, or HTML, whatever it may be. I utilize <a href='http://www.codeplex.com/htmlagilitypack'>HTMLAgilityPack</a> to search the HTML using XPath, and it turns out to work very well. I always get expected results.<br>
<br>
Let's try one more to give you a little more taste of XPath:<br>
<pre>
C:\>hgrep uri=http://www.youtube.com find=//a[[contains(@href,'watch')]]<br>
</pre>

And the results:<br>
<pre>
Nodes Found:<br>
a [[[href='/watch_queue?all']][[onmousedown='urchinTracker('/Events/Header/UtilLinks/QuickList');']][[class='hLink']]] = Quic...<br>
a [[[href='/watch?v=FREqWf-lUdc&amp;feature=featured']][[rel='nofollow']][[onmousedown='urchinTracker('/Events/Home/PersonalizedHo...<br>
a [[[href='/watch_queue?all']]] = Added<br>
a [[[href='/watch?v=FREqWf-lUdc&amp;feature=featured']][[title='Gowanus, Brooklyn']][[rel='nofollow']][[onmousedown='urchinTracker...<br>
a [[[href='/watch?v=FREqWf-lUdc&amp;feature=featured']][[title='Gowanus, Brooklyn']][[rel='nofollow']][[onmousedown='urchinTracker...<br>
a [[[href='/watch?v=fhkh_xNPTWw&amp;feature=featured']][[rel='nofollow']][[onmousedown='urchinTracker('/Events/Home/PersonalizedHo...<br>
a [[[href='/watch_queue?all']]] = Added<br>
a [[[href='/watch?v=fhkh_xNPTWw&amp;feature=featured']][[title='Sundance Directors Lab 1: Getting Started']][[rel='nofollow']][[on...<br>
...<br>
</pre>

Whoah! Link dump. That's right, HGrep, it turns out, is very useful for scraping stuff. Here I just scraped all of the links on the YouTube homepage. (I've cut off most of the results as well as the ends of the lines for formatting purposes.)<br>
<br>
And just in case you wanted to know exactly how many links I actually got from that query:<br>
<pre>
C:\>hgrep uri=http://www.youtube.com find=//a[[contains(@href,'watch')]] count-only<br>
Nodes Found:<br>
78<br>
</pre>

If you type <pre>hgrep</pre> or <pre>hgrep help</pre> then you will get a listing of all of the different options you can use and what they do.<br>
<pre>
C:\>hgrep help<br>
HGrep url=<fully qualified url> [options]<br>
Options may be qualified by --, -, or / and in some terminals and cases you may need to qualify complete arguments with ".."<br>
The available options include:<br>
clean                         Removes titles from results.<br>
count-only                    Displays only the number of nodes that would be returned from find option.<br>
encode-line-breaks            Replaces \n and \r with textual representations.<br>
find<xpath>                   XPath expression used to find a specific element of elements in the document.<br>
find-regexp=<regex>           Regular expression used to find a specific element of elements in the document. (This is experimental)<br>
help[=<topic>]                Displays this help.<br>
ignore-bad-certs              Ignores when a bad SSL/TLS certificate is being used by a server.<br>
no-attributes                 Suppresses output of attributes from find option.<br>
no-inner-html                 Suppresses output of inner-html from find option.<br>
no-numbering                  Suppresses numbering from find-regexp option.<br>
post-body=<querystring>       Causes the request to post form data.<br>
return-attributes=<attributes>Returns only the specified attributes of an XPath query<br>
return-headers                Displays the response headers of the request.<br>
return-url                    Diaplys the resulting URL from automatic redirects.<br>
scrub                         Disables automatic display of response headers.<br>
timeout                       Overrides the default timeout setting in the config file.<br>
</pre>

Well enjoy, and I hope you find it useful. If there is functionality you desire, then feel free to let me know.<br>
<br>
Nathandelane.