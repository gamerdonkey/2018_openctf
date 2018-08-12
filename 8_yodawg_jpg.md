# The Challenge

This challenge starts off with an important clue and a link to download a file.

    yodawg.jpg 50 ---
    steghide was found on the hackers computer.  https://scoreboard.openctf.com/yodawg.jpg-a5f90bcb58c65886c8b40623ad5bf73ae62545bc

Our first clue is actually the name of the challenge. "yodawg" is almost certainly a [reference to the meme](https://knowyourmeme.com/memes/xzibit-yo-dawg), so we can expect to have something repeated put into something else.

The other clue is the reference to "steghide", which we'll get to later.

After downloading from the link, we can use the `file` command to find out what we are working with.

    # file yodawg.jpg-a5f90bcb58c65886c8b40623ad5bf73ae62545bc 
     yodawg.jpg-a5f90bcb58c65886c8b40623ad5bf73ae62545bc: gzip compressed data, last modified: Fri Aug 10 03:52:47 2018, from Unix, original size 389120

gzip files usually like to have the the correct file suffix, so we need to rename the file before running it through the command line unzipping tool. After that, we wind up with a tar file that needs to be extracted.

    # gunzip yodawg.jpg-a5f90bcb58c65886c8b40623ad5bf73ae62545bc
     gzip: yodawg.jpg-a5f90bcb58c65886c8b40623ad5bf73ae62545bc: unknown suffix -- ignored
    # mv yodawg.jpg-a5f90bcb58c65886c8b40623ad5bf73ae62545bc yodawg.out.gz
    # gunzip yodawg.out.gz
    # file yodawg.out 
    yodawg.out: POSIX tar archive (GNU)
    # tar xvf yodawg.out
    yodawg.jpg

Finally, we have an image that looks something like this:

![Not the actual image, just what it looks like](https://upload.wikimedia.org/wikipedia/commons/thumb/0/04/Flag_of_Gabon.svg/400px-Flag_of_Gabon.svg.png "Not the actual image, just what it looks like.")

## How It Should Be Solved

Seeing nothing interesting about the image itself, we can Google for "steghide" and find that it is a tool used for hiding messages and data inside of inconspicuous-looking files. That sounds super interesting in a CTF, so we can install the program on our computer and try it out.

    steghide info yodawg.jpg 
    "yodawg.jpg":
      format: jpeg
      capacity: 24.5 KB
    Try to get information about embedded data ? (y/n) y
    Enter passphrase: 
    steghide: could not extract any data with that passphrase!

Interesting! But we can't get any info about hidden data in our image file without the passphrase (blank didn't work here).

So let's look back at the image we have. The three brightly-colored bars appears a lot like the flags of several European nations, and putting a "flag" in a "capture the flag" event seems like the exact kind of thing that a contest organizer would do at DEFCON, so let's look into that.

Referencing the [Wikipedia Gallery of Sovereign State Flags](https://en.wikipedia.org/wiki/Gallery_of_sovereign_state_flags) reveals that the flag of Gabon is an exact match. So let's try that!

    # steghide info yodawg.jpg 
    "yodawg.jpg":
      format: jpeg
      capacity: 24.5 KB
    Try to get information about embedded data ? (y/n) y
    Enter passphrase: Gabon
      embedded file "IHeardYouLikeFlags.jpg":
        size: 8.8 KB
        encrypted: rijndael-128, cbc
        compressed: yes

And we're in! We can quickly extract the file with the steghide tool:

    # steghide extract -sf yodawg.jpg 
    Enter passphrase: Gabon
    wrote extracted data to "IHeardYouLikeFlags.jpg".

This image simply has a white background with the phrase "I heard you liked flags" in black letters. Trying that and the filename as the flag (CTF flag, not nation flag) doesn't work, so we're not quite done.

Now is when the "yodawg" part kicks in. We had some stego, so maybe the author put some stego in our stego (so we can stego while we stego). Fortunately, we can use steghide right away to check.

    # steghide info IHeardYouLikeFlags.jpg 
    "IHeardYouLikeFlags.jpg":
      format: jpeg
      capacity: 365.0 Byte
    Try to get information about embedded data ? (y/n) y
    Enter passphrase: 
      embedded file "so.txt":
        size: 63.0 Byte
        encrypted: rijndael-128, cbc
        compressed: yes

And this time a blank passphrase works! Let's get that extracted:

    # steghide extract -sf IHeardYouLikeFlags.jpg 
    Enter passphrase: 
    wrote extracted data to "so.txt".

Finally, we end up with something that looks definitely like a flag.

    # cat so.txt 
    I_put_this_as_the_flag_in_a_flag_so_that_you_can_flag_the_flag.

## How I Solved It

When I first read the challenge, I somehow missed the reference to "steghide". But I did notice the "yodawg" reference, so for around an hour I worked under the assumption that a jpg was somehow embedded in the jpg.

After wasting lots of time on that, I finally noticed that I needed to use steghide. Due to a typo, however, I did not find it in my Linux distribution's package repository.

    # apt search stegide

I've seen other CTFs use esoteric software, so I didn't think much of it an hunted for downloads through Google. Sadly, there were no Debian-style packages for download from the official Steghide site, but they did distribute the source and Redhat RPM packages. So, I decided to compile it myself.

This was a mistake.

After finding and installing the depencies, I still ran into a confusing compilation error that Google was no help for. As my frustration grew, I ran across a tool to convert software packages from one distribution to another. So I decided to try that.

This was a mistake.

The tool took many long minutes to download, and it threw a Perl error with the RPM package when I did get it to run. At this point, I was lost and ready to give up.

But, salvation.

On one of my many Google searches, I found that there were steghide packages in the official Debian repositories. This set my brainwheels turning, and I was suddenly aware of my earlier typo.

    # apt search steghide
    Sorting... Done
    Full Text Search... Done
    steghide/kali-rolling,now 0.5.1-12 amd64
      steganography hiding tool

After laying my head on the table for a while, I installed the program and got back to work.

I figured out the Gabon flag part with no trouble, but for some reason I neglected to try a blank passphrase on `IHeardYouLikeFlags.jpg`.

I spent a long time researching white-background flags on Wikipedia (did you know that Afghanistan used to fly a blank white flag as its national flag?) with no results. 

I made a simple BASH-script password cracker that used some of the Kali dictionaries in an attempt to brute force the passphrase.

Finally, I asked the author of the challenge for a clue, and she told me I was making this harder than it should be. She also asked "What is a password you would use when you don't care about passwords?"

So I knew what to do. I sat down and tried like 18 more passwords before finally remembering that steghide accepts blank passwords.

And that is how I captured the flag.
