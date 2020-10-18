### Personal Self-Assessment

When I first started at Southern New Hampshire University I had experiance with programming either with games or even with robotics but what I achieved here allowed me to hone my skills and get some well needed project experiance. Before the program I was mainly just tinkering and playing around but even just these 3 projects I have shown have given me a ton of experiance that I will need going towards the future. Overall it has been a great experiance here at SNHU and even with the capstone to be able to look back and see all the progress I have made. It makes me excited to move into the future to whatever career I move into and hopefully gain some more needed experiance.

For collaborating in a team enviroment I was able to get some key experiance when working either with members of my team on cyberpatriot, classwork, or even other projects online with people. Working together and communication is something that is very important when working in a company and I feel I have a ton of experiance in this area.
For security everyone needs to understand how important it is especially when programming because you not only have your company to keep in good standing but also your consumers as well. You have to make sure your program is bug free by either extensive bug testing or doing code reviews with peers to make sure something catastrophic doesn't happen to possibly give someone access to whatever you have made. Thankfully I have had a ton of experiace with this either creating an editing projects to make sure they were up to standards or even participating in hackathons and cyberpatriot to get some first hand experiance in what these hackers are looking for. Another major thing about security is keeping up with it and making sure to go back and fix your own programs down the road if a new exploit is discovered.

### Code Review

https://youtu.be/9byuzVa9r_g

The video link above goes to a code review of the following projects of this portfolio. Code review is extremely important not only for you but your peers as well because we help and critque each other so that we all progress. This is something I have gotten some decent experiance with and am excited to get started in a more professional capacity.

### Software Design and Engineering

```
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
```

This artifact is from my CS-340 class earlier this yea (2020) and is a restful api to work with a database. This is an important piece of code to show my progress with
working with different APIs to get the job done. This item was selected because it shows my ability to code from scratch and then apply that to another concept such as
databases and MySQL or mongo. This also shows my ability to work with different programming languages like python when my main language is C++ or Java. Being able to be flexible with programming languages is very important because some languages can achieve things faster and more efficient than others. 

### Algorithms and Data Structure

