# FIT5222-Assignment2
# Wechat: csprojhelp 备注: github

FIT5222-Assignment2 cs 1v1 tutor 专业辅导

FIT5222 Assignment 2 Pacman: Capture the Flag

In this assignment we will develop an AI controller for a multiplayer variant of Pacman. In this game two teams, red and blue, compete with each other to capture as many dots as possible from the opposite side of the map.

Originally developed at UC Berkeley, this setup presents us with a challenging planning domain and an opportunity to solve a fun problem. At the end of the semester we will hold a contest to find out who developed the most effective controller. Glory and a fancy certificate await the top three students!

Part 1: Installation

Piglet PDDL Solver

In this assignment you will use the piglet library to solve PDDL problems. This requires installing piglet to the python environment:

1. Go to piglet-public folder (If you use a virtual environment, activate the virtual env first. You could use your flatland virtual environment as well.)

2. git fetch

3. git checkout pddl_solver //if successful, you should find a pddl_solver.py under lib_piglet/utils/ folder

4. python setup.py install

Check the myTeam.py starter implementation to see how to use the newly installed library. The competition server will have the same piglet installed in the environment as a PDDL solver.

Update Pacman Code

Make sure you have the latest version of the Pacman CTF game environment: FIT5222 Planning and automated reasoning

1. Go to the pacman-public folder (you cloned the repo in week 1)

2. git fetch

3. git reset --hard //in case any changes you made locally

4. git pull

5. If your code is newest, you should see staffTeam.py and berkeleyTeam.py in the repo.

All codes for the game environment are now found in the pacman folder.

Part 2: Game Rules

We can characterise Pacman CTF as follows:

● Multi-agent: Two agents need to work together against an opposing team.

● Discrete: The environment is a grid maze and time is discretised into unit time steps.

● Dynamic: The environment changes as food is consumed.

● Partially observable: Each agent has a limited sensing range.

● Sequential: Past decisions affect future actions.

● Deterministic: The current state depends only on the teams and their actions.

● Offline: The world stops while an agent deliberates.

● Known: All the rules of the game are available a priori.

Environment

The game map is divided into two halves: red (left) and blue (right). When on the red side, a red

agent is a ghost. When crossing into enemy territory, the agent becomes a Pacman. Red

agents must defend the red food while trying to eat the blue food.

Scoring

When a Pacman agent eats a food dot, that dot is removed from the board and placed in a

virtual backpack. When the agent returns to their own side of the board, they automatically

"deposit" the contents of their backpack, earning one point per dot. The Red team scores

positive points, while the Blue team scores negative points.

Watch out! If an agent is caught by a ghost, before reaching their own side of the board, all

dots in their backpack will be deposited back to their original positions on the board. The agent

also returns (as a ghost) to its original starting position. No points are scored in this case. No

points are awarded to the opposing team either (for catching Pacman). FIT5222 Planning and automated reasoning

Power Capsules

A small number of power capsules are scattered throughout the maze. These can be eaten by a Pacman agent. When this happens ghosts become “scared” and are susceptible to being caught by the powered up Pacman. This ghost effect lasts for the next 40 timesteps, or until caught, whichever comes sooner.

Observations

Agents can only observe an opponent's configuration (position and direction) if they or their teammate are within 5 squares (Manhattan distance). In addition, an agent always gets a noisy distance reading for each agent on the board, which can be used to reason about unobserved opponents. The reading has an error of +/- 6 steps from the true distance. A general direction (toward the opponent) is not specified.

Winning

A game ends when one team returns all but two of the opponents' dots. Games are also limited to 300 time steps (i.e., 300 moves for each of the four agents). If this move limit is reached, whichever team has returned the most food wins. If the score is zero (i.e., tied) this is recorded as a tie game.

Part 3: Task & Student Workflow

At every time step the simulator will present you with the current state of the game board. Your task will be to reason over this current state and decide how your agents should react.

This will require you to think about the planning process in several different ways: at the high-level, where you decide what strategy your team will pursue; at the low-level, where you transform that strategy into a concrete sequence of low-level actions; and you will need to decide when to replan your agents, in response to changing circumstances.

We illustrate an example of the student workflow in the diagram below.
Your starting point for modifying this workflow is the function chooseAction in file myTeam.py. This function is called by the simulator at every time step and for each agent. The simulator provides the agent with a description of the game board. At the end of the function you must return a concrete action for the agent to execute in the next timestep.

How to decide which action the agent should take is entirely up to you. The starting code provides a basic template for the decision-making process and some simple, though not very effective, strategies. You will need to modify this code to create a winning AI.

The next sections give further details about the decision-making process. For more details regarding the Pacman CTF code, the myTeam.py implementation, and other general tips, refer to the additional document “Assignment 2 Code Documentation.pdf”!

3.1. Choose a High Level Goal

What is the high-level strategy that agents should pursue? For example, they could visit the opposing side and try to score points, or they could stay home, and defend their score. Each agent chooses its own high level goal, but that decision may be influenced by the current state of the board, the current score, and the strategy and position of the teammate agent.

Your starting point here is the getGoals function in the file myTeam.py. Given a PDDL description of the current state, this function returns positive and negative PDDL state descriptions that together describe a high-level goal. You can create multiple different goals and then choose between them based on the information in the current state. FIT5222 Planning and automated reasoning

3.2. Generate a High Level Plan

To achieve your high-level goals you will need to compute a corresponding sequence of high-level actions (i.e., a plan). Given a PDDL description of the initial (i.e., the current) states and the goal states, the function getHighLevelPlan computes such a plan. You do not need to modify this function. Just call it as necessary.

