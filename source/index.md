---
title: BattleShip

language_tabs:
  - ruby

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>


search: true
---

# Introduction

Basic Battleship game made on Ruby. Local comunication achieved via TubeSocks.

## Rules

Set 5 ships down and take down your opponent's before they can take yours. Fire one shot at a time by declaring a coordinate on your opponent's map. 


#Board Class

Each player has his or her own board as well as a board representing the
opponent's board.
A player's Board displays where their ships are, and an enemie's Board 
represents places where the player has attacked.

```ruby
  def initialize()
    @board = Matrix.build(11,11) {|row, col| "[-]"}
    for i in 1..10
      @board[0,i] = "[#{i}]"
      @board[i,0] = "[#{i}]"
    end
    @hits = 0
    @hp = 0
    @ships = 0
    @name= "?"
  end
```
##setName
Basic set function for the name variable

```ruby
def getName()
  return @name
end
```

##hit
Reieves an x and a y coordinate.
Checks to see if the space that matches those coordinates is ocupied
by a ship. If it is, the function report a "[X]", signifing a hit.
Anything else, returns a "[0]", indicating a miss.
If the hit connects and it is the last remaining Ship space,
hit returns "@name,death", signifing the end of the game.

```ruby
def hit(x, y)
  x = x.to_i
  y = y.to_i
  if @board[x,y] == "[-]" || @board[x,y] =="[X]"
    @board[x,y] = "[0]"
    return "#{@name},report,[0],#{x},#{y}"
  else
    @board[x,y] = "[X]"
    @hits = @hits + 1
    if(@hits == @hp)
      return "#{@name},death"
    end
    return "#{@name},report,[X],#{x},#{y}"
  end
end
```

##printAll
Turns the board into an array for easier printing.

```ruby
def printAll()
  arrenglones = []
  for i in 0..11
    renglon ="" 
    for j in 0..11
      renglon = renglon.to_s + @board[i,j].to_s
    end
    arrenglones[i] = renglon
  end
  return arrenglones
end
```

##putsShips
Function in charge of validating and setting a ship in place.
Recieves the following parameters:
* x/y = x and y coordinates for the start point of the ship.
* n = Name of the ship.
* o = Orientation for the ship, V for vertical and H for Horizontal
* l = length of the ship.  

```ruby
def putsShips(x, y, n, o, l)
  #5 is the maximum number of ships.
  if(@ships==5)
    return true
  end
  l = l.to_i - 1
  x = x.to_i
  y = y.to_i

  #Validate the ship's position.
  if(@board[x.to_i,y.to_i] == "[-]")
  if(o == "V")
      if(x+l >= 11)
        puts "Error, espacio inadecuado"
        return true
      end
      for i in 0..l
        if(@board[i+x,y] != "[-]")
          puts "Espacio ya ocupado"
          return true
        end
      end
    else(o =="H")
      if(y+l >= 11)
        puts "Error, espacio inadecuado"
        return true
      end
      for i in 0..1
        if(@board[x,y+1] != "[-]")
          puts "Espacio ya ocupado"
          return true
        end
      end
    end
  else
    puts "Espacio ya ocupado"
    return true
    end

  #All clear!
  #Set the ship in place
  if(o == "V")
    for i in 0..l
      @hp = @hp + 1
      @board[x+i,y] = "[#{n}]"
    end
  else(o =="H")
    for i in 0..l
      @hp = @hp + 1
      @board[x, y+i] = "[#{n}]"
    end
  end
  @ships = @ships.to_i + 1
  return false
end
```


##Game Over
Reiceves 1 for victory and 0 for failure.
If the board is the losing board, set all values to "[X]"
If the board won, set "[0]"

```ruby
def gameOver(x)
  x = x.to_i
  if(x == 1)
    @board = Matrix.build(11,11) {|row, col| "[0]"}
  elif(x == 0)
    @board = Matrix.build(11,11) {|row, col| "[X]"}
  end
end
```

