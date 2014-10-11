---
layout: post
title:  "Parsing Amazon with PyQuery and requests"
date:   2014-10-11 11:35:13
categories: python pyquery parsing
---
The company that I'm working for wanted us to write a project which includes parsing the Amazon pages, specifically the book pages. We needed to extract the information of books like title, authors, cover image, price, publisher, etc. from a given url.

There are lots of parsing and scraping libraries in Python world. The most popular ones are [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/), [HtmlParser](https://docs.python.org/2/library/htmlparser.html) and [Scrapy](http://scrapy.org/). But I needed something which has rather slightly gentle learning curve because I just wanted to get things done as quickly as possible.

With the advice of a colleague, I discovered [PyQuery](https://pypi.python.org/pypi/pyquery) which is a jquery-like library which allows you to access html elements with their types, ids and classes. I jumped right into it, because I thought using this will bring me some speed as a guy who has a jquery background even just a little. 

We can divide this problem of parsing Amazon in to two different parts. The first one is getting the data and the second is processing over that data to get information out of it.

Making A Request
----------------
Python comes with a handy url opener called [urllib2](https://docs.python.org/2/library/urllib2.html). But I decided to use [requests](http://docs.python-requests.org/) for this little project just because to see its interface and to get involved with it. The thing that we are going to use requests for is pretty straightforward and it could've been done easily with urllib2 too. But again, it's your decision to choose whatever you prefer. By the way requests make a difference in the API side when comparing to urllib2.

First we need to make a request to an Amazon link than get the html source of the web page. And that will be the data that we mentioned as a first step of this project. Let's gets started!

{% highlight python %}
import requests

# The url of the first book of the Harry Potter series as an example
url = "http://www.amazon.com/dp/059035342X/"

# Make a request and store the result in the variable called response
response = requests.get(url)

# Get the content of the response
html = response.content

# Print the source to see everything's right
print html
{% endhighlight %}

Oops! When we look closely at the output of the program which is the html source code we get from Amazon, we will see that something's not right. This is because Amazon thought that our request is made by a bot not an actual person. And it has a point, right? That's why it didn't send the original response, instead it sends a message that it detects that the request is not made form a web browser. In order to solve this problem, first let's take a look at the concept of header of a request.

All We Need is Header
---------------------
When we type some link in our browser and hit enter, our browser creates a packet that includes variety of information about us like response type that we are going to accept, cookie that is sent previously by the server, user agent string that identifies ourselves and many many more. This is called request header. The most vital one of its fields for us is user-agent. By injecting that information to our code, Amazon will think that our request is made by a real web browser. And now let's see what a header looks like.

If you're using Chrome, we can view that info by using Chrome Developer Tools. First start your Chrome browser and open developer tools window by pressing Ctrl + Shift + I (or Cmd + Opt + I on Mac). Go to Network tab and enter a random website. When the website starts to load, you will see network window will be filled with some data. Go on top of the list that is located on the left side and click the uppermost one. We will see the request header and its fields in this window.


[![Request header example](http://i.imgur.com/1guaWoI.png "Amazon request header")](http://i.imgur.com/1guaWoI.png)


Let's add this user-agent to our code and move on.

{% highlight python %}
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.94 Safari/537.36',
} # sorry pep8

url = "http://www.amazon.com/dp/059035342X/"
response = requests.get(url, headers=headers)
html = response.content

print html
{% endhighlight %}

The result is streaming like in The Matrix which is a good sign that indicates we're getting what we want.

And, Parsing Begins...
----------------------
I'm one of those who believes practicing is the best way to learn new things. That's why we are going to cover the second step of the project which is parsing the html by doing an example. Let's assume that we have an html file like the one below (I know it has some weird points in it) and we are going to extract all the fields it has. Let's start!

{% highlight html %}
<div class="book-info">
    <h2>Book Info</h2>
    <h2 id="title">Programming in Scala</h2>
    <ul id="author">
        <li id="main">Martin Odersky</li>
        <li>Lex Spoon</li>
        <li>Bill Venners</li>
    </ul>
    <p class="center highlighted" id="money-field">$37.66</p>
    <p class="center">Paperback</p>
    <p class="center">January 4, 2011</p>
</div>
{% endhighlight %}

First of all, there's not exactly one way to extract the information we need. For the sake of learning different aspects of PyQuery, we are going to extract the information in different ways. First target: book title. (I'm going to use [IPython](http://ipython.org/) which is an advanced version of Python shell. It has tab completion and you can run system shell commands, and many more cool features.)

{% highlight python %}
In [1]: from pyquery import PyQuery as pq

In [2]: html = '''
  ...:     <div class="book-info">
  ...:         <h2>Book Info</h2>
  ...:         <h2 id="title">Programming in Scala</h2>
  ...:         <ul id="author">
  ...:             <li id="main">Martin Odersky</li>
  ...:             <li>Lex Spoon</li>
  ...:             <li>Bill Venners</li>
  ...:         </ul>
  ...:         <p class="center highlighted" id="money-field">$37.66</p>
  ...:         <p class="center">Paperback</p>
  ...:         <p class="center">January 4, 2011</p>
  ...:     </div>
  ...: '''

In [3]: doc = pq(html)

In [4]: doc('h2') # we can access the elements by giving element type like in jquery. As a result we have a list of elements with h2 tag.
Out[4]: [<h2>, <h2#title>]

In [5]: doc('h2#title') # since the book title has id 'title' we can narrow down the query.
Out[5]: [<h2#title>]

In [6]: doc('h2#title').text() # we need to call 'text' method to get the content, without it we get object representation.
Out[6]: 'Programming in Scala'

In [7]: doc('h2').eq(1).text() # since there are two elements with h2 tag we can get the second one with eq method. It accepts 0-based index as a parameter.
Out[7]: 'Programming in Scala'

In [8]: doc('#title').text() # since there are no other elements with id 'title', it is sufficient in the query just by itself.
Out[8]: 'Programming in Scala'
{% endhighlight %}

We did pretty good job on getting title. Now our mission is to extract the authors. This is a little bit different because it has a nested structure.

{% highlight python %}
In [9]: doc('ul').children() # get the children elements of 'ul'
Out[9]: [<li#main>, <li>, <li>]

In [10]: doc('ul').children().eq(0).text() # we can combine it with 'eq' to say get the element with index zero.
Out[10]: 'Martin Odersky'

In [11]: doc('ul').children().text() # we can get the content of all children like this. But as you can see we didn't take the authors separately. Instead we took as a whole string. We may want to change our approach.
Out[11]: 'Martin Odersky Lex Spoon Bill Venners'

In [12]: for li in doc('ul').children(): # the solution is iterating all children with a for loop.
	print li.text
	.....:
Martin Odersky
Lex Spoon
Bill Venners

In [13]: [li.text for li in doc('ul').children()] # we can add all authors to a list in a very pythonic way.
Out[13]: ['Martin Odersky', 'Lex Spoon', 'Bill Venners']

In [14]: doc('ul#author') # let's say there are many ul elements in an html file. We can specify it by adding its id to the query.
Out[14]: [<ul#author>]

In [15]: doc('ul').find('li#main').text() # we can find nested elements with 'find'
Out[15]: 'Martin Odersky'

In [16]: doc('ul li#main').text() # or with less verbose by leaving space between elements.
Out[16]: 'Martin Odersky'
{% endhighlight %}

Authors: done. Now we are moving on to price of the book.

{% highlight python %}
In [17]: doc('p.center') # there are three <p> tags with the class 'center' but we need the one with 'money-field'
Out[17]: [<p#money-field.center.highlighted>, <p.center>, <p.center>]

In [18]: doc('p.center').filter('#money-field').text() # for that reason we need to filter out the one with 'money-field'. That's where 'filter' gives a hand.
Out[18]: '$37.66'

In [19]: doc('p.center.highlighted').text() # or we can get price easily by giving two class names.
Out[19]: '$37.66'
{% endhighlight %}

Up to this point, we learned a lot. We accessed elements through their types, ids, classes also indices, we handled the nested structures, we filtered out the results to get what we want, etc. 

I'm aware that we didn't do the extraction of last two pieces of information: cover type and publishing date. I want to leave those to you as a little task to learn PyQuery better.

How About Parsing Amazon?
-------------------------
Now that we have the data and practical knowledge to process it, parsing Amazon becomes relatively easier. All we need to do is analyzing the html source code and finding the patterns to extract book information. In other words, we have to find the tags (it will be possibly nested and not as simple as our example) that a particular book info got enclosed. 

Moving on with the previous Harry Potter url, when we inspect the html of the page, we are going to see the line where title is located at:

{% highlight html %}
<span id="productTitle" class="a-size-large">Harry Potter and the Sorcerer's Stone (Harry Potters)</span>
{% endhighlight %}

This line for the author:

{% highlight html %}
<a data-asin="B000AP9A6K" class="a-link-normal contributorNameID" href="/J.K.-Rowling/e/B000AP9A6K/ref=dp_byline_cont_book_1">J.K. Rowling</a>
{% endhighlight %}

And this line for the price:

{% highlight html %}
<span class="a-size-medium a-color-price offer-price a-text-normal">$6.92</span>
{% endhighlight %}

Let's write the whole code to extract those information.

{% highlight python %}
from pyquery import PyQuery as pq
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.94 Safari/537.36',
}

def get_book_info(url):
    response = requests.get(url, headers=headers)
    doc = pq(response.content)

    book = {}

    title = doc('#productTitle').text()
    book['title'] = title

    author = doc('.contributorNameID').text()
    book['author'] = author

    price_text = doc("span.offer-price.a-color-price").text()
    price = float(price_text[1:]) # get rid of the dollar sign
    book['price'] = price

    return book

def main():
    print get_book_info('http://www.amazon.com/dp/059035342X/')

if __name__ == "__main__":
    main()

# output: {'price': 6.92, 'author': 'J.K. Rowling', 'title': "Harry Potter and the Sorcerer's Stone (Harry Potters)"}
{% endhighlight %}

That's it! Finally we achived what we want. But we didn't cover the all the scenerios. For example, there are books written by more than one authors. In that case, our code will find only one of them. Also we didn't extract the cover image, publishing date, publisher, etc. These won't be a big problem, because now we know the tools to implement those too. For the sake of keeping the post no more longer I won't go over those details. 

I hope you find the post helpful. Please share your thoughts and everyhing on the comment section below. Have a good one :)