```
from graphics import *
import sys
import tkMessageBox
#+-added to be able to select a random number from a range of numbers
from random import randrange

'''
The revisions in the code are labelled by the prefix '#+-'
'''
class Checkers:
    def __init__(s):
        s.state = 'CustomSetup'
        s.is1P = False
        #+-added string
        s.compIsColour = 'not playing'    #computer not playing by default
        s.placeColour = 'White'
        s.placeRank = 'Pawn'
        s.placeType = 'Place'                   #Place or Delete (piece)
        s.pTurn = 'White'                       #White or Black (turn)
        
        s.selectedTile = ''
        s.selectedTileAt = []
        s.hasMoved = False
        s.pieceCaptured = False

        s.BoardDimension = 8
        s.numPiecesAllowed = 12

        s.win = GraphWin('Checkers',600,600)    #draws screen
        s.win.setBackground('White')
        s.win.setCoords(-1,-3,11,9)              #creates a coordinate system for the window
        s.ClearBoard()
        
        s.tiles = [[Tile(s.win,i,j,False) for i in range(s.BoardDimension)] for j in range(s.BoardDimension)]  #creates the 2D list and initializes all 8x8 entries to an empty tile

        #+-added two lists
        s.moves = []
        s.badMoves = []

        gridLetters = ['A','B','C','D','E','F','G','H',]
        for i in range(s.BoardDimension):
            Text(Point(-0.5,i+0.5),i+1).draw(s.win)                 #left and right numbers for grid
            Text(Point(8.5,i+0.5),i+1).draw(s.win)
            Text(Point(i+0.5,-0.5),gridLetters[i]).draw(s.win)      #bottom and top letters
            Text(Point(i+0.5,8.5),gridLetters[i]).draw(s.win) 
        
        s.SetButtons()
        
        s.SetupBoard()
	####
    #handles the setup of the board (i.e. piece placement)
	####
    def SetupBoard(s):
        while s.state == 'CustomSetup':
            s.Click()
        if s.state == 'Play':
            s.Play()
	####
    #handles the general play of the game
	####
    def Play(s):
        while s.state == 'Play':
            #+-added if statement
            if s.is1P and s.compIsColour == s.pTurn:
                s.CompTurn()
            else:
                s.Click()
        if s.state == 'CustomSetup':
            s.SetupBoard()
	####
    #+-added to be able to control the computer's turn
	####
    def CompTurn(s):
        s.moves = s.movesAvailable()
        s.badMoves = []
        s.goodMoves = []
        #######consider making s.goodMoves which replaces s.badMoves for many uses (changes conditions a bit as well)

        
        #To prevent leaving the back row (currently top priority)
        for move in s.moves:
            if s.movesFromBack(move):
                s.badMoves.append(move)
        s.removeBadMoves()

        #This may not always work, it is not expected/designed to (hopefully it usually works)
        #It is meant to promote the use of moves that allow capture of another piece afterwards
        for move in s.moves:
            if not s.PieceCanCapture(s.moveEndsAt(move)[0],s.moveEndsAt(move)[1]):
                s.badMoves.append(move)
        s.removeBadMoves()

        #Promotes move which make kings
        for move in s.moves:
            if not (((s.moveEndsAt(move)[1]==7 and s.tiles[move[0]][move[1]].isWhite) or \
               (s.moveEndsAt(move)[1]==0 and s.tiles[move[0]][move[1]].isBlack)) and \
                    s.tiles[move[0]][move[1]].isPawn):
                s.badMoves.append(move)
        s.removeBadMoves()


        #Promotes moves which take kings for free
        for move in s.moves:
            if not ((s.tiles[move[2]][move[3]].isKing) and s.isMoveSafe(move)):
                s.badMoves.append(move)
        s.removeBadMoves()

        #This may not always work, it is not expected/designed to (hopefully it usually works)
        #Promotes moves which trade your pawn for opponent king
        for move in s.moves:
            if not ((s.tiles[move[0]][move[1]].isPawn) and (s.tiles[move[2]][move[3]].isKing) and not s.isMoveSafe(move)):
                s.badMoves.append(move)
        s.removeBadMoves()

        #This may not always work, it is not expected/designed to (hopefully it usually works)
        #Promotes moves which trade kings when ahead in ***pieces <--might want to change classification of being ahead
        for move in s.moves:
            if not ((s.tiles[move[0]][move[1]].isKing) and (s.tiles[move[2]][move[3]].isKing) and not s.isMoveSafe(move) and s.hasMorePieces()):
                s.badMoves.append(move)
        s.removeBadMoves()

        #This may not always work, it is not expected/designed to (hopefully it usually works)
        #Promotes moves which trade pawns when ahead in ***pieces <--might want to change classification of being ahead
        for move in s.moves:
            if not ((s.tiles[move[0]][move[1]].isPawn) and (s.tiles[move[2]][move[3]].isPawn) and not s.isMoveSafe(move) and s.hasMorePieces()):
                s.badMoves.append(move)
        s.removeBadMoves()

        #Promotes moves which does not endanger itself needlessly
        for move in s.moves:
            if not s.isMoveSafe(move):
                s.badMoves.append(move)
        s.removeBadMoves()
        
        #should implement the next part to pick at random from the remaining moves (does not do this)
        #performs the select and move action for the computer
        m = randrange(0,len(s.moves))
        if s.selectedTileAt == []:
            s.Action(s.moves[m][0],s.moves[m][1])
        s.Action(s.moves[m][2],s.moves[m][3])

    ####
	#+-added to determine whether the player whose turn it is has more pieces than opponent
	####
    def hasMorePieces(s):
        return s.numColour(s.pTurn) > s.numColour(s.opposite(s.pTurn))
    
    #+-added new method
    #Returns true if a GIVEN piece cannot be taken immediately after ITS proposed move
    #########Might want another method to know if a move exposes another piece
    #Likely to not work (depends on how PieceCanCapturePiece method works)
    def isMoveSafe(s,move):
        X1,Y1 = [s.moveEndsAt(move)[0]-1,s.moveEndsAt(move)[0]+1],[s.moveEndsAt(move)[1]-1,s.moveEndsAt(move)[1]+1]
        for i in range(2):
            for j in range(2):
                if s.SpecialPCCP(s.tiles[move[0]][move[1]].pieceColour,X1[i],Y1[j],s.moveEndsAt(move)[0],s.moveEndsAt(move)[1],move[0],move[1]):
                    return False
        return True

    #+-added new method
    #modification to PieceCanCapturePiece such that it works with the isMoveSafe method
    def SpecialPCCP(s,piece2Colour,x,y,X,Y,initX,initY):

        X1,X2,Y1,Y2 = [x-1,x+1],[x-2,x+2],[y-1,y+1],[y-2,y+2]
        if ((0<=X<8) and (0<=Y<8)) and ((0<=x<8) and (0<=y<8)):
            if (piece2Colour == s.opposite(s.tiles[x][y].pieceColour)):
                if s.CanDoWalk(x,y,X,Y,exception=False):
                    for i in range(2):
                        for j in range(2):
                            if X1[i]==X and Y1[j]==Y:
                                if (0<=X2[i]<8) and (0<=Y2[j]<8):
                                    if not (s.tiles[X2[i]][Y2[j]].isPiece) or (X2[i]==initX and Y2[j]==initY):
                                        return True
        return False

    #+-added new method
    #Removes badMoves from the list of moves that can be made
    #if all of the moves that can be made are badMoves, we still need to do something so it leaves all of them
    #Note: at any time this is run, the moves that are considered bad should be considered bad for the same reason (i.e. they are equivalently bad)
    def removeBadMoves(s):  #assumes moves were added to badMoves in the same order as they appear in moves
        if s.moves != s.badMoves:
            for move in s.badMoves:
                s.moves.remove(move)
        s.badMoves = []

    #+-added new method
    def movesFromBack(s,move):
        if (move[1]==0 and s.compIsColour=='White') or \
               (move[1]==7 and s.compIsColour=='Black'):
            return True
        else:
            return False

    #+-added new method
    def moveEndsAt(s,move): #takes 4 element array representing a move
        if s.tiles[move[2]][move[3]].isPiece:
            return [move[0]+(move[2]-move[0])*2,move[1]+(move[3]-move[1])*2]
        else:
            return [move[2],move[3]]
    
    #+-added to calculate all the available valid moves
    def movesAvailable(s):
        moves=[]
        for j in range(8):
            for i in range(8):
                X1,Y1 = [i-1,i+1],[j-1,j+1]
                for a in range(2):
                    for b in range(2):
                        if 0<=X1[a]<8 and 0<=Y1[b]<8:
                            if s.moveIsValid(i,j,X1[a],Y1[b]):
                                moves.append([i,j,X1[a],Y1[b]])
        return moves
                

	#Resets the board to be empty
    def ClearBoard(s):
        s.tiles=[[Tile(s.win,i,j,False) for i in range(s.BoardDimension)] for j in range(s.BoardDimension)]  #creates the 2D list and initializes all 8x8 entries to an empty tile
        for i in range(s.BoardDimension):
            for j in range(s.BoardDimension):
                s.ColourButton(s.TileColour(i,j),i,j)
        s.state = 'CustomSetup'
        s.pTurn = 'White'
        s.SetButtons()

    def ColourButton(s,colour,X,Y,width=1,height=1):        #function to create a rectangle with a given colour, size, and location
        rect = Rectangle(Point(X,Y),Point(X+width,Y+height))
        rect.setFill(colour)
        rect.draw(s.win)
            
    def TileColour(s,x,y):
        if (x%2 == 0 and y%2 == 0) or (x%2 == 1 and y%2 == 1):
            return 'Red'   #sets every other square to red 
        else:
            return 'White' #every non red square to white

    ##########
    #Draws Buttons
    ##########    
    def SetButtons(s):
        s.ColourButton('White',-1,-3,12,2)
        s.ColourButton('White',9,-1,2,10)

        if s.state == 'CustomSetup':
            s.DrawStandard()
            s.DrawStart()
            s.DrawClear()
            s.Draw1P()
            s.Draw2P()
            s.DrawLoad()
            s.DrawSave()
            s.DrawTurn()
            s.DrawX()
            
            s.DrawW()
            s.DrawB()
            s.DrawK()
            s.DrawDel()

            s.DrawScore() #not actually a button
        elif s.state == 'Play':
            #+-updated method name
            s.DrawResign()
            s.DrawSave()
            s.DrawTurn()
            s.DrawX()

            s.DrawScore() #not actually a button


    def DrawStandard(s):
        s.ColourButton('White',-1,-2,2,1)    #Standard Setup button
        Text(Point(0,-1.3),'Standard').draw(s.win)
        Text(Point(0,-1.7),'Setup').draw(s.win)
    def DrawCustom(s):
        s.ColourButton('White',-1,-3,2,1)    #Custom Setup button
        Text(Point(0,-2.3),'Custom').draw(s.win)  
        Text(Point(0,-2.7),'Setup').draw(s.win)
    def DrawStart(s):
        s.ColourButton('Yellow',1,-2)    #Start! button
        Text(Point(1.5,-1.5),'Start!').draw(s.win)
    def DrawClear(s):
        s.ColourButton('White',-1,-3,2,1)    #Clear Board button
        Text(Point(0,-2.3),'Clear').draw(s.win)
        Text(Point(0,-2.7),'Board').draw(s.win)
    def Draw1P(s):
        col = 'Red'
        if s.is1P:
            s.DrawCompColour()
        else:
            s.ColourButton(col,3,-2,2,1)    #1Player   -- (1AI)
            Text(Point(4,-1.3),'1Player').draw(s.win)
            Text(Point(4,-1.7),'Game').draw(s.win)
    def DrawCompColour(s):
        s.ColourButton(s.compIsColour,3,-2,2,1)
        txt1 = Text(Point(4,-1.3),'Comp Is')
        txt2 = Text(Point(4,-1.7),s.compIsColour)
        txt1.draw(s.win)
        txt2.draw(s.win)
        txt1.setFill(s.opposite(s.compIsColour))
        txt2.setFill(s.opposite(s.compIsColour))
    def Draw2P(s):#2Player
        col = 'Green'
        if s.is1P:
            col = 'Red'
        s.ColourButton(col,3,-3,2,1)    
        Text(Point(4,-2.3),'2Player').draw(s.win)  
        Text(Point(4,-2.7),'Game').draw(s.win)
    def DrawLoad(s):#Load
        s.ColourButton('White',6,-3,2,1)    
        Text(Point(7,-2.5),'Load').draw(s.win)
    def DrawSave(s):    #Save
        s.ColourButton('White',8,-3,2,1)
        Text(Point(9,-2.5),'Save').draw(s.win)
    def DrawX(s):#X
        s.ColourButton('Red',10,-3)    
        Exit_txt = Text(Point(10.5,-2.5),'X')
        Exit_txt.draw(s.win)
        Exit_txt.setFill('White')
    def DrawW(s):#W
        col = 'Green'
        if s.placeColour != 'White':
            col = 'Red'
        s.ColourButton(col,6,-2)    
        Text(Point(6.5,-1.5),'W').draw(s.win)
    def DrawB(s):#B
        col = 'Red'
        if s.placeColour != 'White':
            col = 'Green'
        s.ColourButton(col,7,-2)    
        Text(Point(7.5,-1.5),'B').draw(s.win)
    def DrawK(s): #K
        col = 'Red'
        if s.placeRank == 'King':
            col = 'Green'
        s.ColourButton(col,8,-2) 
        Text(Point(8.5,-1.5),'K').draw(s.win)
    def DrawDel(s):#Del
        col1 = 'Black'#square colour
        col2 = 'White'#text colour
        if s.placeType == 'Delete':
            col1 = 'Green'
            col2 = 'Black'        
        s.ColourButton(col1,9,-2)    
        deleteTxt = Text(Point(9.5,-1.5),'Del')
        deleteTxt.draw(s.win)
        deleteTxt.setFill(col2)
    def DrawResign(s):
        s.ColourButton('White',6,-3,2,1)    #Load
        Text(Point(7,-2.5),'Resign').draw(s.win)
    def DrawTurn(s):
        col1 = 'White'
        col2 = 'Black'
        if s.pTurn == 'Black':
            col1 = 'Black'
            col2 = 'White'
        s.ColourButton(col1,9,8,2,1)    #Standard Setup button
        txt1 = Text(Point(10,8.7),col1)
        txt2 = Text(Point(10,8.3),'Turn')
        txt1.draw(s.win)
        txt2.draw(s.win)
        txt1.setFill(col2)
        txt2.setFill(col2)
    def DrawScore(s): # draw score
        Text(Point(10,7.5),'# White').draw(s.win)
        Text(Point(10,7.1),'Pieces:').draw(s.win)
        Text(Point(10,6.7),s.numColour('White')).draw(s.win)
        Text(Point(10,5.9),'# Black').draw(s.win)
        Text(Point(10,5.5),'Pieces').draw(s.win)
        Text(Point(10,5.1),s.numColour('Black')).draw(s.win)
                 
    def Click(s):
        click = s.win.getMouse()        #Perform mouse click
        X, Y = s.ClickedSquare(click)   #Gets click coords
        s.Action(X,Y)
        
    def Action(s,X,Y):      #performs action for the location X,Y --essentially means user clicked there or computer is 'clicked' there
        if s.state == 'CustomSetup':
            s.clickInCustom(X,Y)
        elif s.state == 'Play':
            s.clickInPlay(X,Y)
            
    def clickInCustom(s,X,Y):
        #+-added X button if statement
        if (10<=X<11 and -3<=Y<-2): #X clicked
            ExitGame(s.win)
        elif (-1<=X<1 and -2<=Y<-1): #Standard clicked
            s.StandardSetup()  
        elif (1<=X<2 and -2<=Y<-1): #Start! clicked
            num_wh = s.numColour('White')
            num_bl = s.numColour('Black')
            if ((num_wh == 0) and (num_bl == 0)): #This means there are no pieces on the board; this is a pointless setup.
                tkMessageBox.showinfo("Error", "No pieces have been placed!")
            else:
                s.state = 'Play'
                s.SetButtons()
        elif (-1<=X<1 and -3<=Y<-2): #Clear Board clicked
            s.ClearBoard()
        #+-added 1Player and Comp Is if statements
        elif (3<=X<5 and -2<=Y<-1 and not s.is1P): #1Player clicked
            s.is1P = True
            s.compIsColour = 'White'
            s.SetButtons()
        elif (3<=X<5 and -2<=Y<-1 and s.is1P): #'Comp Is' button clicked --requires the user to have already designated it to be a 1 player game, else the option is not available
            s.compIsColour = s.opposite(s.compIsColour)
            s.SetButtons()
        elif (3<=X<5 and -3<=Y<-2): #2Player clicked
            s.is1P = False
            s.SetButtons()
        elif (8<=X<10 and -3<=Y<-2): #Save clicked
            s.SaveSetupToFile()
        elif (6<=X<8 and -3<=Y<-2): #Load clicked
            s.LoadSetupFromFile()
        elif (9<=X<11 and 8<=Y<9): #pTurn button clicked during CustomSetup
            s.pTurn = s.opposite(s.pTurn)
            s.SetButtons()
        elif (6<=X<7 and -2<=Y<-1): #W clicked
            s.placeColour = 'White'
            s.placeType = 'Place'
            s.SetButtons()
        elif (7<=X<8 and -2<=Y<-1): #B clicked
            s.placeColour = 'Black'
            s.placeType = 'Place'
            s.SetButtons()
        elif (8<=X<9 and -2<=Y<-1): #K clicked
            s.placeRank = s.opposite(s.placeRank)
            s.placeType = 'Place'
            s.SetButtons()
        elif (9<=X<10 and -2<=Y<-1): #Del clicked
            s.placeType = s.opposite(s.placeType)
            s.SetButtons()
        elif (0<=X<8 and 0<=Y<8): #Tile clicked in CustomSetup
            if s.tiles[X][Y].TileColour(X,Y) == 'White': #Clicked tile is White
                tkMessageBox.showinfo("Error", "Illegal Placement")
            elif s.numColour(s.placeColour) >= s.numPiecesAllowed and s.placeType == 'Place': #clicked tile would result in too many of colour being placed
                tkMessageBox.showinfo("Error", "Illegal Placement")
            elif (Y == 7 and s.placeColour == 'White' and not(s.placeRank == 'King')) or (Y == 0 and s.placeColour == 'Black' and not(s.placeRank == 'King')): #placing a non-king on a king square
                tkMessageBox.showinfo("Error", "Illegal Placement")
            else: #Valid tile update action (i.e. piece placement or deletion)
                s.tiles[X][Y] = Tile(s.win,X,Y,s.placeType == 'Place',s.placeColour,s.placeRank)    #updates that square in array
                s.SetButtons()

	#Handles mouse clicks
    def clickInPlay(s,X,Y):
        #+-added X and Save clicked if statements
        if (10<=X<11 and -3<=Y<-2): #X clicked
            ExitGame(s.win)
        elif (8<=X<10 and -3<=Y<-2): #Save clicked
            s.SaveSetupToFile()
        elif (6<=X<8 and -3<=Y<-2): #Resign clicked
        #+-added message box indicating which player had quit/resigned
            tkMessageBox.showinfo("Resignation", str(s.pTurn) + ' has resigned! ' + str(s.opposite(s.pTurn)) + ' wins!')
            s.state = 'CustomSetup'
            s.SetButtons()
        elif (0<=X<8 and 0<=Y<8): #Tile Clicked in Play
            if s.selectedTileAt != []: #move if able
                if s.selectedTileAt[0] == X and s.selectedTileAt[1] == Y and not s.pieceCaptured: #Re-Selecting the already selected piece de-selects it
                    s.selectedTileAt = []
                    s.tiles[X][Y] = Tile(s.win,X,Y,s.tiles[X][Y].isPiece,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank)
                elif s.pTurn == s.tiles[X][Y].pieceColour and not s.pieceCaptured and (s.PieceCanCapture(X,Y) or not s.PlayerCanCapture()): 
                    s.tiles[s.selectedTileAt[0]][s.selectedTileAt[1]] = Tile(s.win,s.selectedTileAt[0],s.selectedTileAt[1],s.tiles[s.selectedTileAt[0]][s.selectedTileAt[1]].isPiece,s.tiles[s.selectedTileAt[0]][s.selectedTileAt[1]].pieceColour,s.tiles[s.selectedTileAt[0]][s.selectedTileAt[1]].pieceRank)
                    s.selectedTileAt = [X,Y]
                    s.tiles[X][Y] = Tile(s.win,X,Y,s.tiles[X][Y].isPiece,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank,isSelected=True)
                elif s.moveIsValid(s.selectedTileAt[0],s.selectedTileAt[1],X,Y):
#####################+-added extra code here
                    if s.tiles[X][Y].isPiece:
                        X=X+(X-s.selectedTileAt[0])
                        Y=Y+(Y-s.selectedTileAt[1])
############################################################                        
                    s.move(s.selectedTileAt[0],s.selectedTileAt[1],X,Y)
                    if not (s.pieceCaptured and s.PieceCanCapture(X,Y)):
                        s.pieceCaptured = False
                        s.selectedTileAt = []
                        s.pTurn = s.opposite(s.pTurn)
                        s.SetButtons()
                        #+-added if statement to check defeat
                        if s.movesAvailable() == [] and s.numColour(s.pTurn):
                            tkMessageBox.showinfo("Defeat", str(s.pTurn) + ' has no available moves! ' + str(s.opposite(s.pTurn)) + ' wins!')
                            s.state = 'CustomSetup'
                            s.SetButtons()

                else:
                    tkMessageBox.showinfo("Error", "Cannot perform that action.")
            else: #Select a Piece to move
                if s.pTurn != s.tiles[X][Y].pieceColour:
                    tkMessageBox.showinfo("Error", "Select a piece of current player's colour")
                elif (not s.PieceCanCapture(X,Y)) and s.PlayerCanCapture():
                    tkMessageBox.showinfo("Error", "Invalid selection, current player must take a piece")
                else:
                    s.selectedTileAt = [X,Y]
                    s.tiles[X][Y] = Tile(s.win,X,Y,s.tiles[X][Y].isPiece,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank,isSelected=True)

    #+-added to determine whether the tile attempting to be selected is valid
    def validTileSelect(s,X,Y):
        if (0<=X<8 and 0<=Y<8): #Tile Clicked in Play
            if s.selectedTileAt != []: #move if able
                if s.selectedTileAt[0] == X and s.selectedTileAt[1] == Y and not s.pieceCaptured: #Re-Selecting the already selected piece de-selects it
                    return False
                elif s.pTurn == s.tiles[X][Y].pieceColour and not s.pieceCaptured and (s.PieceCanCapture(X,Y) or not s.PlayerCanCapture()): 
                    return True
                elif s.moveIsValid(s.selectedTileAt[0],s.selectedTileAt[1],X,Y):
                    return False
                else:
                    return False
            else: #Select a Piece to move
                if s.pTurn != s.tiles[X][Y].pieceColour:
                    return False
                elif (not s.PieceCanCapture(X,Y)) and s.PlayerCanCapture():
                    return False
                else:
                    return True
        else:
            return False

    #+-added to determine whether the tile attempting to be moved to is valid
    def validTileMove(s,X,Y):
        if (0<=X<8 and 0<=Y<8): #Tile Clicked in Play
            if s.selectedTileAt != []: #move if able
                if s.selectedTileAt[0] == X and s.selectedTileAt[1] == Y and not s.pieceCaptured: #Re-Selecting the already selected piece de-selects it
                    return False
                elif s.pTurn == s.tiles[X][Y].pieceColour and not s.pieceCaptured and (s.PieceCanCapture(X,Y) or not s.PlayerCanCapture()): 
                    return False
                elif s.moveIsValid(s.selectedTileAt[0],s.selectedTileAt[1],X,Y):
                    return True
                else:
                    return False
            else: #Selecting a Piece to move
                return False
        else:
            False 

    def moveIsValid(s,x,y,X,Y): #parameters -> self,starting x,starting y,final X,final Y
        #+-added if statement to ensure it the selected piece's turn
        if s.tiles[x][y].pieceColour == s.pTurn:
            if s.tiles[X][Y].pieceColour == s.opposite(s.pTurn): #valid if can jump target piece
                return s.PieceCanCapturePiece(x,y,X,Y)
            elif s.PieceCanJumpTo(x,y,X,Y): #valid if can jump to target location
                return True
            elif s.CanDoWalk(x,y,X,Y) and not s.PlayerCanCapture(): #valid if piece can travel to X,Y normally and PlayerCanCapture==False
                return True
            else:
                return False

########
# This Function Enables a Piece to move. Trace Back --> Requirement 1.3
########
    def move(s,x,y,X,Y): #parameters -> self,starting x,starting y,final X,final Y      assumes valid move as input           
        s.tiles[X][Y] = Tile(s.win,X,Y,True,s.tiles[x][y].pieceColour,s.tiles[x][y].pieceRank)

        if (Y==7 and s.tiles[X][Y].isWhite) or \
           (Y==0 and s.tiles[X][Y].isBlack):
            s.tiles[X][Y].pieceRank = 'King'

        s.tiles[X][Y] = Tile(s.win,X,Y,True,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank)
        s.tiles[x][y] = Tile(s.win,x,y,isPiece=False)
        if X-x == 2 or X-x == -2:
            if s.numColour(s.tiles[x+(X-x)/2][y+(Y-y)/2].pieceColour) == 1:
                tkMessageBox.showinfo("Winner", str(s.tiles[X][Y].pieceColour) + ' Wins!')
                #+-updated to allow another game to be played after a winner is declared
                s.state = 'CustomSetup'
                s.SetButtons()
            s.tiles[x+(X-x)/2][y+(Y-y)/2] = Tile(s.win,x+(X-x)/2,y+(Y-y)/2,isPiece=False)

            s.tiles[X][Y] = Tile(s.win,X,Y,True,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank)
            if s.PieceCanCapture(X,Y):
                s.tiles[X][Y] = Tile(s.win,X,Y,True,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank,isSelected=True)

            s.selectedTileAt = [X,Y]
            s.pieceCaptured = True
        else:
            s.selectedTileAt = []
            
            s.tiles[X][Y] = Tile(s.win,X,Y,True,s.tiles[X][Y].pieceColour,s.tiles[X][Y].pieceRank)
            s.pieceCaptured = False
                 
#the below few functions need conditions added to handle out of bounds errors (for being off grid, i.e. 0<=X<8 or 0<=Y<8 doesn't hold)      <--- I think this is handled in PieceCanCapturePiece      
    def PlayerCanCapture(s):
        for i in range(s.BoardDimension):
            for j in range(s.BoardDimension):
                if s.pTurn == s.tiles[i][j].pieceColour: #Current piece belongs to current player
                    if s.PieceCanCapture(i,j):
                        return True
        return False

    def PieceCanCapture(s,x,y):
        X1,X2,Y1,Y2 = [x-1,x+1],[x-2,x+2],[y-1,y+1],[y-2,y+2]

        for i in range(2):
            for j in range(2):
                if s.PieceCanCapturePiece(x,y,X1[i],Y1[j]):
                    return True
        return False

#########
# This Function enables a tile to capture and remove an opponent piece. Trace Back --> Requirement 1.5
#########
    def PieceCanCapturePiece(s,x,y,X,Y):
        X1,X2,Y1,Y2 = [x-1,x+1],[x-2,x+2],[y-1,y+1],[y-2,y+2]
        #+-added first two if conditions
        if ((0<=X<8) and (0<=Y<8)) and ((0<=x<8) and (0<=y<8)):
            if (s.tiles[x][y].pieceColour == s.opposite(s.tiles[X][Y].pieceColour)):
                if s.CanDoWalk(x,y,X,Y,exception=True):
                    for i in range(2):
                        for j in range(2):
                            if X1[i]==X and Y1[j]==Y:
                                if (0<=X2[i]<8) and (0<=Y2[j]<8):
                                    if not (s.tiles[X2[i]][Y2[j]].isPiece):
                                        return True
        return False

############
# This Function enables a tile to jump over opponent piece. Trace Back --> Requirement 1.5
############
    def PieceCanJumpTo(s,x,y,X,Y):
        X1,X2,Y1,Y2 = [x-1,x+1],[x-2,x+2],[y-1,y+1],[y-2,y+2]
        for i in range(2):
            for j in range(2):
                if X2[i]==X and Y2[j]==Y:
                    if s.PieceCanCapturePiece(x,y,X1[i],Y1[j]):
                        return True
        return False

########
# This Function enables a king to move front and back. Trace Back --> Requirement 1.5
######## 
    def CanDoWalk(s,x,y,X,Y,exception=False): #The final parameter is to add a special case such that PieceCanCapturePiece may use this method
        X1,Y1 = [x-1,x+1],[y-1,y+1]

        for i in range(2):
            for j in range(2):
                if X1[i]==X and Y1[j]==Y:
                    if (0<=X<8) and (0<=Y<8):
                        if(s.tiles[x][y].isWhite and j==1) or \
                            (s.tiles[x][y].isBlack and j==0) or \
                            (s.tiles[x][y].isKing):
                            if not(exception or s.tiles[X][Y].isPiece) or \
                               (exception and s.tiles[X][Y].isPiece and \
                               (s.pTurn != s.tiles[X][Y].pieceColour)):
                                return True
        return False

##########
# This Function launches the standard/original setup of the board. Trace Back --> Requirement 1.1
##########
    def StandardSetup(s): #defines the standard button
        s.ClearBoard()
        s.state = 'CustomSetup' #in custom mode
        for i in range(s.BoardDimension):
            for j in range(s.BoardDimension):
                if s.tiles[i][j].TileColour(i,j) == 'Red' and (j < 3):
                    s.tiles[i][j] = Tile(s.win,i,j,True,'White','Pawn')
                if s.tiles[i][j].TileColour(i,j) == 'Red' and (j > 4):
                    s.tiles[i][j] = Tile(s.win,i,j,True,'Black','Pawn')
                    #places all the pieces in default checkers postitions

    def numColour(s,colour): #counts the number of pieces of a given colour
        c = 0 #initiate counter
        for i in range(s.BoardDimension):
            for j in range(s.BoardDimension):
                if colour=='White' and s.tiles[i][j].isWhite:
                    c += 1
                elif colour=='Black' and s.tiles[i][j].isBlack:
                    c += 1
        return c

    def opposite(s,opp): #Returns the 'opposite' of a given parameter (only works for specific things)
        if opp == 'White':
            return 'Black'
        elif opp == 'Black':
            return 'White'
        
        elif opp == 'King':
            return 'Pawn'
        elif opp == 'Pawn':
            return 'King'

        elif opp == 'Place':
            return 'Delete'
        elif opp == 'Delete':
            return 'Place'
        else:
            #+-returns instead of printing error message
            return opp
        
    #####
    #function returns the bottom left coordinate of the square clicked
    #####
    def ClickedSquare(s,click):   
        try:
            clickX = click.getX()
            clickY = click.getY()
            if clickX < 0:
                clickX = int(clickX)-1
            else:
                clickX = int(clickX)
            if clickY < 0:
                clickY = int(clickY)-1
            else:
                clickY = int(clickY)
            return clickX, clickY
        except IndexError:          #some positions on the outskirts of the screen are invalid locations
            s.Click()

#######
# This Function Saves the game to be resumed later. Trace back --> Requirement 1.6
#######
    def SaveSetupToFile(s):   #method writes the tiles array to file checkers.txt
        # can have a dialog box to ask for the text file name to save to
        saveFile = open ('checkers.txt' , 'w') #opens file to write
        for i in range(s.BoardDimension):
            for j in range(s.BoardDimension):
                if (s.tiles[i][j].isPiece):
                    i_string = str(i)
                    j_string = str(j)
                    saveFile.write(i_string + j_string + str(s.tiles[i][j].pieceColour)[0] + \
                                   str(s.tiles[i][j].pieceRank)[0] + "\n")
        saveFile.write(s.pTurn[0]) #saves whose turn it is too
        tkMessageBox.showinfo("Saved Complete", "Game setup was saved to checkers.txt")
        saveFile.close()

##########################
#Function LoadSetupFromFile(s) corresponds to Requirement 1.2
##########################
    def LoadSetupFromFile(s): #method gets the setup saved and places pieces accordingly
        loadFile = open ('checkers.txt' , 'r') #opens file to read
        piece_list = loadFile.readlines()
        tkMessageBox.showinfo("Loading", "Will now clear the board and \nplace the saved setup")
        s.ClearBoard()
        for i in range(len(piece_list) - 1):
            tot_string = piece_list[i]
            x_var = int(tot_string[0])
            y_var = int(tot_string[1])
            #checks the text file, with these specific letters signifying what each piece is
            #first letter - 'W' is white, 'B' is black
            #second letter - 'K' is a King piece, 'P' is a pawn piece
            if (tot_string[2] == 'W'): #it is a white piece
                if (tot_string[3] == 'K'): #it is a white King piece
                    s.tiles[x_var][y_var] = Tile(s.win,x_var,y_var,True,'White','King')
                else : #piece is a white pawn
                    assert(tot_string[3] == 'P')
                    s.tiles[x_var][y_var] = Tile(s.win,x_var,y_var,True,'White','Pawn')
            else: #piece is black
                assert(tot_string[2] == 'B')
                if (tot_string[3] == 'K'): #piece is a black King
                    s.tiles[x_var][y_var] = Tile(s.win,x_var,y_var,True,'Black','King')
                else: #piece is a black pawn
                    assert(tot_string[3] == 'P')
                    s.tiles[x_var][y_var] = Tile(s.win,x_var,y_var,True,'Black','Pawn')
        #whose turn it was is restored
        if (piece_list[len(piece_list)-1] == 'W'): #it is white's turn
            s.pTurn = 'White'
        else: #it is black's turn
            s.pTurn = 'Black'
        s.SetButtons()
        loadFile.close()


'''
# We took out Class Piece from Checkers_v6 and implemented Class Tile in Checkers_v18.
    This was done to ensure a seamless transition for when the game actually ran.
'''
##########
#defines a tile and holds its current state
##########

class Tile:
    def __init__(s,win,X,Y,isPiece,pieceColour='',pieceRank='',isSelected=False):
        s.win = win
        s.x = X
        s.y = Y
        s.isPiece = isPiece
        s.isWhite = ('White' == pieceColour) and s.isPiece
        s.isBlack = ('Black' == pieceColour) and s.isPiece
        s.isKing = ('King' == pieceRank) and s.isPiece
        s.isPawn = ('Pawn' == pieceRank) and s.isPiece

        s.pieceColour=''
        s.pieceRank=''
        if s.isWhite:
            s.pieceColour = 'White'
        elif s.isBlack:
            s.pieceColour = 'Black'
        if s.isKing:
            s.pieceRank = 'King'
        elif s.isPawn:
            s.pieceRank = 'Pawn'

        s.c = Point(s.x+.5,s.y+.5)
        s.circ = Circle(s.c,0.4)

        if isSelected:
            s.circ.setOutline('Yellow')
        else:
            s.circ.setOutline('Black')
            s.circ.undraw()

        s.kingTxt = Text(s.c,'K')
        s.kingTxt.undraw()

        s.ColourButton(s.TileColour(s.x,s.y),s.x,s.y)
        if s.isPiece:
            s.DrawPiece()
##########
#function to create a rectangle with a given colour, size, and location
##########
    def ColourButton(s,colour,X,Y,width=1,height=1):
        rect = Rectangle(Point(X,Y),Point(X+width,Y+height))
        rect.setFill(colour)
        rect.draw(s.win)
        
##########
#Tiles on the board
##########        
    def TileColour(s,x,y):
        if (x%2 == 0 and y%2 == 0) or (x%2 == 1 and y%2 == 1):
            return 'Red'   #sets every other square to red 
        else:
            return 'White' #every non red square to white

##########
#Displays White or Black or King Pieces
##########
    def DrawPiece(s):
        s.circ.draw(s.win)

        if s.isWhite:
            col1,col2 = 'White','Black'
        elif s.isBlack:
            col1,col2 = 'Black','White'

        s.circ.setFill(col1)

        if s.isKing:
            s.kingTxt.draw(s.win)
            s.kingTxt.setFill(col2)

##########
#Quit
##########
def ExitGame(win):
    win.close()
    sys.exit()
    
game = Checkers()
```

