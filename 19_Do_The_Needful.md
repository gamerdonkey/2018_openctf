## The Challenge

This challenge starts as a file download of `DoTheNeedful-98e4c6ba71f88e4201a08e7503b0df6124607e39`, which running `file` tells us is a TAR archive.

After untarring the file, we end up with a plain text file containing the following:

    =AAAAMjU/o7Z+0V17r06KDNmaZHQB1VSlR7wsTDuNk1ok3wfRPMl5YAAV/DwDzAIAERyH3wAAsVVGNBAIs4H

## The Solution

The character set is recognizable as base64, but the equals sign at the beginning indicates that something is wrong. Since the equals sign is used as padding, it normally appears at the end of a base64 encoded string. So, our string looks backward.

Handily enough, there is a bash command, `rev`, that we can use to simply reverse the string before decoding it, and that way we can quickly check on that backwards base64 hunch.

    # cat Challenge.txt | rev | base64 -d

That decodes successfully, which is a good sign, but the output looks like a bunch of junk.

No worries, though. Junk can be useful. We just neeed to know what kind of junk we have. But we should run a quick `reset` command first, because binary output on the console can lead to unexpected behavior. 

So after our reset, let's put our junk in a file and run `file` again to see what it is.

    # cat Challenge.txt | rev | base64 -d > Challenge.out
    # file Challenge.out 
    Challenge.out: gzip compressed data, last modified: Mon Jul 23 03:05:55 2018, from Unix, original size 51Challenge.out

Ooh, gzip data. That's easy to deal with.

    # cat Challenge.txt | rev | base64 -d | gunzip 
    466c61677b6577373332386866386573676839663233677d0a

Now it looks like we have somthing. All of those characters look like hexadecimal values, and experienced observers might recognize that last `0a` as the ASCII linefeed character, used for newlines on unix systems.

So, this is probably a string of ASCII characters, just in hexadecimal. All we need to do is use a tool like CyberChef or reference a lookup table to decode it into:

    Flag{ew7328hf8esgh9f23g}

And that pretty definitely looks like the flag.
