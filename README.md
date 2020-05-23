"# LEGOScraper" 
Web Scraping using Scrapy and Python 3

Now I want to start off and talk a little about scraping. Web scraping has actually been around for a very long time, but certain companies like craigslist have filed a lot of lawsuits due to people stealing data or crashing servers constantly by scraping-which has made people go to using API’s more. Now that does sound scary, especially when you hear the words lawsuit right? Well there are certain things you are able to do WITHOUT getting yourself into trouble and we will talk about a few of those today! 
So, what is Web scraping, well the term usually used in the industry is web crawling or Spidering…but what does it all mean you ask? Well To put it simple it is literally just going over a collection of web pages and extracting the data. This is an extremely powerful tool when you are working with data on the web.  With this tool you can get large amounts of data from mining certain products WITHOUT the use of an API!
What we will be going over tonight will be learning the basic fundamentals of the scraping/spidering process and we are going to be using a site that deals with my daughters FAVORITE toys….LEGOS!!!! We will be using a site called Brickset and we will be extracting the data in regard to HUNDREDS of LEGO sets, and displaying it to your screen! 

for all of this to run properly you will need Python 3 installed and anaconda. Since most of you probably don’t have anaconda installed I will need everyone to head over to this site
https://docs.anaconda.com/anaconda/install/
This is called anaconda, I have been using this for YEARS now, you can run the program later and explore more but our main focus this evening is getting the anaconda installed and allowed to be open through VS code ( the steps are perfectly explained and simple) Once you have this installed close out your VS code and reopen it and we will then begin! 
CREATING THE WEB CRAWLER
Now to begin you have to always remember how scraping works
first you have to build “something” that will be able to find and download the pages
and secondly you want to add to that build so you can then extract the information
So now that we have that in mind lets start
Scrapy is one of the most awesome and powerful Python crawling libraries in python. Think of it as more of the “Batteries included” approach of crawling(meaning it handles the main functionality for you so you as the developer do not have to keep rebuilding each time! It’s actually a pretty straight forward process! Scrapy is located in the PyPi store https://pypi.org/   a site if you didin’t know that holds THOUSANDS of python packages on this community-owned repo! 
Now if you have python3 installed you will have pip installed which is going to what we use to install scrapy!
So lets go ahead and create the project directory( and if you are new to programing think of a directory as a folder on your desktop) 
So open up your terminal in VS code by pressing ctrl+`  (control+tilda)  and if you are on windows run 
mkdir legoset-scraper
now run the command cd legoset-scraper     (cd stands for change directory and then place the name of the directory we just made) and what this does is now places our whole environment in this directory! AWESOME RIGHT!!! Let’s keep going

Inside of this directory at the top lets go ahead and create a file called scraper.py
We’ll start by making a very basic scraper that uses Scrapy as its foundation. To do that, we’ll create a Python class that subclasses scrapy.Spider, a basic spider class provided by Scrapy. This class will have two required attributes:
name — just a name for the spider.
start_urls — a list of URLs that you start to crawl from. (We will start with one URL.)

import scrapy
class BrickSetSpider(scrapy.Spider):
    name = "brickset_spider"
    start_urls = ['http://brickset.com/sets/year-2020']


Next, we take the Spider class provided by Scrapy and make a subclass out of it called BrickSetSpider. Think of a subclass as a more specialized form of its parent class. The Spider subclass has methods and behaviors that define how to follow URLs and extract data from the pages it finds, but it does not know where to look or what data to look for. By sub-classing it, we can give it that information. Finally, we give our scraper a single URL to start from: http://brickset.com/sets/year-2016. If you open that URL in your browser, it will take you to a search results page, showing the first of many pages containing LEGO sets. Now let’s test out the scraper. You typically run Python files by running a command like python path/to/file.py. However, Scrapy comes with its own command line interface to streamline the process of starting a scraper. Start your scraper with the following command:
in your terminal now I want you to watch what I do at the end of the line start_urls press 
shift+enter           -----and what this will do is fire up anaconda in your terminal if it doesn’t fire  up then it’s okay simply run the command
pip install Scrapy -----this will go out and install the Scrapy library from the Pypi store now run
scrapy runspider scraper.py

So what happened here, so when we ran the scraper it loaded all the additional components and extensions it needed while reading all of the data from the site URL we provided
it then used that same link in the start_urls list, it made a digital copy of the HTML in the background (sort of how a web browser works) and then passed all of the HTML it copied to the parse method (which since a parse method was never created by us the spider literally finishes it without doing anything else!
Now that was okay but lets do one better, lets actually bring back the data from that site

I will open the html src from the inspect tool and we will see some headers literally on every page we are looking at and then the search data including what we are matching to! And literally everything looks like an Ordered list! 
So we will need to now extract each set of data pertaining to the LEGO sets 
and then for each LEGO set we want to pull the data down 

Scrapy lets us grab all sorts of data based on selectors which are simple just the patters we use to find the elements on the page. We can use CSS selectors or XPath selectors. We will use CSS selectors. Since we are looking for a specific class we will use .set for our CSS selector and after that we will just pass that specific selector into the response object.

def parse(self, response):
        SET_SELECTOR = '.set'
        for brickset in response.css(SET_SELECTOR):
            

This code grabs all the sets on the page and loops over them to extract the data. Now let us extract the data from those sets so we can display it. The brickset object we are looping over has its own css method, so we can pass in a selector to locate child elements. So now below the response lets type

              NAME_SELECTOR = 'h1 ::text'
              yield {
                  'name': brickset.css(NAME_SELECTOR).extract_first(),
              }


Some things you might have questions about lets circle back to the code now, you will notice in our name selector that we have some weird code ::text    what we are doing here is appending ::text to our selector (CSS pseudo-selector), and all it is doing is grabbing all of the text INSIDE the a tag instead of the tag itself
And then the extract_first() is being called on the object brick_set.css line, because we want the first element that matched the selector which will return us a string instead of a bunch of elements we can’t work with! 
Save the file and run 
scrapy runspider scraper.py 
AND LOOK AT THAT…. Notice the output has names now! 
Now I would like to now add new selectors for images, miniature figures or otherwise called minifigs that are coming with our set. The image for the set is stored in the src attribute of an img tag inside an a tag at the start of the set. We can use another CSS selector to fetch this value just like we did when we grabbed the name of each set. Getting the number of pieces is a little trickier. There’s a dt tag that contains the text Pieces, and then a dd tag that follows it which contains the actual number of pieces. We’ll use XPath, a query language for traversing XML, to grab this, because it’s too complex to be represented using CSS selectors. Getting the number of minifigs in a set is like getting the number of pieces. There is a dt tag that contains the text Minifigs, followed by a dd tag right after that with the number.

            NAME_SELECTOR = 'h1 ::text'
            PIECES_SELECTOR = './/dl[dt/text() = "Pieces"]/dd/a/text()'
            MINIFIGS_SELECTOR = './/dl[dt/text() = "Minifigs"]/dd[2]/a/text()'
            IMAGE_SELECTOR = 'img ::attr(src)'
            yield {
                'name': brickset.css(NAME_SELECTOR).extract_first(),
                'pieces': brickset.xpath(PIECES_SELECTOR).extract_first(),
                'minifigs': brickset.xpath(MINIFIGS_SELECTOR).extract_first(),
                'image': brickset.css(IMAGE_SELECTOR).extract_first(),
            }

Press ctrl+s to save your work and lets run the scraper command once again and see what output we get
scrapy runspider scraper.py

Look at the data now we are returning the name the image the minifigs WOW nice work everyone! 
BUT WE AREN”T DONE YET! 
Now lets modify our scraper so that our spider can follow all the links! 
This is commonly called Crawling Multiple Pages and this is out last part


So to break down what we have done so far we extracted all the data from the initial site, but what if we wanted to go to page 2 or 3 or 4 ?how would we do it? The whole point of the spider is to traverse through the links to the other pages and grab all of that data as well. If we go to the html you will notice that there is a greater than sign/arrow pointing right  >     after every page, which gives us a hint on our last step. All we have to do now is tell it “follow the link, and if it is going on to another page open that next page and continue extracting data” 


        NEXT_PAGE_SELECTOR = '.next a ::attr(href)'
        next_page = response.css(NEXT_PAGE_SELECTOR).extract_first()
        if next_page:
            yield scrapy.Request(
                response.urljoin(next_page),
                callback=self.parse
            )


First, we define a selector for the “next page” link, extract the first match, and check if it exists. The scrapy.Request is a value that we return saying “Hey, crawl this page”, and callback=self.parse says “once you’ve gotten the HTML from this page, pass it back to this method so we can parse it, extract the data, and find the next page.“ This means that once we go to the next page, we’ll look for a link to the next page there, and on that page we’ll look for a link to the next page, and so on, until we don’t find a link for the next page. This is the key piece of web scraping: finding and following links. In this example, it’s very linear; one page has a link to the next page until we’ve hit the last page, But you could follow links to tags, or other search results, or any other URL you’d like. Now, if you save your code and run the spider again,
scrapy runspider scraper.py

You will see that it doesn’t just stop once it iterates through the first page of sets. It keeps on going and In the grand scheme of things, it’s not a huge chunk of data, but now you know the process by which you automatically find new pages to scrape.
Now last and final command lets run
scrapy runspider scraper.py -o file.csv 
and what this will do is create a csv file inside of our directory than you can see all of the data that we just extracted from the site! Simply highlight and copy and paste one of the link images into your browser and notice it is correct and working! 