#SendEvents
Encompasses all the functions related with reading player input.
Used to be a 150 long method. It got better.

##EventDispatch
In charge of recieving the basic input and calling the apropriate function.

```ruby
def self.EventDispatch(inst)
  if(inst[0].to_s == GlobalConstants::BOARDB.getName)  
    if(inst[1].to_s == "SetS")
      SetShipEvent(inst, "H")
    elsif(inst[1].to_s == "Hit")
      HitEvent(inst, "H")
    end
  elsif(inst[0].to_s == GlobalConstants::BOARDA.getName)
    if(inst[1].to_s == "SetS")
      SetShipEvent(inst, "M")
    elsif(inst[1].to_s == "Hit")
      HitEvent(inst, "M")
    end
  end
end
```

##SetShipEvent
Gets the global boards and attempts to put ships in them.

```ruby
def self.SetShipEvent(inst, who)
  if(who.to_s == "H")
    GlobalConstants::BOARDB.putsShips(inst[2].to_i, inst[3].to_i, inst[4].to_s, inst[5].to_s, inst[6].to_i)
  elsif(who.to_s == "M")
    GlobalConstants::BOARDA.putsShips(inst[2].to_i, inst[3].to_i, inst[4].to_s, inst[5].to_s, inst[6].to_i)
  end
end
```


##HitEvent
Sends a hit for a player.
If the hit is a KO, ends the game.

```ruby
def self.HitEvent(inst, who)
  if(who.to_s == "H")
    result= GlobalConstants::BOARDA.hit(inst[2].to_s, inst[3].to_s)
    res = result.split(",")
    if(res[1].to_s == "death")
      winner = GlobalConstants::BOARDB.getName
      loser = GlobalConstants::BOARDA.getName
      userW =  User.find_by_name(winner)
      
      userW.win = userW.win + 1
      userW.save
      
      userL =  User.find_by_name(loser)
      userL.lose= userL.lose + 1
      userL.save

      GlobalConstants::BOARDAAUX.gameOver(0)
      GlobalConstants::BOARDBAUX.gameOver(1)
      GlobalConstants::BOARDA.gameOver(1)
      GlobalConstants::BOARDB.gameOver(0)
    else
      Redis.new.publish "chat", result
    end
  elsif(who.to_s =="M")
    result= GlobalConstants::BOARDB.hit(inst[2].to_s, inst[3].to_s)
      res = result.split(",")
      if(res[1].to_s == "death")
      winner = GlobalConstants::BOARDB.getName
      loser = GlobalConstants::BOARDA.getName
      userW =  User.find_by_name(winner)
      userW.win = userW.win + 1
      userW.save


      userL =  User.find_by_name(loser)
      userL.lose= userL.lose + 1
      userL.save
      
      GlobalConstants::BOARDBAUX.gameOver(0)
      GlobalConstants::BOARDAAUX.gameOver(1)
      GlobalConstants::BOARDB.gameOver(1)
      GlobalConstants::BOARDA.gameOver(0)
      else
       Redis.new.publish "chat", result
     end
  end
end
```

#Matrix

Simple new function to help assign values in a matrix.

```ruby
class Matrix
  #[]
  #Allows easier setting of the values of a specific Matrix spot
  def []= (row, column, value)
  @rows[row][column] = value
  end
end
```


#Controllers
Of them all, chat_controller and sessions_controller are the most important.


##chat_controller

Connects the app to the Ralis chat. Sends and recieves most messages, and as such, is in charge of showing both boards.


