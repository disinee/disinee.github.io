---
layout: post
title:  Hashiwokakero
date: 2024-03-04
category: [Projects]
---

Hashiwokakero is a logic puzzle game where players are presented with a grid circle, each containing a number. As this was a constraint satisfaction problem, our team employed a backtracking search algorithm on a 2D array as the data structure. Recursion on this 2D array enabled us to deploy forward checking to detect failure (unsatisfied constraints). 

<!--more-->

The program begins by scanning the map given from standard input via the scan_map() function. It scans all the islands into a dictionary where the keys are the island number and the value is a class type Island which holds all the relevant information of that particular island. 

Then all the neighbours of each island is initialized. It goes through each row and initializes left and right neighbours of each island, and then goes through each column to initialize up and down neighbours of each island. All the information regarding each island's neighbours are stored in the Island class that is stored within each value of island_dict. A dictionary of bridges where the keys are in the form "island number i, island number j" and the value is a class type Bridge which holds all the relevant information of that particular bridge. This dictionary is called bridges, and is returned in scan_neighbours() to form the following 2D matrix. Then the 2D matrix of bridges is initialized with size len(island_dict) x len(island_dict), namely bridge_matrix. bridge_matrix[i] [j] represents the planks between island number i and island number j. All values will be initialized to -1 unless there is no possibility for a bridge between island i and island j, in which case the value will be 0.

The bulk of program is comprised within solve_recursively(). This function employs recursion to test out possible solutions and unwinds or "backtracks" when a constraint hasn't been satisfied. This functions iterates through each value within bridge_matrix and uses no_crossover() to check that there are no vertical crossovers between horizontal neighbours and no horizontal crossovers between vertical neighbours, before actually adding a bridge. If there exists a potential bridge between 2 islands, trial different values for the number of planks on that bridge (0, 1, 2 or 3). Repeat this until you reach the end of each row in bridge_matrix and then check if the sum of the row adds up to the number of bridges required to be connected to the island corresponding to that row number.

Whilst bridge_matrix, is getting updated, so is the dictionary bridges which is used to print out the solution.
The base cases for this function includes:
 - If we complete the entire traversal of bridge_matrix, implying we have successfully solved the puzzle, return True
 - If we reach the end of a row, and:
 - If it adds up, then the constraint of that island is satisfied and we can recurse into the next row at column 0 to repeat the same process!
- If not, return False. The function will check if False is returned and if so, the function resets the values in bridge_matrix to -1 and the planks to 0 causing the function to backtrack to try out a different combination of planks in that row of bridge_matrix, until it finally get the correct row sum.
- If bridge_matrix [row] [col] is not -1, recurse into the next column on the same row.

```Python
#solve_recursively function employs recursion to test out possible solutions and unwinds or
# "backtracks" when a constraint hasn't been satisfied.
#
# Parameters:
#     row: Integer
#     col: Integer
#     bridge_matrix: [[Integer]]
#     bridges: {string: Bridge}
#     island_dict: {string: Island}
# Return:
#     Boolean
def solve_recursively(row, col, bridge_matrix, bridges, island_dict):
	#-------------Base case-----------------
	# When at the end of the matrix
	if row == len(bridge_matrix[0]):
		return True

	# When at the end of the row
	elif col == len(bridge_matrix[0]):
		sum = 0
    	for i in range(0,len(bridge_matrix[0])):
     		if bridge_matrix[row][i] != -1:
                  sum = bridge_matrix[row][i] + sum
        _island = island_dict[f"{row}"]
        # CONSTRAINT CHECK: Checks sum
		if sum == _island.initial:
     		return solve_recursively(row+1,0, bridge_matrix, bridges, island_dict)
     	else:
			return False
	# ----------------Recursion--------------
	# Checks if there is a value already assigned
	if bridge_matrix[row][col] != -1:
		return (solve_recursively(row,col+1, bridge_matrix, bridges, island_dict))
	# CONSTRAINT: Checks for crossover bridges
	if (no_crossover(bridges,island_dict[f"{row}"],island_dict[f"{col}"]) == False):
		return solve_recursively(row,col+1, bridge_matrix, bridges, island_dict)
	# Find the minimum bridge number possible give the remaining number of bridges on 			the corresponding islands.
	start = min(min(island_dict[f"{row}"].remaining_bridges, 3),island_dict[f"				{col}"].remaining_bridges)
	for i in range(start,-1,-1):
		# Assign value to call in matrix
		bridge_matrix[row][col] = i
		bridge_matrix[col][row] = i
		# Update bridges
		if f"{row},{col}" in bridges:
			bridges[f"{row},{col}"].add_plank(i)
		else:
			bridges[f"{col},{row}"].add_plank(i)
		# Update the remaining amount of bridges for the corresponing islands
		island_dict[f"{row}"].set_remaining_bridges(island_dict[f"								{row}"].remaining_bridges - i)
		island_dict[f"{col}"].set_remaining_bridges(island_dict[f"								{col}"].remaining_bridges - i)

		# recursively move to the next cell in the row.
		if (solve_recursively(row,col+1, bridge_matrix, bridges, island_dict)) :
			return True
		island_dict[f"{row}"].set_remaining_bridges(island_dict[f"								{row}"].remaining_bridges + i)
		island_dict[f"{col}"].set_remaining_bridges(island_dict[f"								{col}"].remaining_bridges + i)
	#------------------Reset------------------------------
	# Reset the matrix for the cell if nothing was found
	bridge_matrix[row][col] = -1
	bridge_matrix[col][row] = -1

	# Reset the bridge to 0 (meaning no bridge exists)
	if f"{row},{col}" in bridges:
		bridges[f"{row},{col}"].add_plank(0)
	else:
		bridges[f"{col},{row}"].add_plank(0)
	# Return false to backtrack
	return False
```

**Design Decisions:**
As this was a constraint satisfaction problem, we employed a backtracking search algorithm on a 2D array as the data structure. Recursion on this 2D array enabled us to deploy forward checking to detect failure (unsatisfied constraints). And if falure was detected, the search was terminated and it utilised recursion to backtrack and "fix" the mistake by applying a different combination of planks. 

When determining the number of planks to apply on a bridge, we decided to use the minimum between the "island's remaining_bridges" and "3" to save computational time as opposed to checking every single value of (0, 1, 2, 3) for every bridge as part of our herustics. We also decided to traverse in reverse number of planks - from  #highest to lowest (3, 2, 1, 0) as opposed to (0, 1, 2, 3) to make the program more efficient as there is greater chance of higher number of planks to be used for islands with higher values of bridges required to be connected to. We simultaneously updated the values of our Island and Bridge objects to keep track of remaining legal values for unassigend variables. This allows us to prune off and backtrack quicker. Furthermore, the Island and Bridge classes were stored in dictionary to O(1) lookup.
