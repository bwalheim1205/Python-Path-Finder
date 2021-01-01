# Python Path Finder
 This python script uses the Pygame module to create a GUI to visualize different path finidng algorithms.
 
 Currently Represented Algorithms:
 - A* Path Find Algorithm
# Installation
Uses these commands to install proper modules for repository
```
pip install pygame
pip install queue
```
# Grid Node Class
This represents one node on the grid. Holds information about the tile such as row and col node is located in grid, width of node and total rows in the grid. Also holds x and y cordinate on GUI and color which represents different status of the node.
```python
class GridNode:

    '''
    Constructor creates new instance of GridNode
    Input:
        row - row node is in
        col - col node is in
        width - width of node tile
        total_rows - total rows in grid
    '''
    def __init__(self, row, col, width, total_rows):
        self.row = row
        self.col = col
        self.x = row*width
        self.y = col *width
        self.width = width
        self.total_rows = total_rows
        self.color = WHITE
```
Checks of the color of the node to determine it's to the status 
-
Red - Closed
Green - Opened
Black - Barrier
Orange - Start
Purple - End
White - None
```python
    #Node Status Checkers

    '''
    Checks to see if color is red and node is closed
    Output:
        True if red/closed
    '''
    def is_closed(self):
        return self.color == RED

    '''
    Checks to see if color is white and node is open
    Output:
        True if white/open
    '''
    def is_open(self):
        return self.color == GREEN

    '''
    Checks to see if color is black and node is barrier
    Output:
        True if black/barrier
    '''
    def is_barrier(self):
        return self.color == BLACK

    '''
    Checks to see if color is orange and node is start
    Output:
        True if orange/start
    '''
    def is_start(self):
        return self.color == ORANGE

    '''
    Checks to see if color is purple and node is end
    Output:
        True if purple/end
    '''
    def is_end(self):
        return self.color == PURPLE
```
Status Changes
-
Changes color of node according to the status. Color seen above
```python
    '''
    Changes node color to white signifying reset
    '''
    def reset(self):
        self.color = WHITE

    '''
    Changes node color to red signifying closed
    '''
    def make_closed(self):
        self.color = RED
    
    '''
    Changes node color to orange signifying start
    '''
    def make_start(self):
        self.color = ORANGE

    '''
    Changes node color to purple signifying end node
    '''
    def make_end(self):
        self.color = PURPLE

    '''
    Changes node color to black signifying barrier
    '''
    def make_barrier(self):
        self.color = BLACK

    '''
    Changes node color to green signifying open
    '''
    def make_open(self):
        self.color = GREEN

    '''
    Changes node color to cyan signifying path
    '''
    def make_path(self):
        self.color = CYAN
```
Algorithm Functions
-
```
    # Node Functions

    '''
    Gets position of node grid
    Output:
        (row, col)
    '''
    def getPos(self):
        return self.row, self.col

    '''
    Fills neighbors array with adjacent nodes that aren't barriers
    '''
    def update_neighbors(self, grid):

        #Empty neighbors node
        self.neighbors = []

        #Checks if node above exists and is not barrier
        if self.row < self.total_rows - 1 and not grid[self.row + 1][self.col].is_barrier(): # DOWN
            self.neighbors.append(grid[self.row + 1][self.col])

        #Checks if node below exists and is not barrier
        if self.row > 0 and not grid[self.row - 1][self.col].is_barrier(): # UP
            self.neighbors.append(grid[self.row - 1][self.col])

        #Checks if node to right exists and is not barrier
        if self.col < self.total_rows - 1 and not grid[self.row][self.col + 1].is_barrier(): # RIGHT
            self.neighbors.append(grid[self.row][self.col + 1])

        #Checks if node to left exists and is not barrier
        if self.col > 0 and not grid[self.row][self.col - 1].is_barrier(): # LEFT
            self.neighbors.append(grid[self.row][self.col - 1])
```
Pygame Functions
-
Draws the node in the window
```python
    '''
    Draws node on pygame display
    Input:
        win - pygame window
    '''
    def draw(self, win):
        pygame.draw.rect(win, self.color, (self.x, self.y, self.width, self.width))
```
# Grid Class
This represents the grid which is composed of 2D aray of GridNodes. It handles the drawing of grid and accessing of nodes.
```python
class Grid:

    '''
    Constructor creates new instance of Grid
    Input:
        rows - number of rows in Grid
        cols - number of cols in Grid
        width - Width of the screen
    '''
    def __init__(self, rows, cols, width):
        
        self.rows = rows
        self.cols = cols
        self.width = width
        
        #Populates grid with node objects
        self.grid = []
        gap = width//rows
        for i in range(rows):
            self.grid.append([])
            for j in range(cols):
                self.grid[i].append(GridNode(i, j, gap, self.rows))

    '''
    Draws the grid on pygame display
    Input:
        win - pygame display window object
    '''
    def draw_grid(self, win):

        #Fills Background with white
        win.fill(WHITE)

        #Draws Nodes
        for row in self.grid:
            for node in row:
                node.draw(win)

        #Draws grid lines
        gap = self.width // self.rows
        for i in range(self.rows):
            #Draws horizontal lines
            pygame.draw.line(win, GRAY, (0, i * gap), (self.width, i * gap))
            for j in range(self.rows+1):
                #Draws vertical lines
                pygame.draw.line(win, GRAY, (j * gap, 0), (j * gap, self.width))

        #Updates display
        pygame.display.update()

    '''
    Gets node in specified row and col
    Input:
        row
        col
    Output:
        Node at specified row col
    '''
    def get_node(self, row, col):
        return self.grid[row][col]
```
# A* Path Finding Visualizations
H functions is used to approximate distance
```python
'''
Calculates the distance from p1 and p2
using Taxi Cab Distance formula
Input:
    p1 - Point 1 (x,y) 
    p2 - Point 2 (x,y)
'''
def h(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return abs(x1-x2) + abs(y1-y2)
```
This function is algorithm for A* visualization calculates the H, G and F score for each block in open_set. Then compares the score to neighboring tiles and closes the tile with lowest g_score. If neighbor has higher g score and is not in open set it is opened and added to open set. This is continued until proper path is found.
```python
'''
A* Path Finding algorithm function
Input:
    draw - draw function
    grid - 2D array of nodes
    start - start node
    end - end node
'''
def algorithm(draw, grid, start, end):
	count = 0
	open_set = PriorityQueue()
	open_set.put((0, count, start))
	came_from = {}
	g_score = {spot: float("inf") for row in grid for spot in row}
	g_score[start] = 0
	f_score = {spot: float("inf") for row in grid for spot in row}
	f_score[start] = h(start.getPos(), end.getPos())

	open_set_hash = {start}

	while not open_set.empty():
		for event in pygame.event.get():
			if event.type == pygame.QUIT:
				pygame.quit()

		current = open_set.get()[2]
		open_set_hash.remove(current)

		if current == end:
			reconstruct_path(came_from, end, draw)
			end.make_end()
			return True

		for neighbor in current.neighbors:
			temp_g_score = g_score[current] + 1

			if temp_g_score < g_score[neighbor]:
				came_from[neighbor] = current
				g_score[neighbor] = temp_g_score
				f_score[neighbor] = temp_g_score + h(neighbor.getPos(), end.getPos())
				if neighbor not in open_set_hash:
					count += 1
					open_set.put((f_score[neighbor], count, neighbor))
					open_set_hash.add(neighbor)
					neighbor.make_open()

		draw()

		if current != start:
			current.make_closed()

	return False
```
# Driver Function
This handles set up and then handling pygame events
```python
'''
Driver Function
Input
    window - pygame display window
    width - width of grid
'''
def main(window, width):
    ROWS= 50
    board = Grid(ROWS, ROWS, width)

    start = None
    end = None

    run = True
    started = False
    while run:

        #Updates board
        board.draw_grid(window)

        #Handles Pygame events
        for event in pygame.event.get():
            #Checks if quit button is presed
            if event.type == pygame.QUIT:
                run = False

            #Prevents interaction when algorithm running
            if started:
                continue

            #Handles Left Mouse Click
            if pygame.mouse.get_pressed()[0]: 
                
                #Gets mouse position
                pos = pygame.mouse.get_pos()
                row, col = get_clicked_pos(pos, ROWS, width)
                
                #Checks if click is inbounds
                if row > 49 or col > 49:
                    continue

                #Gets node
                node = board.get_node(row,col)

                #First sets start if not set
                if not start:
                    start = node
                    start.make_start()
                #Then sets end if not set
                elif not end:
                    end = node
                    end.make_end()

                #If start and en set then makes barrier
                elif node != end and node != start:
                    node.make_barrier()

            #Handles Right Mouse Click
            elif pygame.mouse.get_pressed()[2]:
                #Gets mouse position
                pos = pygame.mouse.get_pos()
                row, col = get_clicked_pos(pos, ROWS, width)
                
                #Checks if click is inbounds
                if row > 49 or col > 49:
                    continue

                #Gets node
                node = board.get_node(row,col)

                #Resets node and handles start
                node.reset()
                if node == start:
                    start = None
                elif node == end:
                    end = None

            #Handles keyboard presses
            if event.type == pygame.KEYDOWN:

                #When space is pressed run A* algorithm
                if event.key == pygame.K_SPACE and start and end:
                    for row in board.grid:
                        for node in row:
                            node.update_neighbors(board.grid)

                    algorithm(lambda: board.draw_grid(window), board.grid, start, end)

                #When c is pressed clears the board
                if event.key == pygame.K_c:
                    start = None
                    end = None
                    board = Grid(ROWS, ROWS, width, ROWS)            

    #Quits out of pygame
    pygame.quit()
```
# Future Additions
- Add more path finding algorithms
- Add legend on GUI to increase understaindg of visualization