This is a very good example of my experiance with algorithms and data structures being that it is from an articifical intelligence algorithm for playing checkers. I included this for two very good reasons. It shows my ability to fix and manage an already existing code base and shows my progress in reading someone elses code which I struggled with before coming to SNHU and getting some much needed practice. Being able to modify and manage an already existing code base is something I find very important for a programmer because that is something we will have to do a ton when we become professionals. Either with already created code or helping peers on a project we will always have to be reading others code. I learned some valuable lessons when I was working on this in class and then enhancing it for this portfolio. Taking already created code and editing it to make it better is a hard task mostly because it is not my code and you have to trace down issues and then implement fixes or changes altogehter. Overall I am happy with what I have produced.

### Databases

```
update.py

import json
from bson import json_util
from bson.json_util import dumps
from pymongo import MongoClient
connection = MongoClient('localhost', 27017)
db = connection['market']
collection = db['stocks']

class Database:
  
  def check_document(self,ticker):
    query = {"Ticker" : ticker}
    result=collection.find_one(query)
    print(dumps(result))
  
  def update_using_single_Ticker(self,query,new_update):
    try:
      collection.update_many(query,new_update)
      result=collection.find(query)
      print("*" * 50)
      print(dumps(result))
    except ValidationError as ve:
      abort(400, str(ve))
      
  def update_using_Multiple_Ticker(self,file_name):
    with open(file_name) as json_file:
      for line in json_file:
        ticker = line
        volume = raw_input("New Volume For Ticker["+line+"]>")
        query = {"Ticker" : ticker}
        print("*" * 50)
        print("Below Documents Will Be Updated...")
        result=collection.find(query,{"Ticker":1,"Industry":1})
        print(dumps(result))
        new_update =  { "$set":{"Volume":volume}}
        
        try:
          collection.update_many(query,new_update)
          result=collection.find(query)
          print("*" * 50)
          print("Below Documents Have Been Updated...")
          print(dumps(result))
        except ValidationError as ve:
          abort(400, str(ve))
      
      
  def menu(self):
    print("-----------------------")
    print("0. Confirm Document(s)")
    print("1. Update Using Multiple Tickers From File")
    print("2. Update Using Single Ticker")
    print("3. Quit")
  

def main():
  
  db = Database()
  line = "--" * 50
  print(line+"\n")
  print("|")
  print("|                   Update Documents Using Tickers          |")
  print(line+"\n")
  checker = True
  while checker:
    db.menu()
    choice = raw_input("Enter Choice ~")
    if choice == "1":
      
      file_name = raw_input("Enter File Name(Format e.g file_name.txt)")
      db.update_using_Multiple_Ticker(file_name)
      print(line+"\n\n")
      
    elif choice == "2":
      ticker = raw_input("Enter Ticker Value:")
      volume = raw_input("New Volume For Ticker["+ticker+"]>")
      query = {"Ticker" : ticker}
      print("*" * 50)
      print("Below Documents Will Be Updated")
      result=collection.find(query,{"Ticker":1,"Industry":1})
      print(dumps(result))
      
      new_update =  { "$set":{"Volume":volume}}
      db.update_using_single_Ticker(query,new_update)
      
    elif choice == "3":
      print("Good Byeee")
      checker = False
      
    elif choice == "0":
      ticker = raw_input("Enter Ticker Value:")
      db.check_document(ticker)
      
    else:
      print("Invalid Choice")
      
  
main()
```

