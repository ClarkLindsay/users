# Users Lab
Lab 5, CPS206, Spring 2017

## Description
This lab will leverage everything covered in the semester. You will implement a command prompt driven user database with the following patterns:

* Builder Pattern
* Observer Pattern
* Memento Pattern
* Data Access Object Pattern

Your application driver will be built around a infinite state machine. As the state changes, it will notify its observers as to the state change and provide a callback mechanism to the driver if an observer needs to make changes to the 
drivers state. 
 
Each observer will decide if the state change requires action by them, and if so, perform their prescribed operations while in that current state.

Provided in the repository is an enumeration of State, with all possible states.

You are required to have 3 observers. 

* Prompter
* UserDAO
* Logger


Here are the interfaces required to complete this lab.
<center><img src="http://i.imgur.com/7UdhoI1.jpg" height=400></center>

## Application Driver
<center><img src="http://i.imgur.com/NXCounx.jpg" height=400></center>

Your application driver does two things, and two things only. First, it manages its state. Second, it notifies all observers a state changed. The following are the possible state paths.


<center><img src="http://i.imgur.com/FP9NASY.jpg" height=400></center>


To use a state machine, you will begin at `Start` as your application begins. This is arbitrary because you will never go back to `Start`. You will need to however go into the first notifiable state, `STATE_CHANGE_REQUEST`.
 
### STATE_CHANGE_REQUEST
`STATE_CHANGE_REQUEST` has 7 possible state changes. We will cover them all here.
If you are in this state, the `Prompter` will prompt will all possible state changes, eg:

```java
Please select an action:
1. Create a user
2. Find a user
3. List all users
4. Update a user
5. Delete a user
6. Show history 
7. Quit
```
#### Create a user
If the create a user option is selected, the application driver will simply change state to `CREATE_USER` and notify all observers with 

```java
observer.observe(currentState, stateData, this);
```
When the observer receives this method call, they will check to see if that is a state they care about, use the data provided (`null` in this case), and if needed, use the callback mechanism to update the next state. Please realize, only **one** observer should update the next state. I am purposely leaving this very susceptible to breaking, meaning if you call use the callback mechanism and you are not supposed to, you could cycle through the states spuriously until you end up back at the `STATE_CHANGE_REQUEST`. I am leaving it up to you on how to prevent this from happening. 

**The only allowed state after `CREATE_USER` is `USER_CREATED`.** 

#### Find a user
If the find a user option is selected, the application driver will simply change state to `FIND_USER` and notify all observers,

**The only allowed state after `FIND_USER` is `USER_FOUND`.** 

#### List all users
If the find all users option is selected, the application driver will simply change state to `FIND_ALL_USERS` and notify all observers,

**The only allowed state after `FIND_ALL_USERS` is `FOUND_ALL_USERS`.** 

#### Update a user
If the update user option is selected, the application driver will simply change state to `UPDATE_USER` and notify all observers,

**The only allowed state after `UPDATE_USER` is `USER_UPDATED`.** 

#### Delete a user
If the delete user option is selected, the application driver will simply change state to `DELETE_USER` and notify all observers,

**The only allowed state after `DELETE_USER` is `USER_DELETED`.** 

#### Show History
If the show history option is selected, the application driver will simply change state to `HISTORY_REQUSTED` and notify all observers,

**The only allowed state after `HISTORY_REQUESTED` is `HISTORY_FOUND`.** 


### Notifying the Observers
There is no need to sequentially notify the observers. Because of so, you can simply use the following in your 
`notifyObservers` method:

```java
observers.parallelStream().forEach((observer) -> observer.observe(currentState, stateData, this));
```

### Application Conclusion
Hopefully you see that our `Application` is simply providing a mechanism for our observers to act. We should be able to add any given observer as our application scales without the need for really modifying the application driver. This is our observer pattern.
 
 
 
 
 
## UserDAO
The User Data Access Object (UserDAO) is part of the Observer Pattern and integrates the Builder Pattern. The following is the UML for this portion of the lab.

<center><img src="http://i.imgur.com/YpCw3lI.jpg" height=400></center>

Through the `Observable` interface, the `UserDAO` will listen for the following state changes:

* `CREATE_USER`
* `FIND_ALL_USERS`
* `FIND_USER`
* `UPDATE_USER`
* `DELETE_USER`

### Event `CREATE_USER`
The `UserDAO` on this event will use the `userBuilder` to create a new user. The `UserBuilder` class will via the `HumanInteface` class polymorphed from `Prompter`, ask the user of the application for a name and then an email. When called upon to build, the `UserBuilder` will generate an id, returning a `User` object to the `UserDAO`. The `UserDAO` will then put this new user object into its collection keyed by the `User::id`.

Once completed, it will then use the callback mechanism to access the applications `setStateData` and `nextState` 
methods.

### Event `FIND_ALL_USERS`
The `UserDAO` on this event will simply set the state data to a **clone** of the users data and call upon next state.

### Event `FIND_USER`
To find a user, the `UserDAO` will use the `UserBuilder:getID` method, and once the `User` object is built, use that 
objects id to get the `User` from its collection. You will then migrate the data in the `User` object into the 
returned object by the builder, or simply clone the collections `User`.

Once completed, it will then use the callback mechanism to access the applications `setStateData` and `nextState` 
methods.

