# John Conways' game of life in python
<p align='center'> <img src="https://upload.wikimedia.org/wikipedia/commons/e/e5/Gospers_glider_gun.gif"> </p>

This small project represents the python Implementation of *John Conways'* game of life, if you don't know what I'm talking about stick with me, I'll explain everything from idea to implementation.

Project repo: [Game Of Life](https://github.com/Zowlex/Python-projects)

Table of contents:

1. TOC
{:toc}

## What is the game of life 
 
 According to [Wikipedia](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) the game of life is in simple terms a game that mimics the behavior and lifecycle of biological cells in nature following 4 simple rules:
  1. Any live cell with fewer than two live neighbours dies, as if by underpopulation.
  2. Any live cell with two or three live neighbours lives on to the next generation.
  3. Any live cell with more than three live neighbours dies, as if by overpopulation.
  4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

Starting from these simple rules we can create starting configurations that lead to complex systems and shapes like [oscillators](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life#/media/File:Game_of_life_pulsar.gif), [spaceships](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life#/media/File:Animated_Hwss.gif) and [turing machines](https://en.wikipedia.org/wiki/Hashlife#/media/File:Turing_Machine_in_Golly.png)


## Motivation

I was glad that I stumbled upon a great talk on youtube called [The Art of Code](https://www.youtube.com/watch?v=gdSlcxxYAA8). Dylan Beattie is aprogrammer and musician talked about different beautiful artistic moments of coding and what coders have achieved from writing mesmerizing [quines](https://en.wikipedia.org/wiki/Quine_(computing)) to coming up with weird but funny programming languages like [Rockstar programming language](https://codewithrockstar.com/) but what has really caught my attention is talking about the game of life which made my jaw drop and let me think very deeply about its capabilities. At this moment I knew nothing about it and suddenly had the urge to implement it in my favourite programming language (python)

## Implementation

### Engine

At this point I have no idea how to implement this graphically so I started first by creating a Grid class which represents our infinte 2d space. This grid is in fact a 2d array which translates to a nested list in python. our grid of length w and width h is initialized by w*h zeros when called.

{% include info.html text="N.B: 0 means that the cell is dead and 1 is alive" %}

class Grid:
```python
import random

class Grid(list):
    def __init__(self, w, h):
        """
        initializes a grid of zeros with length w and width h 
        """
        self.w = w
        self.h = h
        super()
        for row in range(w):
        # Add an empty array that will hold each cell
        # in this row
            self.append([])
            for column in range(h):
                self[row].append(0)  # Append a cell
    
    def show(self):
        for row in range(self.w):
            ch='['
            for col in range(self.h):
                ch += str(self[row][col])+','
            if col == self.h-1:
                ch=ch[:-1]+']'
                print(ch,'\n') 
```
Example:
```python
grid = Grid(5,5)
grid.show()
```
Output:
  
  [0,0,0] 

  [0,0,0] 

  [0,0,0] 

Now that we have defined and created our data structure the next task is to define a function to calculate the number of alive neighbors for each cell of position x,y by visiting  its next eight neighbors and summing up their values. 
```python
 def alive_neighbors(self,x,y):
        """
        returns the number of alive neighbors (=1) for a certain x,y position
        """
        res = 0
        for row in range(x-1,x+2):
            for col in range(y-1,y+2):
                if (row==x) and (col==y):
                    continue
                try:
                    if self[row][col] == 1:
                        res+=1
                        pass
                except IndexError:#If this error is raised for cells on the edges we consider the next edge cells are the neighbors, like we are wrapping a sheet of paper so the edges touch 
                    row = (x+row+self.w)%self.w
                    col = (y+col+self.h)%self.h
                        
        return res
```
Example:
  
  [0,1,0] 

  [0,0,0] 

  [1,1,1]
```python
neighbors = grid.alive_neighbors(1,1) # cell in position 1,1
print(neighbors)
```
Output:
 4 which is the sum of neighbor alive cells

Before thinking about gui we have to try it on terminal and see if everything works fine. We start by making a grid and choose starting configuration, get each cell's state and update it in next_grid, after looping through all cells we print the updated version (next_grid) and each loop through the grid represnts a new generation of cells.

test.py
```python
from grid import Grid


grid = Grid(3,5)

grid[1][1] = 1
grid[1][2] = 1
grid[1][3] = 1



if __name__ == '__main__':
    grid.show()
    next_grid = Grid(3,5)
    for row in range(3):
        for col in range(5):

            state = grid[row][col]
            #get every element's neighbors of grid
            neighbors = grid.alive_neighbors(row,col)
            
            if neighbors == 3 and state ==0 :# These are the first 3 rules stated above
                next_grid[row][col] = 1
            elif (neighbors<2 or neighbors>3) and state ==1 : # This is the last rule of game of life
                next_grid[row][col] = 0
            else:
                next_grid[row][col] = state

    print('='*10)
    next_grid.show()        
    grid = next_grid
```
Output: this is the representation of an oscillator 

[0,0,0,0,0]   ===>     [0,0,1,0,0] 

[0,1,1,1,0]   ===>     [0,0,1,0,0]

[0,0,0,0,0]   ===>     [0,0,1,0,0]
 
### Interface

For the gui I decided to use pygame for its simplicity and powerful functionalities, we start by defining some global variables.
```python
# Define some colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)

# This sets the WIDTH and HEIGHT of each grid location
SQUARE = 20

# This sets the margin between each cell
MARGIN = 2

#Screen resolution
w = 700
h = 500

#calculate row_num, col_num from current resolution
row_num = w//SQUARE
col_num = h//SQUARE
```
Next define the draw method which represents our grid of zeros and ones by graphical squares of white color if dead and green color if alive:
```python
#def initialize grid
def draw_grid(row_num, col_num, g):
                # Draw the grid
                for row in range(row_num):
                    for column in range(col_num):
                        color = WHITE
                        pygame.draw.rect(screen,
                                            color,
                                            [(MARGIN + SQUARE) * row + MARGIN,
                                            (MARGIN + SQUARE) * column + MARGIN,
                                            SQUARE,
                                            SQUARE])
                        if g[row][column] == 1:
                            pygame.draw.rect(screen,
                                    GREEN,
                                    [(MARGIN + SQUARE) * row + MARGIN,
                                    (MARGIN + SQUARE) * column + MARGIN,
                                    SQUARE,
                                    SQUARE])
                pygame.display.flip()
```


Then we have to create a window of size w,h and draw initial grid
```python
def create_window(w, h):
    # Set the HEIGHT and WIDTH of the screen
    screen = pygame.display.set_mode([w,h])
    # Set title of screen
    pygame.display.set_caption("John Conway's game of life")
    return screen

#create window of size 700px by 500px
screen = create_window(w, h)

#draw initial grid
draw_grid(row_num, col_num, grid)  
```
The final step is defining the main loop to keep the game running and see some **magic patterns**, once we press the space key we apply the same rules on the grid like we did on the first prototype of the game and draw each new generation of cells to the screen of our window.
```python
# -------- Main Program Loop -----------
while not done:
    for event in pygame.event.get():  # User did something
        if event.type == pygame.QUIT:  # If user clicked close
            done = True  # Flag that we are done so we exit this loop
        elif event.type == pygame.KEYDOWN: # Game starts
            if event.key == K_SPACE:
                print('space key pressed')
                x = 0
                while True:
                    next_grid = Grid(row_num, col_num)
                    for row in range(row_num):
                        for col in range(col_num):

                            state = grid[row][col]#current grid state 0 or 1
                            
                            neighbors = grid.alive_neighbors(row,col)#current grid alive neighbors
                            
                            if state == 1 and (neighbors<2 or neighbors>3):
                                next_grid[row][col] = 0
                            elif state == 0 and neighbors == 3: 
                                next_grid[row][col] = 1
                            else:
                                next_grid[row][col] = state
                                
                    draw_grid(row_num, col_num, next_grid)
                    grid = next_grid
                    x+=1
                    print('generation:',x)
                    # Limit to x frames per second
                    clock.tick(x)

# Be IDLE friendly. If you forget this line, the program will 'hang'
# on exit.
pygame.quit()
```
Final Result of a random config:
<p align='center'> <img src="https://github.com/Zowlex/Python-projects/blob/master/gameoflife/screenshots/gol.gif"> </p>

## Conclusion
I had so fun working on this project and I hope it helps you because at first place I did not find a good python implementation of this project.
 
 ### Next Steps:
   - Add drawing functionality by mouse, so you can choose easily the starting config
   - I think of optimizing the game by using numpy instead of grid 
   - Make an interactive web app of it using flask and deploy it
