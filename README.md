# release notes / change log / task list

### phase 3 - ui

I took to heart the direction not to read ahead, so only now is the structure of this assignment fully dawning on me.

- Phase 1 Data
- Phase 2 Application
- Phase 3 UI

Which is pretty close to how I like to design greenfield projects, usually data first.

UI will be the weakest link for me. My last UI work was in my previous role, almost six years ago. That was in Angular, 1.3 I think? It was my first and only experience with more modern Javascript frameworks. We used TypeScript with that, which made it easier/more relateable for me. 

So with that, and a bit of Googling to find out what's currently popular in web UIs, I feel like AngularJS might be a good option. Let's do that. This part of the project will contain the most unknowns for me, but I'll do my best to lay out a good starting approach.

I found a number of walkthroughs online for using AngularJS with a Java Spring MVC application. I'd create Angular Views in the project, which would directly call the RESTful webservice developed in the previous phase. 

There are more bullet points in this phase, so I'll just respond to those.

 - ### How business-logic is enforced and how the UI handles errors (this was built in Phase 1)
I'd try to keep as much business logic and error handling as possible in the models, hopefully providing REST responses in a way that's easy/standard for Angular to deal with. Obviously some validations can be performed client-side, or mitigated/enforced through controls like a datetime picker. I vaguely recall some patterns around this in my previous Angular work, I assume there are some standards/best pratices for how to handle and display errors in the various controls. How to automatically display a red info icon next to a text box, for example. 

 - ### How the application could accommodate a different UI on top of the same work built in previous phases
The UI in this case is just calling the REST service, I could easily develop multiple concurrent UIs. 

 - ### How you might test that the UI is functioning correctly. Note, depending on the type of UI, this can be pretty expensive to implement. A simple note in the README is sufficient to describe how you would do this if the scope extends beyond the 1-2 hours required for this exercise
I'm going to assume automated testing here? I know that Selenium is popular with teams that do UI work in my current group, but I've literally never done any kind of automated UI testing. I have zero experience in this area, and would need to research more. 

### The UI needs to support the following actions:

 - ### Buying a stock (user specifies stock symbol and quanitity of shares to purchase)