### Event `UPDATE_USER`
To update a user, you will simply use the same builder concept as finding a user to get a user id. Once you have that
 id, simply ask for name and email and then build. Once built, you will then replace the collections value at the key 
 with the new 
`User` object.

Once completed, it will then use the callback mechanism to access the applications `setStateData` and `nextState` 
methods.

### Event `DELETE_USER`
To delete a user, reuse the techniques in the find user event to get the id of the user to delete. Once that user 
object is given to you, you will then find that value in the collection of users, cache it, then delete the 
collections key/value pair. 

Once completed, it will then use the callback mechanism to access the applications `setStateData` and `nextState` 
methods.


### UserDAO Conclusion
The `UserDAO` is by far the most involved. Plan on spending time with it. The great thing about the builder pattern 
is that for each event, you can and should, redo the builder, eg:

```java
// example to remove a user
User user = UserBuilder.newBuilder().getID().build();
this.users.remove(user.getId());
```

```java
// example to find a user
User user = UserBuilder.newBuilder().getID().build();
user = this.users.get(user.getId());
```

```java
// example to create a user
User user = UserBuidler.newBuilder().addName().addEmail().build();
```

```java
// example to update a user
User user = UserBuilder.newBuilder().getID().addName().addEmail().build();
```

## Prompter 
<center><img src="http://i.imgur.com/cbR8zpe.jpg" height=400></center>

The `Prompter` provides two main features, it acts as the tool that will manage event change notifications to the 
user of your application, and it provides a `HumanInterface` mechanism. Realize `HumanInterface` is not an interface 
as in the java interface, but is simply an interface for command line prompts in this lab.

## Events it will act upon
 
* 	`STATE_CHANGE_REQUEST`
*  	`USER_CREATED`
*  	`USER_FOUND`
*  	`ALL_USERS_FOUND`
*  	`USER_UPDATED`
*   `USER_DELETED`
*   `HISTORY_FOUND`

### Event `STATE_CHANGE_REQUEST`
This event will cause the `Prompter` to print the menu options:

```java
Please select an action:
1. Create a user
2. Find a user
3. List all users
4. Update a user
5. Delete a user
6. Show history 
7. Quit
```

Upon selection, it will call upon the application to change its next change state. This change is different than all 
the others. Every other state change only has one option. This one has many. Being so, you must call `setNextState` 
with the correct state requested by the application user. 

#### Event `USER_CREATED`
This event occurs after the `CREATE_USER` event and will take the passed data and print out a logical message with 
the state data. Upon finishing that, it will then change the application state to the next in the sequence.

#### Event `USER_FOUND`
This event occurs after the `FIND_USER` event and will take the passed data and print out a logical message with 
the state data. Upon finishing that, it will then change the application state to the next in the sequence.

#### Event `ALL_USERS_FOUND`
This event occurs after the `FIND_ALL_USERS` event and will take the passed data and print out a logical message with 
the state data. Upon finishing that, it will then change the application state to the next in the sequence. The 
caveat here, is that we do not wish to use the `toString()` method to print out full user data, we only wish to print
 out the listing information. So you will use the collection data passed, iterate each user, and call `toListing()` 
 to the get string to print out.
 
#### Event `USER_UPDATED`
This event occurs after the `UPDATE_USER` event and will take the passed data and print out a logical message with 
the state data. Upon finishing that, it will then change the application state to the next in the sequence.

#### Event `USER_DELETED`
This event occurs after the `DELETE_USER` event and will take the passed data and print out a logical message with 
the state data. Upon finishing that, it will then change the application state to the next in the sequence.

#### Event `HISTORY_FOUND`
This event occurs after the `HISTORY_REQUESTED` event and will take the passed data and print out a logical message 
with the state data. Upon finishing that, it will then change the application state to the next in the sequence.

### As the HumanInterface
The `HumanInterface` simply provides `ask` and `tell` methods. Classes that need to interact with the applications 
user will use the `Prompter` class, polymorphed into `HumanInterface` to interact with the user.

### Prompter Conclusion
Hopefully you see a lot of reusable code here. Use that hint to your advantage.



## Logger
<center><img src="http://i.imgur.com/zZr7Pac.jpg" height=400></center>

The `Logger` is a memento pattern to record the events of the application user. Each state change of meaning should be logged. It should simply record the state change as a string. 

The events of "meaning" are:
* USER_CREATED
* USER_FOUND
* ALL_USERS_FOUND
* USER_UPDATED
* USER_DELETED

This means if the event is `CREATE_USER`, the recorded event should be something like `At [time], request to create a
 user`.
 
The only state change that it acts upon that could change the state of the application is `HISTORY_REQUESTED`. In 
that event, the `Logger` gets the events as a **clone** from the `CareTaker`. It then like all other observers sets 
the state data and tells the application to make a state change.


## Grading
As with all take home labs, this will be graded as pass/fail. It is worth 25 points.

## Extra credit
As will all take home labs, extra credit is awarded for writing unit tests. You do not need to write tests for your 
driving application, but should for each other instantiable class in the lab.

## Due Date
This lab has a due date. It will be due the day before the last *instructional* day of the course. Because of so, extra 
credit will be awarded with working code on that day. Please note that since this is due the last day of class, you will not be able to turn it in any later after the last day of instruction. You will have to complete this lab either in 
teams or on your own, and have it fully functioning by the end of the semester if you wish to have a shot at the 
extra credit. 