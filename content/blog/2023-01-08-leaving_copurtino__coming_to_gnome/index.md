---
title: Leaving Copurtino, coming to Gnome
draft: false
subtitle: A story about the journey which led me to the Gnome Desktop Environment.
pubdate: '2023-01-08T00:00:00.000Z'
type: artikel
todo:
  - make more english friendly
image: ''
categories:
  - gnome
  - technology
  - awesomewm
  - st
  - dwm
  - english
recept:
  tijd:
    bereidingstijd: ''
    werktijd: ''
  ingrediënten: []
  calculator:
    plusfactor: 0
    start: 0
    end: 0
    meervoud: ''
    enkelvoud: ''

---

<style>
figure img{
  border-radius: 2px;
}
</style>

> This is the 1st episode of a series of articles about my computer desktop
> setup.

In the little village of Port Lligat with a overwhelming sea view hidden behind
tiny islands and rocky capes, Salvador Dali found the piece he needed to create
many of his intriguing paintings and sculptures. It is this dreamy place I too
found the piece to start writing about my Gnome setup which grew upon me the
last 2 years.


{{< img "imgs/Port-Lligat.png" "The bay of Port Lligat" >}}

I've been a long time MacOS user, and though it's a religious thing to say,
I think MacOS was the best desktop environment for users who work with
almost every interface a computer has to offer. When creating graphical design
or content, the many small graphical utilities and the fanatic
drag-and-drop-everywhere-you-can-philosophy, prevents your thinking and
creating are being slowed down. This can bring you in a magical creative flow.
When work involves text input like programming together with testing your
application, MacOS had a great mouseless workflow too. Vim'ing here cmd-tab'ing
there, Apple helped me coming in hyperfocus paradise many times.

{{< img "imgs/macos-screenshot-area-tool.jpg" "MacOS X screen capture tool also useful measuring pixels of an area" >}}

I don't exactly know when it started, but my appreciation for Apple's computer
interface slowly drifted away. It started with a slight feeling of discomfort, but
ended in a angry rebellic feeling of being prisoner in a country that has
changed while I was not aware there was revolution going on. One day I
woke up and realized this is my country anymore.

{{< img "imgs/macos-not-my-home-anymore.png" "Apple believes everything should look like an iPhone" >}}

And so I had to search for a new computer interface country. Obviously this
will be some kind of open source desktop environment, running on Linux. Living
in the open source world bring completely different rules to live by. You can't
expect some software ecosystem to perfectly match all your desires all at once.
You have to try staying somewhere for a while, fix some missing pieces. If it
feels good you can stay a little longer and of course if you think there is no
future for you, you can travel further. Don't be disappointed about the waste
of time. Every place has it's own unique ideas and you can take them with you
everywhere you go. That is the beauty of ideas. They bring fortune and
happiness and they travel with you without any costs.

Enough philosophic blabla for now. Let me tell you how I came to Gnome and what
I needed to change to feel at home. As I overate myself at the endless buffet
in Apple's restaurant, my journey began with a great need for purism and
simplicity. Less is more. This brought me straight to suckless.org with their
idea's of bloat-free software.

