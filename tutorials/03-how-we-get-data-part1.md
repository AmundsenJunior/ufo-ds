# UFO Data Science - How We Get Data, Part 1

This is the first technical post for the Data Science Through UFOs project, and in my fashion, I’m diving into some detail that at first glance appears a reasonable initial step to take. But, I’ve probably jumped the gun, over eager to get my hands dirty with some of this data, and too impatient for pouring over every paragraph of the various chapters and guides at my disposal that provide a proper introduction and context for performing good data science. Instead, let’s dive into the data, and by the end of this post, you may find that it was, if nothing else, a useful exercise in acquiring some awareness of the value of process.

##WGET, GREP, and Better Options

Jumping right in, let’s pull a webpage down into our local working directory, in my case ```~/dev/ufo-ds```:

```
$ wget -o nuforc_ct.txt http://www.nuforc.org/webreports/ndxlCT.html
```

* ```wget``` pulls code from websites.
* ```-o FILE``` writes wget output to a specific *FILE*.

We next want to ```grep``` the ```nuforc_ct.txt``` file to pull all lines including “haven” and their directly previous line. We want the ```href``` link text from the previous line, as the ```<td>``` tags for each entry go from report link to City on the next line.

```grep --help```  and [the man page](http://unixhelp.ed.ac.uk/CGI/man-cgi?grep) provide details on the command syntax and options you need to execute the command, but had me assuming I needed to write some obscure regex (a lesson for another day, to be sure) in order to conduct my simple keyword search. The internet can and did quickly provide concrete examples of real-world grep commands, just as the following is now itself a decent example:

* ```-i --ignore-case``` ignore case distinctions.
* ```-n --line-number``` print line number with output lines.
* ```-B --before-context=NUM``` print *NUM* lines of leading context.

```
$ grep -B 1 -i -n "haven" nuforc_ct.txt > grep_nuforc_ct.txt
```

However, there are also mentions of town name in the Summary. This grep then looks for each instance of ```<a href="``` on any line, then includes the single subsequent line, City, while also eliminating the Summary and Duration information. Both are redundant for when we pull from the links into a database next.

* ```-A --after-context=NUM``` print *NUM* lines of trailing context.

```
$grep -A 1 -i "a href" grep_nuforc_ct.txt > grep_nuforc_haven.txt
```

We then take our Bash foo to level two with piping in multiple commands, reducing the pile of data files down in the process. I had to go through the long process above to learn ```grep``` before piping commands together.

```
$ grep -B 1 -i -n "haven" nuforc_ct.txt | grep -A 1 -i "a href" > grep_haven_href.txt
```

Lastly, we can get a count of the number of records pulled from ndxlCT.html with ```grep -c```

* ```-c --count``` print only a count of matching lines per *FILE*.

```
$ grep -c -i "a href" grep_haven_href.txt
> 45
```

Okay, let’s expand this effort. The above exercise was a small dose of what is known as “web scraping”, the automated extraction of website data. The process is completed via one or more scripts, such as Bash (above) or Python, for example. For extracting data, web scraping is an incredibly useful and efficient process, if done correctly. The efficiency comes from being both automated and selective; the loops and conditional statements searching for  replace a  This is how search engines develop their data, according to [Udacity’s Intro to Computer Science](https://www.udacity.com/course/cs101) course.

[A friend](https://twitter.com/sergeography/status/485829905213431808) has, since I’d first tested the above, pointed me to [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/), a Python library for parsing HTML documents into a tree format that is easily traversable with web-scraping tools. I started with wget and grep, however, for two reasons:

The process is conducive to learning pipes, a Level 2 Bash skill. For me, at least, combining commands is a first step beyond keeping my bearings in the directory tree and staying aware of file permissions, while also being a first step towards higher-level scripting and task automation in Bash.

They were the only options of which I’d initially been aware. This, too, is an interesting teaching point, because in my head, this first stretch of the UFO data workflow was aiming towards storing all of the data in a database, ready to be manipulated. I’m familiar with SQLite and pysqlite already, on multiple projects (See [trac-closed-timestamp](https://github.com/AmundsenJunior/trac-closed-timestamp) and [rapidsms-opsmile](https://github.com/AmundsenJunior/rapidsms_opsmile) repos), so I’d assumed (again, in my head, not in print) that I would just want to try my hand at PostgreSQL in combination with the Anaconda stack (based on [Google searches](https://www.google.com/search?q=python+and+postgresql+for+data+science) and [NewHaven.IO](http://www.newhaven.io/) conversations).

I also found out about [web scraping etiquette](http://stackoverflow.com/questions/2022030/web-scraping-etiquette), (thankfully not the hard way!).

I think the above, especially point #2, intimates that a full overview post is also needed, setting out some more concrete project goals and technical objectives in order to keep from going rogue on learning data science. This is a very dismissive hindsight mentality, so I’ll allow that I probably wouldn’t have arrived at such conclusions - that the above Bash commands weren’t the most efficient means to initiating the project, nor with a clearer idea of the next steps to be taken - without having first gone through this process itself. Here, finally, I want to emphasize that learning is more the goal than any short-sighted crusade of proving the in-house data visualization team at The Economist to be lazy or inept. Given the choice, the default on this project will be to take the adventurous track, into the unknown database, library, or visualization tool. To deepen awareness of the breadth of what I don’t yet know how to do, and with that awareness strive to close that gap. With the rate of technological expansion and evolution what it is, I want to be comfortable with knowing that I’ll never fully close the gap, never stamp the pursuit as done, never stop digging.
