## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/xKONIGSTIGERx/SNHUPortfolioClaudHarmon/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/xKONIGSTIGERx/SNHUPortfolioClaudHarmon/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Code Review

https://youtu.be/9byuzVa9r_g

### Software Design and Engineering

### Algorithms and Data Structure

### Databases

'''
#!/usr/bin/python
import json
from bson import json_util
from bson.json_util import dumps
import bottle
from bottle import route, run, request, abort
#imports for database
from pymongo import MongoClient
connection = MongoClient('localhost', 27017)
db = connection['market']
collection = db['stocks']
collection_comp = db['company']
line = "--" * 50+"\n"


class Database:
  def __init__(self,name = "Custom Data Functions"):
    self.name = name
    
  #this method is used to insert new documents in the stock collection
  #used for creation of new stocks
  def insert_new_document(self,document):
    record_created = collection.insert(document) 
    added_doc = collection.find_one({"_id":record_created})
    stars = "--" * 50+"\n"
    return stars+ "Created Document \n "+dumps(added_doc)+"\n\n "+stars #return inserted document.
  
  def update_document(self,data,ticker):
    #loop and update data
    print("Below Documents Will BE Updated...")
    line = "--" * 50+"\n"
    
    #print items before update
    print(line)
    query = { "Ticker" :ticker}
    result= collection.find(query)
    result = dumps(result)
    print(result)
    
    print(line)
    
    #find items and update them using a loop
    for item in data:
      print(data[item])
      new_update =  { "$set":{item:data[item]}}
      #create query
      query = { "Ticker" :ticker}
      collection.update(query,new_update)
      
    updated_doc = collection.find({"Ticker":ticker}) 
    line = "--" * 50+"\n"
    result = dumps(updated_doc) #
    #return updated documents
    return line+"\n"+str(result)+"\n"+line
  
  def getReport(self,ticker):
    #stage one 
    stage_1 = { '$project': { 'Ticker':1,'Average True Range':1,'EPS growth this year':1,'Shares Outstanding':1,'Volume':1 } }
    #Create query for stage two
    stage_2 = { '$match': { "Ticker": ticker } }
    #stage three which will use summation, average, outstanding shares and others
    stage_3 = { '$group': { '_id': "$Ticker", 'Total Shares Outstanding': {'$sum': "$Shares Outstanding" },
                             'Average True Range':{'$avg':"$Average True Range"},
                             'Average EPS growth this year':{'$avg':'$EPS growth this year'},
                             'Max Shares Outstanding':{'$max':'$Shares Outstanding'},
                             'Total Volume':{'$sum':'$Volume'} } }


    #create query
    query = [stage_1,stage_2,stage_3] 
    #run all the queries
    result=collection.aggregate(query) 
    #create a result that can be printed
    result = dumps(result)
    return result
  
  def getPortfolio(self,industry):
      
      #stage one 
      stage_1 = { '$project': { 'Industry':1,'Ticker':1,'EPS growth this year':1,'Shares Outstanding':1,'Volume':1 } }
      #Create query for stage two
      stage_2 = { '$match': { "Industry": industry } }
      #stage three which will use summation, average, outstanding shares and others
      stage_3 = { '$group': { '_id': "$Industry", 'Total Shares Outstanding': {'$sum': "$Shares Outstanding" },
                               'Average True Range':{'$avg':"$Average True Range"},
                               'Average EPS growth this year':{'$avg':'$EPS growth this year'},
                               'Max Shares Outstanding':{'$max':'$Shares Outstanding'},
                               'Total Volume':{'$sum':'$Volume'} } }

      stage_4 = { '$limit' : 5 }
      #create query
      query = [stage_1,stage_2,stage_3,stage_4] 
      #run all the queries
      result=collection.aggregate(query) 
      #create a result that can be printed
      result = dumps(result)
      return result
    
    
# set up URI paths for REST service
@route('/createStock/', method='POST')
def createStock():
  data = request.json
  print("Server Have Received ......")
  print(data)
  db = Database()
  print("Sending Data to Database")
  result = db.insert_new_document(data)
  print(result)
  
  
#-------------------------------------------------------#
#                     update stocks documenst           #
#           this method will use the PUT Method         #
#-------------------------------------------------------#

@route('/update_document/<ticker>', method='PUT')
def createStock(ticker):
  data = request.json #retrive data from url
  #display data, in server
  print(data) 
  db = Database()
  result = db.update_document(data,ticker)
  print("Below Documents Have Been Updated....")
  print(result)
  
#-------------------------------------------------------#
#                     Delete Stocks                     #
# this method will use the GET Method  To delete items  #
#-------------------------------------------------------# 

@route('/delete_document/<Ticker>', method='GET')
def delete_doc(Ticker):
  #CREATE QUERY
  query = {"Ticker" :Ticker}
  result= collection.find(query)
  result = dumps(result)
  #Print data to be deleted
  print(line)
  print("##          Documents Below Will Be Deleted       ##")
  print(result) 
  #delete documents using the created query
  result = collection.delete_many(query)
  #create message
  print(line)
  print("##          Documents Have Been Deleted       ##")
  print(line)
  
#-------------------------------------------------------#
#                     stock Stocks                      #
# this method will use Post Method to create report     #
#-------------------------------------------------------#  
@route('/stockReport/', method='POST')
def run_create(): 
  data = request.json #retrive data from url
  print(data)
  db = Database()
  items = []
  #This for loop uses each ticker in the list,
  #get ist summer and add the summery to the items list
  for item in data:
      list_of_items = data[item]
      array_of_items = list_of_items.split(",")
      #get a report for each item
      for x in array_of_items:
        print(x)
        retrived_item = db.getReport(x)
        if len(retrived_item) == 0:
          retrived_item = "...Ticker Symbol Not Found...."
        items.append("#### Report For Ticker "+x+" ##### \n"+retrived_item+"\n######\n")
  
  return items #return an array of report

#-----------------------------------------------------------#
#                     Industry Report                      -#
#  Obtain a json data which is used to create the report   -#
#-----------------------------------------------------------#

@route('/industryReport/', method='POST')
def run_create():
  data = request.json #retrive data from url
  print(data)
  db = Database()
  for item in data:
      industry = data[item]
      
  print(line)
  print("#              Top Five                #")
  print(line)
  
  print(industry)
  result = db.getPortfolio(industry)
  #print results to user.
  return "-------- \n Portfolio Report For The First Five "+industry+" Industries \n\n "+result+" \n-------- \n\n"

@route('/portfolio/<company>', method='GET')
def run_create(company):
  company = company.replace("_"," ") 
  print(company)
  query = {"Company":company} #formulates query
  result=collection.find(query) #find documents using the query
  result = dumps(result)
  return "-------- \n Portfolio Report For "+company+" Company \n\n "+result+" \n-------- \n\n"
   
  
if __name__ == '__main__':
  #app.run(debug=True)
  run(host='localhost', port=8080)
'''

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