```
insert.py

import json
from bson import json_util
from pymongo import MongoClient
connection = MongoClient('localhost', 27017)
db = connection['market']
collection = db['stocks']

class Database:
  
  def insert_from_file(self,file_name):
    with open(file_name) as json_file:
      
      for line in json_file:
        print(line)
        line = json.loads(line)
        try:
          result = collection.insert(line)
          print("New Data Inserted...")
        except Exception as e:
          print(str(e))
    
  def insert_new_document(self,document_data):
    is_added = True
    
    try:
      
      result = collection.insert(document_data)
      print("<< Thank You, Document Inserted >>")
      
    except Exception as e:
      print(str(e))
      is_added = False
      
    
    return is_added
  
  def menu(self):
    print("-----------------------")
    print("1. Data From File")
    print("2. Data Standard Input")
    print("3. Quit")

def main():
  db = Database()
  line = "--" * 50
  print(line+"\n")
  print("|")
  print("|                   Inserting New Document Into Stocks Collection Market Database           |")
  print(line+"\n")
  checker = True
  while checker:
    db.menu()
    choice = raw_input("Enter Choice ~")
    if choice == "1":
      file_name = raw_input("Enter File Name(Format e.g file_name.txt)")
      db.insert_from_file(file_name)
      print(line+"\n\n")
    elif choice == "2":
      new_document = raw_input("Enter Your Document:")
      print("You Requested To Add This Docuument \n"+line)
      print(new_document)
      print (db.insert_new_document(json.loads(new_document)))
      print(line+"\n\n")
    elif choice == "3":
      print("Good Byeee")
      checker = False
    else:
      print("Invalid Choice")
  
main()
```
These scripts are written in python to insert and update data within a mongo db. Specifically these were created to work with the restful_api to work on its mongo db. This is a very important artifact for databases to show that I know how to create an interface for databases and then understanding databases as well. 
