# Getting started with Selenium Hub and Watir in Rails

### Gemfile

    group :test do
      gem 'watir-rails'
      gem 'watir-rspec'
    end

### spec_helper.rb

    # Configuration for selenium-webdriver
    require 'selenium-webdriver'

    # Configuration for watir-rspec
    require "watir/rspec"

    RSpec.configure do |config|
      # Use Watir::RSpec::HtmlFormatter to get links to the screenshots, html and
      # all other files created during the failing examples.
      config.add_formatter(:progress) if config.formatters.empty?
      config.add_formatter(Watir::RSpec::HtmlFormatter)

      ARGS = ['--ignore-certificate-errors', '--disable-popup-blocking', '--disable-translate'].freeze

      # Open up the browser for each example.
      config.before :all, type: :request do
        @browser = Watir::Browser.new :chrome, url: "http://hub:4444/wd/hub", options: { args: ARGS }
      end

      # Close that browser after each example.
      config.after :all, type: :request do
        @browser.close if @browser
      end

      # Include RSpec::Helper into each of your example group for making it possible to
      # write in your examples instead of:
      #   @browser.goto "localhost"
      #   @browser.text_field(name: "first_name").set "Bob"
      #
      # like this:
      #   goto "localhost"
      #   text_field(name: "first_name").set "Bob"
      #
      # This assumes that you've used @browser as an instance variable name in
      # before :all block.
      config.include Watir::RSpec::Helper, type: :request

      # Include RSpec::Matchers into each of your example group for making it possible to
      # use #within with some of RSpec matchers for easier asynchronous testing:
      #   expect(@browser.text_field(name: "first_name")).to exist.within(2)
      #   expect(@browser.text_field(name: "first_name")).to be_present.within(2)
      #   expect(@browser.text_field(name: "first_name")).to be_visible.within(2)
      #
      # You can also use #during to test if something stays the same during the specified period:
      #   expect(@browser.text_field(name: "first_name")).to exist.during(2)
      config.include Watir::RSpec::Matchers, type: :request
    end

### docker-compose

    # To execute this docker-compose yml file use docker-compose -f <file_name> up
    # Add the "-d" flag at the end for deattached execution
    version: '2'
    services:
      firefox:
        image: selenium/node-firefox-debug:3.9.1
        volumes:
          - /dev/shm:/dev/shm
        depends_on:
          - hub
        environment:
          HUB_HOST: hub

      chrome:
        image: selenium/node-chrome-debug:3.9.1
        volumes:
          - /dev/shm:/dev/shm
        depends_on:
          - hub
        environment:
          HUB_HOST: hub

      hub:
        image: selenium/hub:3.9.1
        ports:
          - "4444:4444"

### Scaling the grid on-demand


##### Scaling up Chrome nodes

    $ docker-compose scale chrome=5
    # Spawns four additional node-chrome instances linked to the hub

##### Scaling up Firefox nodes

    $ docker-compose scale firefox=5
    # Spawns four additional node-firefox instances linked to the hub

### Usage

      require "spec_helper"

      describe "Google" do
        before { goto "http://google.com" }

        it "has search box" do
          expect(text_field(name: "q")).to be_present
        end

        it "allows to search" do
          text_field(name: "q").set "watir"
          button(id: "gbqfb").click
          results = div(id: "ires")
          expect(results).to be_present.within(2)
          expect(results.lis(class: "g").map(&:text)).to be_any { |text| text =~ /watir/ }
          expect(results).to be_present.during(1)
        end
      end