The best spot to start my journey and a great source of inspiration which still
has influence on how I see software. Knowing that it's possible to compose your
day to day computer environment with a small collection of self compiled
minimalistic tools makes you feel powerful and brings back the proportions of
how complex a desktop environment really is. [DWM](https://dwm.suckless.org),
their suckless window manager, was not suitable for me though. I realized I do
need more visual sugar while pressing keys and dragging my mouse for doing something
useful. I had to move on, but I did take something really important with me.
The Suckless Terminal emulator.

{{< img "imgs/dwm.png" "DWM from suckless.org" >}}

[Suckless Terminal](https://st.suckless.org) or `st` incorporates all
Suckless.org idea's about solving a problem once and for all without
implementing one function to many. My love for `st` is still growing every day
as it is one of my most used software programs and I never feel the need to
change it. I configured it once and it is perfect as it is. I might write more
about `st` but if you're interested in my configuration, you can checkout at [my
`st` setup](https://github.com/mipmip/nixos/blob/main/modules/desktop-st.nix).
It's a Nix derivation, ensuring this setup is completely reproducible, but
that's another story.

{{< img "imgs/suckless-terminal.png" "The Suckless Terminal, minimal yet complete" >}}

So I continued my search for a window manager or desktop environment that could
serv as my daily driver. I was inspired by DWM and I wanted to experiment more
with a tiling window manager, but I also learned that you need to have the
freedom to step out of the tiling system when doing specific work like graphic
design. DWM sells itself as a dynamic window Manager and this is exactly what a
dynamic window manager tries to achieve. I was on a good path. My next major
stop was in the land of AwesomeWM.

{{< img "imgs/awesome3.svg" "Yes awesome!" >}}

The [Awesome Window Manager](https://awesomewm.org/) started as a fork of DWM.
While DWM lets you modify the configuration by changing settings in a header
file and then recompile the program, AwesomeWM let's you configure everything
with Lua scripts. But it does not stop at changing some settings like keyboard
shortcuts, the developers of AwesomeWM created a complete widget API which
allows you to create and integrate everything you like with these same Lua
scripts. The core program does a good job as simple dynamic window manager with
some sane defaults, but the real power comes to light when you start using
scripts made by other AwesomeWM users or when you are brave enough to create
them yourself.

{{< img "imgs/awesome-first.png" "AwesomeWM default configuration" >}}

AwesomeWM was the first environment that served as being my home environment
after so many years of using macOS. I was back in control again and it felt
great. I really liked AwesomeWM and I did reach a point that I created some
widgets myself.

{{< img "imgs/awesomewm-well-tweaked.jpg" "AwesomeWM well tweaked by JavaCafe01" >}}

One strategy I followed when trying to get something achieved in AwesomeWM was
searching for screenshots by other AwesomeWM users. If a picture looked well
and could serve as example I looked for the configuration in the users dotfile
repository and tried to implement it myself. All AwesomeWM community
screenshots are submitted as comment in one big issue on the AwesomeWM github
project. This is not very intuitive to browse through, so I created an
[altenative gallery](https://mipmip.github.io/awesomewm-screenshots/) interface
making it much more easy to browse and search for fitting examples.

{{< img "imgs/awesomewm-screenshot-gallery.png" "Awesome Screenshot Gallery" >}}

Unfortunately it took me way to much time to just make some small improvements.
The AwewomeWM API is powerful but not very well organized and documented. I
spent many hours, reading documentation, searching for examples, running trail
and error test to achieve simple changes. My list with wishes and plain
irritations grew longer and longer until I came to realize I just could not
afford the time I needed to get the most important wishes scripted. With pain
in my heart I decided I had to to move on.

So now I really invested a lot of time in a piece of software that I had leave.
What were the things I learned? What idea's did I take with me? I did not know.
I was just tired of tweaking and trying getting things done and I longed
desperately for working with a nice looking polished desktop environment. I
needed a break.

Back to something I know that works and looks nice. Let try Gnome for a while
and get some rest with a desktop used by many people and trusted by the big
Linux brands.  Well I did not really know Gnome. I played with it sometimes when
installing some kind of Ubuntu stock distro, but besides the polished GTK
interface I was not very enthusiastic with the way things work in Gnome. No
icons on the desktop. The panels and menu's felt strange, the overview looked
great but was something I totally did not need (Gnome 40 just
arrived). One thing I could not ignore were the ugly round corners. Don't know
which purpose they had but I could not see them as corners but I saw them as
hooks. You might be familiar with the image of the young lady and the old
witch. Well I always say the witch.

{{< img "imgs/round-corner-or-hook.png" "Strange Gnome corner or hook" >}}
{{< img "imgs/young-lady-and-the-witch.png" "Is it a young lady or a witch?" >}}

O.K. let's find a way to get rid of these round corners and then just accept a
well thought user interface. Real work has to be done, come on! ...

_Pim: "How can I remove the round corners in Gnome?"_

_Internet: "Install the [remove-rounded-corners](https://extensions.gnome.org/extension/448/remove-rounded-corners/) extension"_

_Pim: "Gnome Shell Extension?"_

Gnome has extensions! Hundreds already. And they are written in JavaScript.
Neat! And I can change anything I want. That is so cool.

{{< img "imgs/yeah-gnome-i-like-you-a-lot.png" "Finally Gnome" >}}

So here starts a complete new chapter in my search for the ideal Computer
Desktop. Next article I'll tell more about my Gnome configuration, the
extensions I use and the extensions I created myself to get me a flexible
distraction free keyboard optimized workflow.
