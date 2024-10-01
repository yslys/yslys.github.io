---
permalink: /
title: "Hi there, I'm Yusen!"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---
I am a second year CS graduate student at the University of Wisconsin-Madison, advised by Prof. Michael Swift.

My research interests include Operating Systems and Computer Architecture.


======
### Research Project:
I designed an interface of near-memory accelerator that communicates with the host via PCIe. This is a software-hardware co-design, which allows me to touch all three layers of computer system: hardware level, kernel level and software level. I regard myself as a combination of three engineering teams. If you do not believe it, you can keep reading. Let's start with kernel level:

Kernel Level
======
* The interface is a portable Linux kernel module that can be easily adjusted to support any PCIe device
* To verify the correctness of the interface, I simulated an accelerator in QEMU. After a series of testing and debugging, the interface functions well.
* With the interface being done, the next question is - how to evaluate its performance? QEMU is a functional simulator, not providing us with timing information. So, gem5 is the best candidate.

Hardware level
======
* I started simulating the accelerator in gem5. 
  * Before doing that, I kept in mind what workloads I would like to optimize - memory-intensive workloads, of which Star-Schema Benchmark is a good candidate.
* Database accelerator simulation
  * I borrowed the design of Intel's Data Streaming Accelerator.
  * For the operations the accelerator support, I picked selection, projection and aggregation.
* Core design question: How to simulate the pipelined behavior of the hardware?
  * In QEMU, since the aim was to verify the correctness of the interface, I broke the execution of work into 3 stages: copy data to accelerator, process the data, then write the result back. But this is not what real hardware would do.
  * In gem5, I started thinking about how to simulate the hardware execution pipeline.
  * At what granularity can I abstract the execution pipeline?
    * Instruction level? No, since that would mean implementing a PCIe device along with some CPU properties, time-consuming. 
    * What I pick: functional level simulation. I broke each task into small units. For instance, if the task is: do a selection on data of length 100 where value of the data is less than x. I broke that up into smaller pieces named "query unit"s. Each query unit represents the smallest execution unit which a hardware functional unit can handle. Suppose we set the granularity to be 20, then there will be 5 query units which will be executing in a pipelined manner.
    * Each query unit has the same execution steps/stages, including 1. obtain virtual address of data 2. do address translation, 3. read data, 4. process data, 5. write data back, 6. ring the doorbell. In this way, different query units will be in its own stage, then transit to the next stage if there is available hardware resource. This is exactly the same as the hardware pipeline (fetch, decode, execute, memory access, writeback)!
    * Why my design is good? Because it allows us to tweak the different parameters to test different hardware configurations easily! I can manually set the process data stage latency to be 100 cycles, or 200 cycles, depending on what operation we are simulating. 
  * Correctness?
    * This is a good question regarding my design - how to prove that the latency we obtained make sense?
    * gem5 has a cool function called `schedule(event, latency)` i.e. scheduling an `event` `latency` ticks later. On completion of any event, there will be a callback function being invoked. I implemented a centralized scheduler which is being invoked on completion of any event, and that scheduler will always scan through all query units and see if they can transit to the next stage, with available hardware resource. So my design is guaranteed to be correct, and accurate, as long as the latencies are reasonable.
* The reason I need to simulate the accelerator is because I need to evaluate my interface, but as you can see, the simulation piece can already be a huge amount of work.

User level
======
* What's next? Workloads implementation
  * Now that we have the simulated accelerator and the interface, we need to run some workloads!
  * One thing I appreciate the modular design is that we can simply modify one layer without needing to modify other layers to get it work. However, in this work, it is not as simple as it is.
  * To verify the correctness of both the accelerator and interface, I already wrote different microbenchmarks, but that is not enough. Evaluation requires real-world workloads!
  * I need to go through the source code of the workload, understand it and figure out where to add the interface.
  * Of course, with endless debugging due to cross-layer design.

What brings me with this experience?
======
* Systems design skills:
  * From the naive design to the pipelined design.
  * Cooperating all three layers to make sure they work properly.
  * Unit tests all the time - make sure the test covers as many edge cases as possible.
* Strong systems programming skills:
  * I wrote a lot of code.
  * I deleted a lot of code (due to false design).
  * There is still a lot of code left.
  * I don't know how many lines of C and C++ code I have written :).
* Strong debugging skills.
  * High cost of making a mistake - each run takes around 10 minutes on microbenchmark, 1 hour for real-world workload. I need to find a fast way to locate the bug with a minimal reproducible example.
  * Debugging tools? I cannot run GDB in gem5's emulated environment since it is too slow.
  * How did I debug? I can only use `printf()` and analyze the trace, due to the concurrent execution of the query units.
