# UFO Data Science - How We Get Data, Part 2

## Advanced wget
*(with liberal amounts of assistance from [the GNU manual](http://www.gnu.org/software/wget/manual/wget.html))*

Working on [the full monthly index of all NUFORC reports](http://www.nuforc.org/webreports/ndxevent.html), let’s dive deep into the wget command capabilities, and pull a full local copy of the index. This way, all further work on the project requiring the data can be done offline.

```-r``` - The recursive argument is the heart of this process, as this automates the entire download process, grabbing local copies of each file with in the specified remote tree, storing them in a local directory tree as files that now contain updated links to the local copies of the files. Amazing. Let’s do it right, though, as we don’t want a giant recursive process to get out of hand.

```-l (lower-case L)``` - This argument limits the depth of a recursive ```wget``` command. ```-l 1``` would return the main page passed in the command, and the pages directly linked from the main page. ```-l 2``` would additionally return the pages from links present on the second step of an ```-l 1``` process.

```-w``` - We need to add a delay to the command process, ensuring that the remote server isn’t overtaxed by ```wget```'s stream of requests. On a cursory glance, I mostly saw plain ’90s-style html (text only), and would not initially expect that, even with tens of thousands of records, ```wget``` of the full index would be a large process. To this, I counter with two things:

1. It is just good ethical practice to add a delay into large processes like recursive ```wget```, as it doesn’t overtax their servers and/or alarm their administrators, especially if you haven’t sent prior notification to their admins of your intentions (a suggested practice), of which they have no idea what it could be.
2. There’s no rush here. I don’t need this data immediately, with some super-critical deadline looming. So, hand-in-hand with point #1, let’s just take our time and add a delay. I’ve seen recommendations around Stack Overflow of between 1 and 15 seconds for what delay to use.

```-H``` - On the NUFORC index page for events sorted by month, as well as each month’s page, there is a link to the NUFORC home page. However, it is outdated, linking to ```http://www.nwlink.com/~ufocntr``` instead of [the current home page](http://www.nuforc.org/), which simplifies our process slightly. The ```wget``` command, by default, ignores all links that point to outside domains, that is, domain names other than ```nuforc.org``` in our case, as this could create a runaway web crawler (without any depth limits in place). So we can run our ```wget``` without worrying about recursively grabbing links from the home page each time we touch that link (on each month’s index page). Alternatively, we could add the ```-H``` argument to span all hosts, with ```-D``` added to either allow only ```nuforc.org``` links or exclude any ```nwlink.com``` links.

```-N``` - If we are thinking long-term for this project, and I am trying to look ahead to improving processes and the longevity of updating this project, one thing to keep in mind is that NUFORC is an active organization, receiving new reports every day, and updating them onto the site database regularly. It would be inefficient to have to re-scrape the entire index from the site for each time that I want to update the local copy. ```wget``` is built for this, by

1. Automatically marking the modified time on the local copy of each file with the last modified data on the server copy of each file, and
2. Providing the -N argument to crawl through the tree and only download any files that have ‘Last Modified’ header data on the server copy of each file that are newer than the local copy (or are missing from the local directory entirely).

I’ve successfully tested the following command for retrieving the full index for July, 2014’s records:

```
$ wget -l 2 -w 2 -Nr http://www.nuforc.org/webreports/ndxe201407.html
```

```-np``` - At the end of the process, ```wget``` reports that it completed in 1h 12m 0s of wall clock time, but downloaded 735 files (a total of 19 MB) in 2m 22s. Intermittently throughout the process, it did retrieve the heading links on each report’s page that point back to the four index pages. In the recursive ```wget``` process, you can add the ```-np``` argument to prevent the process from looking at any links that are above (in the directory tree) the address given in the command itself. The complete command to run, for a full download of the entire index would be:

```
$ wget -l 2 -w 2 -Nr -np http://www.nuforc.org/webreports/ndxevent.html
```

Looking at the results of this process, I’m noticing that one element I’d like to maintain in the scrape process (via *BeautifulSoup*) is the link to each event’s page with the respective event data. On the *NUFORC.org* site, these links are all maintained as relative links on their parent page, of the style . Part of the scraping and parsing process would then have to be prepending the absolute path to these links, as ```http://www.nuforc.org/webreports/```.

##Update, mid-process:##
We’re four hours into the ```wget``` process using the above command, and the wi-fi connection blips, or you forget to plug in your laptop, or have a nervous ```Ctrl-C``` habit that flares up. In any case, the ```wget``` process interrupts, and now you’ve got to resume where you left off. Is this possible in ```wget```? Well, I’m not sure.

With -N, all files downloaded are timestamped, so in a new run of the process, those files are "skipped", but still have to be checked, using the same ```-w``` wait time as with a newly downloaded file. And when most of these NUFORC pages take 0.3s or less to grab, skipping by timestamp isn’t a tremendously efficient way to resume this 90,000-file retrieval, when you’re already four hours (and only 5,050 files) along on your wget journey.

There’s -c, a wget argument for resuming a partially-downloaded file, but that only applies specifically to one file, not to our process here, spanning two levels of a directory tree.

So where does that leave us, mid-stream? It’s tricky, because ```wget``` initally calls the page from the address given when you initialize the process, and then proceeds to seek, evaluate, and execute your request on every - *single* - link listed on that first page. How am I going to tell wget to resume where it left off, without having to start from the top all over again?

Here’s what I have to work with:
The last file completely downloaded was from ```http://www.nuforc.org/webreports/106/S106294.html```. This links to a sighting report from Colorado, occurring on January 13, 2014, listed (by ```wget```'s reading) a little over halfway down the page on [the January, 2014 events page](http://www.nuforc.org/webreports/ndxe201401.html). From here, I would need three things for ```wget``` to do:

1. to resume its process at this specific point on the page,
2. to download the sighting reports linked from the rest of the page, and
3. to then return to [the Report Index by Month](http://www.nuforc.org/webreports/ndxevent.html) (our top-level page for the process) and resume its crawl with the December, 2013 page.

Off the bat, for practicality’s sake, I think I’ll cut my losses, which are slight, and skip worrying about step 1. For 700 records in the month, ```wget``` will spend a few minutes traversing down the page to resume where it left off. The main question is, then, how to command ```wget``` to resume the process at January, 2014’s page, skipping the links for July through February. Let’s try a two-step process. First, finish downloading January, 2014 with the following command:

```
$ wget -l 1 -w 2 -Nr -np http://www.nuforc.org/webreports/ndxe201401.html
```

With this step completed, we now have an easily contained subset of links for which we’d like ```wget``` to ignore. We will instruct it to ignore all links to 2014 pages using the ```--reject-regex urlregex``` argument to ignore those pages via the links ```ndxe2014??.html```...

No, that didn’t pan out. Here’s the plan, and it’s already tested successfully with a small sample:

1. From the command line, write your ```./www.nuforc.org/webreports/``` directory contents to a text file, via:

  ```
  $ ls ./www.nuforc.org/webreports/ > months.txt
  ```
  
2. This populates the file with a list of all normally visible files contained in the ```webreports/``` directory, which includes the index page for each month, along with each of the main index pages. Using your text editor, go into ```months.txt``` and remove the directories we want to ignore in our partial ```wget``` of the rest of the site. In my case, it is the seven months of 2014.
3. Now we can run ```wget``` to pull the files for only the directories listed:
```-i``` points to the input file that lists, with one file per line, the file and directories to initially follow. This file supersedes the list that ```wget``` would build in a default process for following links down from the base URL through the number of levels specified.
```-B``` specifies the base URL from which to start this process. Be sure here that with the address specified, the files listed in ```months.txt``` could be directly appended for ```wget``` to successfully request.

```
  $ wget -l 2 -w 2 -Nr -np -i months.txt -B http://www.nuforc.org/webreports/
```

Overnight, the process had been running, but on a glancing update the following morning, I found that the process kept trying to grab pages from links on the header for the individual event report pages. These were dead links, but ```wget``` was still spending a few seconds on each one - four per page, across all events up through July, 1978 at this point. To rectify, I stopped the process and traced through the command to realize that the ```-l 2``` argument was going too deep with the ```-i -B``` arguments. With ```-i months.txt``` including each month’s index page, ```wget``` was now treating each of those listings as a level 0 element on the tree, and therefore grabbing each event from one of those month indexes was a level 1 process. (Sorry, I’m making up the language as I go along to describe my interpretation of the ```wget``` process; this may not be how ```wget``` itself works.) Knowing that, I altered the ```wget``` command, deleted all filenames in ```months.txt``` up through July, 1978 (```ndxe197807.html```), and restarted the process:

```
$ wget -l 1 -w 2 -Nr -np -i months.txt -B http://www.nuforc.org/webreports/
```

This last step ran all weekend, starting on a Saturday morning, after the above update, and completing after continuous operation on Monday afternoon.

I think that’s all my computer wants to handle this week.

----
[Link to original post](http://scotterenaissance.tumblr.com/post/93524577748/ufo-data-science-how-we-get-data-part-2)