Obviously allowing the user to specify a stock symbol would need some validation. Initially, with the current design, this would need to be a validation that happens sometime after the REST call to the stock service. If the user input is invalid (stock symbol doesn't exist), let them know after submit. Maybe in a later iteration we could load valid lookup values. Maybe there's an appropriate endpoint already in Alphavantage that we could leverage? Quantity is easier, just needs to be a non-zero, non-negative integer. 
 
 - ### Selling a stock (user specifies stock symbol and quantity of shared to sell)
Not sure there's any difference from the above here, with the intended design. Quantity would still be the non-zero and non-negative, the negative would be applied in application logic at the model level. 
 
 - ### Viewing the net-worth of the entire stock portfolio (i.e. all shares of currentlyowned stocks at their current market value)
 Oh cool, a new endpoint. This is maybe the first bit of real logic so far. I think SUM the quantity field, then grab the latest quote. 
 
 - ### Viewing a "ledger" of transactions, in historical order
 New endpoint, pull all records ordered by date. Maybe an arrow button in the UI to flip the order by. Maybe pagination, server side. 
 
 - ### Viewing the realized net gain/loss of the portfolio (the price at which shares were sold minus the price at which they were purchased)
 Similar to viewing current net-worth, but now actually taking into account the stored price and summing that as well. 
 
 - ### Handling any business-logic errors and integration errors with the quote-service
 These are the ones that developers care about. Log them, provide a way to get them back to developers, hopefully with little or no effort from users. 
 
 - ### Bonus Points: the estimated net gain/loss of the portfolio (the total value of stocks if sold now minus the total price at which active shares were purchased)
Similar logic to view current net-worth (which grabs current quote), but sum the (historical) price column and subtract. 


### phase 2

Okay, right off the bat this is moving in a different direction than I expected, but still seems fairly straight forward. Interestingly the links to Alphavantage.co and its Quote Endpoint don't work for me in the PDF viewer I'm using, but I won't spend any time figuring that out just yet since the context seems clear. 

Initail thoughts based on details pulled from the first paragraph

RESTful service
 - RESTful service: Make a GET request to retrieve Price information
 - Get will subsequently make a REST call out to Quote Endpoint to retrieve a Price for a given StockSymbol (or equivalent) on a given TradeTime
 - Use the value returned to populate the Price field of a new trade record

Securing the API secret
 - For all of Pivotal Cloud Foundry's faults, I quite like the Configuration Server pattern it uses. Encrypted secrets hosted in a Github repo, decrypted and injected into applications at runtime
 - I'm just now getting used to Secret Manager in GCP, which seems functionally somewhat similar but lacks the storage options of PCF's implementation
 - Basically I think this decision will be largely driven by the best practices of the applicaiton hosting environment 
 - I wonder how this is typically done in AWS

Paragraph 2

There seem to be two main concerns here
 - Clean interface
 - Testing

Makes sense as unit testing and interface abstraction go hand-in-hand. I'd make an interface class that basically aligns with what I laid out for the first paragraph and hide the Alphavantage.co specifics in its own implementation. 

Paragraph 3

Service mocking for unit tests. I actually don't think I've ever seen anyone try to write a unit test that instantiated/called-out-to an external service. That would be horrifying. Obviously we don't want to be testing against anyone else's service. Unit tests should be entirely self-contained and testing our own code.

Paragraph 4

Integration tests. Today in my current role we're using Newman, the CLI/test runner for Postman, which works fairly well. We mostly use these to verify our own contracts, but I would be interested to learn about better ways to get ahead of contract changes in external services. 


### data model (day 3)
From the document:
 - Buying a quantity of a specific stock, at a specific time, at a specific price
 - Selling a quanity of a specific stock, at a specific time, at a specific price
 - Summarizing the total purchase price of all stocks in the portfolio
 - Summarizing the total purchase price of a single stock
 - Summarizing net gain/loss for all stocks in the portfolio
 - Summarizing net gain/loss for a single stock

At this point I'm thinking maybe a single table, debits and credits, accounting style. I did something similar once when refactoring an inventory system, I liked how that turned out. I would make a Create Table script to define it in a relational database, heavy leveraging of aggregate functions (SUM). Buying is a positive integer in the quantity column, Selling is negative, Summing and Grouping can handle the rest. Quick and dirty. Simple.

Maybe something like... 

CREATE TABLE Trades (
    StockSymbol varchar(5),
    Quantity int,
    Price money,
    TradeTime datetime
);

StockSymbol would be the primary key with a unique constraint. Can always add some other key later if needed. I know the instructions at this point call for a single user, but I feel like multiple users could be brought into things later, which would obviously change the key. Maybe a composit key then. 

## ^^^ begin conceptual design - no implementation ^^^

### data setup (day 2)
- added h2 dependencies and configuration to Java SPA
- installed postgresql for learning/rapid prototyping of data design (never used this, trying to avoid SSMS/SQL Server)
- plan is to create sql scripts for initialization of file-persisted h2

### project setup
- blank spring initilizr project
- new github repo
- authenticate github to intellij
- first commit
- add instructions to readme

### project approach
- Figured I'd stick with Java as that's mostly what I've been exposed to in recent years. At the very least this approach will be valuable experience for getting back to daily coding in my current situation. I'll start with a blank Spring Initilizr project and add dependancies as needed, starting with a Spring Data or Spring Rest to facilitate what I imagine will be some standard CRUD operation RESTful APIs around the h2 relational database I plan to build. I briefly considered something more SPA/framework-y like JHipster, but decided against that as the last Boostrap kind of thing I did was in .NET, I'm not as familiar with Java options, and not reading ahead I didn't want to box myself in too much too early on. 

### dev environment setup (from zero)
- install homebrew
- install openjdk
- install install maven
- install intellij ce
- install git

# summit-stock-portfolio
Stock Portfolio Coding Exercise

Stock Portfolio Coding Exercise 
Wednesday, September 11, 2019 10:45 AM
Overview 
This is a take-home coding exercise designed to see how you design, code and  communicate. There are no obtuse problems to solve here or exotic algorithms you  need to invent. This is meant to be a relatively simple exercise, so it you shouldn't  spend more than 1-2 hours on this.  
This exercise will reveal requirements to youi in phases. So don't skip ahead and read  the next phase until you have completed the previous one. If at all possible, use an  SCM tool (preferrably git) to stage and commit your work in phases. It helps us see  how you iterate on a project. 
The directory structure of the project is up to you, but at a minimum we would like to  see: 
• The application code, broken-down into as many sub-directories as makes sense  for the language and frameworks used (if any) 
• Unit-test code 
• Third-party libraries. For Ruby, a bundler Gemfile, for Python a requirements.txt  or Pipfile, for Java a maven.xml, etc. 
• A top-level README providing a general outline of what is in the project, with  some top-level description of its design and any additional instructions to get  the code to run.  
If you find you are going over time, get to a stopping point, commit the work you have  and add a note to the README describing where you left off and the challenges you  ran into during implementation. Not completing the project isn't a bad thing,  especially if you can articulate why. 
One final note is to make sure that the application and any automated tests will  actually execute (after setting up whatever third-party dependencies are necessary).  We expect to be able to unzip the file you send us and run the code! 
Problem Statement 

One final note is to make sure that the application and any automated tests will  actually execute (after setting up whatever third-party dependencies are necessary).  We expect to be able to unzip the file you send us and run the code! 
Problem Statement 
In this exercise, you will be iteratively building a stock-portfolio application for a single  user. Think of it as a sort of Quicken-lite application. 
You can build this in any language and framework you prefer. Choose the one you are  the most familiar to minimize time spent on simple syntax problems or figuring out  standard libraries. Put your best foot forward! 
Note that there will be some kind of user-interface to build as part of this project, but  what that UI looks like is entirely up to you. Anything from native GUIs, to web applications to a simple command-line interface are all acceptable.  
Phase 1: Initial Design 
Design a data-model for a stock portfolio and implement it using the persistence  mechanism of your choice (RDBMS, SQLite, MongoDB, flat-files, etc.) Be sure to  provide adequate commentary explaining your choice of the persistence mechanism  (putting this in a top-level README file would be a good move). 
If the data-model requires a schema (i.e. when using a RDBMS) provide a mechanism  to create the tables necessary for that mechanism. Additionally, provide a library of  code that provides access to read and write to the data-model. This will likely be  heavily influenced by your choice of language and frameworks. 
The data-model needs to be able to handle these kinds of transactions: 
• Buying a quantity of a specific stock, at a specific time, at a specific price • Selling a quanity of a specific stock, at a specific time, at a specific price • Summarizing the total purchase price of all stocks in the portfolio • Summarizing the total purchase price of a single stock 
• Summarizing net gain/loss for all stocks in the portfolio 
• Summarizing net gain/loss for a single stock 
This is the foundational layer of your application and the commit(s) for this phase only  need to focus on expressing the underlying data-model and how it's expressed in a  model-layer of objects & classes or functions. For now, don't worry about how we  determine the price of a stock and buy/sell time, we'll get to that shortly.

This is the foundational layer of your application and the commit(s) for this phase only  need to focus on expressing the underlying data-model and how it's expressed in a  model-layer of objects & classes or functions. For now, don't worry about how we  determine the price of a stock and buy/sell time, we'll get to that shortly. 
Think about what sorts of unit-testing is appropriate at this level. Think about what  sorts of logical rules need to be enforced (e.g. you can't sell a stock that you don't  own) and how the system expresses and enforces those rules. Also think about how to  identify stocks, purchases and shares as part of your design. 
Phase 2: Quote Integration 
In this next phase the application needs to integrate with an external service to get a  price for a stock on an exchange at a specific moment in time. For this exercise, you  will integrate with Alphavantage.co, a simple REST service for retrieving real-time  pricing. You can sign up for a free API key and integrate with the Quote Endpoint. As  you build this integration, think about how the API key should be managed in terms of  the source-code vs. the deployed and running application. How would you manage  the key in the most secure manner possible? 
Write the integration with this service, but design the integration in such a way that  the underlying implementation could be switched to another provider relatively easily  without rippling extensive changes to other parts of the code-base. Also think about  how to handle the inevitable failures from this system and how much of that needs to  change along with a change in the underlying implementation. 
Since this an integration with an external service, think about how to unit-test this  integration withour relying on it during the unit-test runs. Imagine this is running in a  CI tool. We don't want to tests to fail because a third-party service is unavailable. 
What sorts of integration testing would you do to check the health of the integration?  This could be provided as part of a separate test, a list of manual instructions in the  README, or anything else you can imagine. 
Phase 3: User Interface 
Now that we have a model and basic integration in place, it's time to build a user interface on top of this. Again, the choice of UI is entirely up to you. We are not looking for a beautiful UI but we want to see thoughfulness in terms of its structure  and presentation. 
So over-arching thing to consider when building this interface:

    . ,         .    looking for a beautiful UI but we want to see thoughfulness in terms of its structure  and presentation. 
So over-arching thing to consider when building this interface: 
• How business-logic is enforced and how the UI handles errors (this was built in  Phase 1). 
• How the application could accommodate a different UI on top of the same work  built in previous phases 
• How you might test that the UI is functioning correctly. Note, depending on the  type of UI, this can be pretty expensive to implement. A simple note in the  README is sufficient to describe how you would do this if the scope extends  beyond the 1-2 hours required for this exercise 
The UI needs to support the following actions: 
• Buying a stock (user specifies stock symbol and quanitity of shares to purchase) • Selling a stock (user specifies stock symbol and quantity of shared to sell) • Viewing the net-worth of the entire stock portfolio (i.e. all shares of currently owned stocks at their current market value) 
• Viewing a "ledger" of transactions, in historical order 
• Viewing the realized net gain/loss of the portfolio (the price at which shares  were sold minus the price at which they were purchased) 
• Handling any business-logic errors and integration errors with the quote-service • Bonus Points: the estimated net gain/loss of the portfolio (the total value of  stocks if sold now minus the total price at which active shares were purchased)