* The ability to work with large code bases including Linux, and gem5.
<!-- This is the front page of a website that is powered by the [Academic Pages template](https://github.com/academicpages/academicpages.github.io) and hosted on GitHub pages. [GitHub pages](https://pages.github.com) is a free service in which websites are built and hosted from code and data stored in a GitHub repository, automatically updating when a new commit is made to the respository. This template was forked from the [Minimal Mistakes Jekyll Theme](https://mmistakes.github.io/minimal-mistakes/) created by Michael Rose, and then extended to support the kinds of content that academics have: publications, talks, teaching, a portfolio, blog posts, and a dynamically-generated CV. You can fork [this repository](https://github.com/academicpages/academicpages.github.io) right now, modify the configuration and markdown files, add your own PDFs and other content, and have your own site for free, with no ads! An older version of this template powers my own personal website at [stuartgeiger.com](http://stuartgeiger.com), which uses [this Github repository](https://github.com/staeiou/staeiou.github.io).

A data-driven personal website
======
Like many other Jekyll-based GitHub Pages templates, Academic Pages makes you separate the website's content from its form. The content & metadata of your website are in structured markdown files, while various other files constitute the theme, specifying how to transform that content & metadata into HTML pages. You keep these various markdown (.md), YAML (.yml), HTML, and CSS files in a public GitHub repository. Each time you commit and push an update to the repository, the [GitHub pages](https://pages.github.com/) service creates static HTML pages based on these files, which are hosted on GitHub's servers free of charge.

Many of the features of dynamic content management systems (like Wordpress) can be achieved in this fashion, using a fraction of the computational resources and with far less vulnerability to hacking and DDoSing. You can also modify the theme to your heart's content without touching the content of your site. If you get to a point where you've broken something in Jekyll/HTML/CSS beyond repair, your markdown files describing your talks, publications, etc. are safe. You can rollback the changes or even delete the repository and start over -- just be sure to save the markdown files! Finally, you can also write scripts that process the structured data on the site, such as [this one](https://github.com/academicpages/academicpages.github.io/blob/master/talkmap.ipynb) that analyzes metadata in pages about talks to display [a map of every location you've given a talk](https://academicpages.github.io/talkmap.html).

Getting started
======
1. Register a GitHub account if you don't have one and confirm your e-mail (required!)
1. Fork [this repository](https://github.com/academicpages/academicpages.github.io) by clicking the "fork" button in the top right. 
1. Go to the repository's settings (rightmost item in the tabs that start with "Code", should be below "Unwatch"). Rename the repository "[your GitHub username].github.io", which will also be your website's URL.
1. Set site-wide configuration and create content & metadata (see below -- also see [this set of diffs](http://archive.is/3TPas) showing what files were changed to set up [an example site](https://getorg-testacct.github.io) for a user with the username "getorg-testacct")
1. Upload any files (like PDFs, .zip files, etc.) to the files/ directory. They will appear at https://[your GitHub username].github.io/files/example.pdf.  
1. Check status by going to the repository settings, in the "GitHub pages" section

Site-wide configuration
------
The main configuration file for the site is in the base directory in [_config.yml](https://github.com/academicpages/academicpages.github.io/blob/master/_config.yml), which defines the content in the sidebars and other site-wide features. You will need to replace the default variables with ones about yourself and your site's github repository. The configuration file for the top menu is in [_data/navigation.yml](https://github.com/academicpages/academicpages.github.io/blob/master/_data/navigation.yml). For example, if you don't have a portfolio or blog posts, you can remove those items from that navigation.yml file to remove them from the header. 

Create content & metadata
------
For site content, there is one markdown file for each type of content, which are stored in directories like _publications, _talks, _posts, _teaching, or _pages. For example, each talk is a markdown file in the [_talks directory](https://github.com/academicpages/academicpages.github.io/tree/master/_talks). At the top of each markdown file is structured data in YAML about the talk, which the theme will parse to do lots of cool stuff. The same structured data about a talk is used to generate the list of talks on the [Talks page](https://academicpages.github.io/talks), each [individual page](https://academicpages.github.io/talks/2012-03-01-talk-1) for specific talks, the talks section for the [CV page](https://academicpages.github.io/cv), and the [map of places you've given a talk](https://academicpages.github.io/talkmap.html) (if you run this [python file](https://github.com/academicpages/academicpages.github.io/blob/master/talkmap.py) or [Jupyter notebook](https://github.com/academicpages/academicpages.github.io/blob/master/talkmap.ipynb), which creates the HTML for the map based on the contents of the _talks directory).

**Markdown generator**

I have also created [a set of Jupyter notebooks](https://github.com/academicpages/academicpages.github.io/tree/master/markdown_generator
) that converts a CSV containing structured data about talks or presentations into individual markdown files that will be properly formatted for the Academic Pages template. The sample CSVs in that directory are the ones I used to create my own personal website at stuartgeiger.com. My usual workflow is that I keep a spreadsheet of my publications and talks, then run the code in these notebooks to generate the markdown files, then commit and push them to the GitHub repository.

How to edit your site's GitHub repository
------
Many people use a git client to create files on their local computer and then push them to GitHub's servers. If you are not familiar with git, you can directly edit these configuration and markdown files directly in the github.com interface. Navigate to a file (like [this one](https://github.com/academicpages/academicpages.github.io/blob/master/_talks/2012-03-01-talk-1.md) and click the pencil icon in the top right of the content preview (to the right of the "Raw | Blame | History" buttons). You can delete a file by clicking the trashcan icon to the right of the pencil icon. You can also create new files or upload files by navigating to a directory and clicking the "Create new file" or "Upload files" buttons. 

Example: editing a markdown file for a talk
![Editing a markdown file for a talk](/images/editing-talk.png)

For more info
------
More info about configuring Academic Pages can be found in [the guide](https://academicpages.github.io/markdown/). The [guides for the Minimal Mistakes theme](https://mmistakes.github.io/minimal-mistakes/docs/configuration/) (which this theme was forked from) might also be helpful. -->
