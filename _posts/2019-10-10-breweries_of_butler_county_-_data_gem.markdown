---
layout: post
title:      "Breweries of Butler County - Data Gem"
date:       2019-10-10 19:00:49 +0000
permalink:  breweries_of_butler_county_-_data_gem
---


Welcome back to my blog! This entry is about a little CLI Data Gem that I've written that can display information about local breweries in Butler County, PA.  It's getting to be my favorite time of year to visit them, and so I figured this might be fun.

This is the first time I've done any sort of scraping and building objects from that information, and I knew this was going to be a great learning experience for future projects...

I decided to make a folder for bin (which would hold my file to run my program), a config folder (to hold my environment details), and a lib folder (which holds my files with most of the code).

My bin file (titled BCBeerCircuit) is a simple file.  It requires the environment file that I set up, and makes a call to a new CLI.

```
require_relative '../config/environment'

Cli.call
```

In my environment file, I decided to require Bundler (for the sake of taking care of my dependencies), pry (for helping me debug), nokogiri (for parsing my data into manageable pieces), and open-uri (which is part of the standard library, and is used for wrapping/opening URL's - http, etc.)  Also, you'll notice at the end I opted to require_all 'lib' here that way my program has access to ALL of my library files.

```
require 'bundler'
require 'pry'
require 'nokogiri'
require 'open-uri'

# class Concerns
# As we expand this gem...
#
# end

Bundler.require

require_all 'lib'

```

I might come back to this, and expand this all sometime soon...

Now we get to the real meat and potatoes here.  When we call a new Cli, this is the code that we start our CLI up with...

```
class Cli

  def self.call
    Scraper.get_companies
    input = ""
    while(input != 'exit') do
      puts ''
      puts "Welcome to the Butler County Beer Circuit"
      puts "Here is a list of our local County Breweries: "
      puts "(Select a number to find out more information!)"
      puts "(Enter 'exit' to exit program.)"
      Brewery.list_breweries
      input = gets.chomp

      if input != 'exit' && (input.to_i < 1 || input.to_i > Brewery.all.length)
        begin
          raise InvalidSelectionError
        rescue InvalidSelectionError => error
          puts error.message
        end
      else
        Brewery.brewery_info(input.to_i)
      end
    end
  end
  # def self.menu
  # 
  # end
  class InvalidSelectionError < StandardError
    def message
      "Invalid selection.  Please try again..."
    end
  end
end

```
I've created a simple menu, that generates a list of brewery objects, and then you  are allowed to enter which brewery you would like more information about, or you can exit the gem.  I put a check in place so that you don't enter any sort of input that would be deemed invalid (a number that doesn't exist on the list, or something other than 'exit').  Lastly, once you make a valid selection, it lists the information of the object such as a description, and a URL that will lead you to more information.

Lastly, I have a file that takes care of all the scraping of data and turns our data into Brewery objects.... Let's take a look.

```
class Scraper

  def self.get_companies
    url = open("https://www.visitbutlercounty.com/BeerCircuit")
    doc = Nokogiri::HTML.parse(url)

    doc.css("div.pane-content").css("div.col-md-3").each do |brewery, idx|
      name = brewery.css("h4").css("a").text.strip
      desc = brewery.css("p").text.gsub(/[\u200B]/, '') #Regex for removing the "<U+200B>" string that existed in the Recon Brewery description
      url = brewery.css("h4").css("a").attr("href").to_s
      Brewery.new(name, desc, url) if (!name.empty?)
    end
  end
  # 
  # def self.company_data(url)
  # 
  # end

end

```

This was a tough one... Finding the right elements that you can iterate through to get the right data can be tough.  Here I had to do some manipulations to make it just right.  For instance, there are ten breweries listed here (as we speak - it has potential to grow!).  Everything was scraping flawlessly by this point, for the most part, but about half way down the list of breweries there was some interesting unicode character that seemed to be coming from nowhere.  I couldn't track it down, so I had to go to rubular.com to sort out a regex that would take care of the formatting....

Also, in some of the brewery names that were scraped, they were returning empty strings that weren't showing up when I was debugging through pry.  So in order to deal with this, I just only created a new brewery if there was valid name information......and *viola*!  

We finally have a project that works, and seems to be pretty air tight and DRY.  For my first attempt at scraping, and creating a CLI I'm pretty pleased and I'm looking forward a great deal to furthering my knowledge on creating gems, scraping, and OOP.  I hope you enjoyed my entry....
