# Users Lab
Lab 5, CPS206, Spring 2017

## Description
This lab will leverage everything covered in the semester. You will implement a command prompt driven user database with the following patterns:

* Builder Pattern
* Observer Pattern
* Memento Pattern
* Data Access Object Pattern

Your application driver will be built around a infinite state machine. As the state changes, it will notify its observers as to the state change and provide a callback to the driver if an observer needs to make changes to the drivers state. 
 
Each observer will decide if the state change requires action by them, and if so, perform their prescribed operations while in that current state.

Provided in the repository is an enumeration of State, with all possible states.

You are required to have 3 observers. 

* Prompter
* UserDAO
* Logger

## Application Driver
Your application driver does two things, and two things only. First, it manages its state. Second, it notifies all observers a state changed. The following are the possible state paths.



<center><img src="http://imgur.com/a/Cg3kJ"></center>