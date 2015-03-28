---
layout: post
title:  "Lisp: Smile On Faces"
date:   2015-03-28 21:57:55
categories: lisp functional programming 
---
At the beginning of this year, I came up with so many posts like _Why you should 
learn functional programming?_, _2015 is the year of functional programming_, 
_Functional programming will kick a lot of ass in 2015_. Because of all the hype, 
learning functional programming was in my mind as a future (or a procrastinated) plan.  

One day, I was accompanying my colleagues in the smoke break. One of them, the senior
one, started to talk about Lisp and he was very enthusiastic and excited while talking about it. 
He was saying that Lisp was about lists and processing of them. In fact, the name Lisp
was coming from the LISt Processing. During the whole conversation, there was a little smile on his
face. On the other hand, I was trying to to look like that I understand his words
by nodding my head. But actually I understood nothing. I had this feeling that I
should learn Lisp immediately to catch up but I guess, later on, I forgot I had
a plan like that and didn't dig Lisp.  

At another day in the office, I was getting book suggestions from a friend about
programming and computer science stuff. Even though I didn't ask for a functional
programming book specifically, one of his suggestions was [The Little
Schemer](http://www.amazon.com/The-Little-Schemer-4th-Edition/dp/0262560992).
He told me that if I wanted to get involved with functional programming,
that book could be a good start. He also emphasized that I could get a really good
understanding of recursion with the help of the book. That has to be the sign
that the universe wanted me to learn functional programming. I immediately ordered the
book from Amazon and started to wait for it to come. (The shipment from Amazon
may take a month, sometimes even more, since I live in Istanbul, Turkey.)  

![xkcd](http://i.imgur.com/wO8r22Q.png)  

After about a month later, there was a tech talk about metaprogramming. In the
meantime, my book had been arrived and I had read a few pages of it. Now, I had very
little information about the syntax and it was not that cryptic to me anymore.
Anyway, back to the tech talk. The scope of it was metaprogramming in Ruby, 
Lisp and Scala. I had never heard of metaprogramming before and I thought that 
it would be very educational to attend an event like this. But I thought
I should at least know the dictionary definition of metaprogramming before going.
Here is the first paragraph of the metaprogramming's wikipedia page:

> Metaprogramming is the writing of computer programs with the ability to treat 
programs as their data. It means that a program could be designed to read, 
generate, analyse and/or transform other programs, and even modify itself while running.

Cool. Now, I was ready to go the the tech talk!?  

Firstly the presenter gave brief information about Lisp before passing to metaprogramming
in Lisp. One thing I noticed was that he was also very excited and enthusiastic
about Lisp. While giving his talk he was having that smile on his face. C'mon, this
couldn't be coincidence. Why are people that mentioning about Lisp is having that
excitement and enthusiasm. I couldn’t find an answer at that moment.  

I continued to read the book. I was writing the codes in the book to my editor to
learn better. At some point, the code didn't work. There was a function, let's say `elemAt`
(the name was different on the book but I couldn’t remember), that takes a list and an 
index and returns the element in that index. I was using Racket, different implementation
of Scheme. Apparently, the name of the function was different on Racket. Then, I
thought "Hey, this is not a problem. I can write my own `elemAt` and I didn't
even feel the need of looking up on the internet to find what is the equivalent
of that function on Racket. After some keystrokes, this happened:  

{% highlight lisp %}

(define elemAt
  (lambda (lat i)
    (cond
      ((null? lat) (quote err))
      ((eq? i 0) (car lat))
      (else (elemAt (cdr lat) (sub1 i))))))

{% endhighlight %}

I re-ran the program and it worked. Then I realized that I have the same weird
smile on my face like the other two guys have. The code I wrote was not too complex
(I know, I know it's not complete, either. I doesn't work on negative indices). So, why
do I feel like I achieved something? I guess it's because even if I was a rookie, I
wrote a built-in kind of function. That was a different kind of experience for
somebody who has only imperative language background.  

Now, I am a great fan of Lisp. The idea of building complex things by starting with 
its simple components is marvellous. Also with just a few mathematical and list operations,
you can make a lot of things like in the `elemAt` case. Also, as you can see, simplicity 
was the one of the things that got me.   

If you only have imperative language background like I was before, I strongly
recommend you to check functional programming and Lisp.