```ruby
class ChatController < ApplicationController
  include Tubesock::Hijack
  require 'gameclasses'


  ##Chat
  #In charge of all chat display
  def chat
    hijack do |tubesock|
      redis_thread = Thread.new do
        Redis.new.subscribe "chat" do |on|
          on.message do |channel, message|
            tubesock.send_data message
          end
        end
      end

      tubesock.onmessage do |m|
        boardAaux = GlobalConstants::BOARDA
        boardBaux = GlobalConstants::BOARDB
       
        SendEvents.EventDispatch(m.split(","))

        boardAauxRenglon = []
        boardBauxRenglon =[]
        if(current_user.name != nil)
          Redis.new.publish "chat", "#{boardAaux.getName}'s BOARD * --- * --- * --- * --- * --- * --- * --- * #{boardBaux.getName}'s BOARD"

          if(current_user.name.to_s == boardAaux.getName)
             boardAauxRenglon = boardAaux.printAll
              boardBauxRenglon = GlobalConstants::BOARDAAUX.printAll
            Redis.new.publish "chat", "#{boardAauxRenglon[0]} * --- * --- * --- * #{boardBauxRenglon[0]}"
            for i in 1..10
              Redis.new.publish "chat", "#{boardAauxRenglon[i]}  * --- * --- * --- * --- * #{boardBauxRenglon[i]}"
            end
          else
            boardAauxRenglon = GlobalConstants::BOARDBAUX.printAll
            boardBauxRenglon = boardBaux.printAll
              Redis.new.publish "chat", "#{boardAauxRenglon[0]} * --- * --- * --- * #{boardBauxRenglon[0]}"
            for i in 1..10
              Redis.new.publish "chat", "#{boardAauxRenglon[i]}  * --- * --- * --- * --- * #{boardBauxRenglon[i]}"
            end
          end
        end
      end
      
      tubesock.onclose do
        # stop listening when client leaves
        redis_thread.kill
      end
    end
  end
end
```

##Sessions controller

Logs in the user with the Database. Also in charge of setting the boards with the player name; first player to log in takes BoardA. 


```ruby
class SessionsController < ApplicationController
  def new
  end


  # Create a new session, validating name/password combination
  def create
     user = User.find_by(name: params[:session][:name])
    if user && user.authenticate(params[:session][:password])
     log_in user
    

  # Set the board name in the chat as the current user.  
      current_user = User.find(session[:user_id])
      if(GlobalConstants::BOARDA.getName.to_s == "?")
        GlobalConstants::BOARDA.setName(current_user.name)
      else
        GlobalConstants::BOARDB.setName(current_user.name)
      end
  
  # Remembers the user if the "remember me"  box is checked. 
      params[:session][:remember_me] == '1' ? remember(user) : forget(user)
     redirect_to user
    else
      flash.now[:danger] = "Invalid name/password combination"
      render 'new'
    end
  end


  # Destroy the current session, (log out the user).
  def destroy
    log_out if logged_in?
    redirect_to root_url

  end
end
```

##UsersController

Basic functions related to manipulating the user.


```ruby
class UsersController < ApplicationController

  # Method to show the stats of the user logged in.
  def show
    @user = User.find(params[:id])
  end

  # Method to create a new user.
  def new
    @user = User.new
  end
 
  # Method that create a new user and initialize it with its corresponding attributes.
  def create
    @user = User.new(user_params)
    @user.win=0
    @user.draw=0
    @user.lose=0
    if @user.save
      log_in @user
      flash[:success] = "Welcome to BattleShip"
      redirect_to @user
    else
      render 'new'
    end
end

private


 # Method that validates that only name, password and password confirmation can be submitted.
def user_params
  params.require(:user).permit(:name,:password,:password_confirmation)
end

end
```


#GlobalConstants

Allow for easy access to the Boards from any point. Four boards are used: A, B, AAUX, and BAUX. 
Boards A and B are the main two boards where each player stores his or her ships.
AAUX and BAUX  


```ruby
module GlobalConstants
 require 'gameclasses'
 BOARDA = Board.new()
 BOARDB = Board.new()

 #Aux boards are what any player sees of their opponent's board.
 #Thus you can only see where you have fired a shot.
 BOARDAAUX = Board.new()
 BOARDBAUX = Board.new()
end
```


