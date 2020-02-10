Gamification Engine
====================

Gamification Engine is the tool for implementing the user community 
engagement mechanisms within mobile and web applications. Gamification Engine
allows for capturing the game model concepts, such as points, levels,
badges, challenges, leaderboars, and define the rules for their assignments
in correspondence to the actions performed by the users.

Game Model
------------

Each game model defined with the gamification engine consists of a set of elements, such as game actions, game concepts, rules, and tasks. During the game execution, the game state is being populated and maintained. The game state is represented with the state of its individual players, namely user state, such as current user points, badges earned, etc.

Game
~~~~~~~

The game is a container of other element and is defined with the following set of attributes:

- id: automatically assigned from system. It uniquely identifies the game within the engine instance. The game ID is important for relating the actions to the game state during the execution.
- name: the game name.
- owner: the user owner of the game. This is the ID of the game manager, and is relevant for the multitenant execution environment.
- game model: the set of concept instances used by this specific game (different types of points, badges, rules, classifications, etc).

Game Action
~~~~~~~~~~~~~~~

The game actions represent the game-specific events that triggers changes in the game state. The game actions may be external and internal. The external actions correspond to the user actions (when triggered, e.g., through a mobile app), while the internal ones correspond to the system triggers (e.g., timers to recalculate periodic user classifications). Each action may carry some data with it.

Game Concepts
~~~~~~~~~~~~~~~~~~

The game concepts represent elements like points and badges. Each game may define its own set of points (green points, community points, etc.), as well as badge collections. Participating to the game (through the actions), the user can earn or lose the points (i.e., her state changes).

Game Rules
~~~~~~~~~~~~~~~

A rule defines how the state of the users changes in reaction to some actions. Rules follow the “event-condition-action” paradigm, where event corresponds to the game action triggering the rule, condition represent the situation around the current user state and the action payload, and the action defines how the state should evolve (e.g., points that are added, badge that is earned, and the notification is sent).

Currently, the Drools Rule (http://www.drools.org/) language is used for specifying the game rules.

Game Tasks
~~~~~~~~~~~~

The task defines a periodically scheduled internal actions and their characteristics. A typical example of the game task is the classification task, where for some specific type of points the activity is triggered on a fixed schedule (e.g., daily) to incentivate best users.


Gamification Engine Tools
----------------------------

The functionality of the Gamification Engine is made available via two key modules:

- Gamification Engine API: allows for interacting with the engine programmatically, e.g., to get information about the lidearboard, the user game status, triggering actions, readling notifications, etc.
- Gamification Engine Management API: the set of API interfaces for creating and managing game rules, concepts, and definitions.
- Gaimification Engine Console: the administration UI for accessing the management functionality of the engine
- Gamification Status Console: the UI for accessing and visualizing the current state of  the game, the users, and for performing some of the operations directly on the game data.

The detailed information about the Gamificiation Engine setup, the model, and its usage 
may be found here: https://github.com/smartcommunitylab/smartcampus.gamification/wiki/.