But what if the agent already has a high-level plan, computed in a previous iteration? It’s up to you to decide how to proceed. The agent could retain the existing plan or it could change, and replan anew from the current state. For example, if the pre-conditions for the next action are no longer satisfied, the agent will certainly need to generate a new high level plan. The agent may also decide to revise its current plan (and possibly current goal!) if its execution produces an unexpected result (e.g., being caught by a ghost or a powered-up Pacman).

The staff implementation checks if we can reuse existing plan:

Note: Only a limited set of predicates are included in the basic PDDL state description (see the file myTeam.pddl) and not all of them appear in the given actions. You can create a richer model, and compute more diverse plans, by (1) refining existing actions or creating new ones that use more of the available predicates; (2) enhancing the PDDL state description with additional information from the current game state (i.e., add more predicates). We provide useful programmatic facilities for this purpose. See the get_pddl_state function for examples of how to collect information from the simulator environment (gameState). FIT5222 Planning and automated reasoning

3.3. Generate a Low Level Plan

High level plans tell the agents what to do, but in broad strokes. For example, if the goal is to score points, a necessary action is to move to the opposite side of the board. But how should the agent get there? Once on the opposing side, the next high level action might be to eat a food dot. But which one? Low level planning provides answers to these questions.

The function getLowLevelPlan is your starting point here. Its inputs are a high-level action and a description of the current game state. The function returns a sequence of move actions that the agent needs to take to achieve the high-level effects of the input action.

You are free to implement this function using any method you choose. For example:

● Heuristic Search Strategy: you can develop your own custom strategies by implementing a new expander class for your own customised state-space representation.

● Learning Based Strategy: we provide a very basic example of this approach. You can make your own implementation or improve it by introducing more features to the model. Then you will train the new model to obtain a set of good weights. You should carefully consider, for each high level action, how you will reward or penalise the agent for its performance. You may need to keep adjusting your implementation, observe how your agent behaves, and try something new to overcome the drawbacks you observed.

But what if the agent already has a low-level plan, computed during a previous iteration? Again, it’s up to you to decide how to proceed. The agent could continue following the existing low-level plan or it might decide to revisit its decisions on account of new information provided by the game state. For example, if Pacman is heading towards a food dot, but observes a ghost up ahead, continuing toward the dot may be a bad idea.

The staff implementation reuse low level plans:

Remember! There are many different ways to achieve a high-level action. You may also wish to consider whether your two agents coordinate their low-level actions or act independently of one another. FIT5222 Planning and automated reasoning

3.4. Execute Moves Your low-level strategy has generated a sequence of directions for the agent to move in. The first move in this plan is given to the simulator for execution. Each of your two agents, and the two opposing agents, move according to their own instructions and the timestep is advanced by one. All of this happens after the completion of the chooseAction function.

Based on the new state of the game board, you may need to re-evaluate your goals and plans. This workflow is cyclic and continues until the game is over.

Remember: You can (and should!) compare your implementation (myTeam.py) against the existing baselines (staffTeam.py or berkeleyTeam.py). You can find these implementations in the pacman folder. The baselines are useful to check that your implementation works and to measure your progress.

Part 4: Competition Setup

We will have a competition to see who has the strongest pacman AI. You will be able to submit your implementation to a contest server where they will be run against each other.

Upon submission to the server your agent will be evaluated against a staff baseline implementation (staffTeam.py). Your agent will be evaluated for 49 games on 7 different maps. Within each game there will be a time limit and the agent that has succeeded in eating the most food during that time will win that game. To win the match with staff baseline, your team must win convincingly: 28 out of 49 games.

By winning the staffTeam.py, you will get a pass for Criteria 1 competition score (see more details in marking rubrics), have a position on the leaderboard, and your agent will be evaluated against all other participants on the leaderboard to get additional victory points.

With each opponent on the leaderboard (excluding the staff baseline implementation), your agent will be evaluated on the same set of problems:

● In the event of a convincing win (win at least 28 out of 49 games), you will get 3 victory points.

● In the event of a tie (win less than 28 games, but lose less than 28 games as well, considering tie games neither win nor lost.), you will get 0.5 victory points.

● In the event of a loss (lose 28 games or more), you will get 0 victory points.

If you win the staff baseline implementation, your marks for Criteria 1 competition will be decided by (Your total victory points / All available victory points)%, where the all available victory points is 3 × (number of all participants - 1).

You are expected to clearly win against the staff baseline implementation, otherwise you will not proceed to the next stage of competing with entries from other students. In this case your grade for Criteria 1 of the marking rubric will be a fail. FIT5222 Planning and automated reasoning

The server will keep a record of your matches against every opponent on the leaderboard. When someone have a new best implementation recorded, your score against that student will be updated as well.

More details regarding the competition setup will be provided in a supplementary document “Assignment 2 Contest Submission Instruction.pdf”.

Part 5: Report

Together with your implementation source code you will be required to submit a report that describes your approach. You will need to write a description of your strategy and give a justification for algorithmic choices. Make sure that your report is as detailed and complete as possible, including appropriate references.

When describing your model you should explain not only what it does but also analyse its strengths and weaknesses (e.g., complexity, guarantees etc). You should also discuss how your code implements that model: for example, important data structures, necessary optimisations and parameter choices. Justify your design and implementation choices with discussion and, where appropriate, with experiments.

You can ask questions about the assignment and you can discuss the merits of different design choices (e.g., on Ed). However all submitted work must be completely your own. Do not copy implementations you happen to find online. Do not ask your friends or colleagues for their implementations. You will need to work independently and take reasonable steps to make sure others’ don’t copy or misuse your work. For more information please see https://www.monash.edu/students/study-support/academic-integrity
