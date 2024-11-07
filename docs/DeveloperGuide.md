---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# DorManagerPro Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
2. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
3. The command can communicate with the `Model` when it is executed (e.g. to delete a person). Certain types of commands (FileAccessCommand) can also communicate with the `Storage` when it is executed.<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object `Model` and `Storage`) to achieve.
4. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* stores a history of concrete commands (commands that can be undone) executed successfully by the user, which allows the user to undo commands.
* only depends on the Logic component due to the undo feature.

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### Undo feature

#### Implementation

The undo mechanism is facilitated by the abstract class `ConcreteCommand`. 
It extends `Command` and has the `undo()` method. The `undo()` method is called when the user executes the `undo` command. 
The `undo()` method reverses the effects of the command that was previously executed.
The `undo()` method is implemented in the concrete command classes, such as `AddCommand`, `DeleteCommand`, `EditCommand`, etc.

The `Model` component stores a history of executed concrete commands in a stack.
When a command is executed successfully, the command is pushed onto the stack.
When the user executes the `undo` command, the `Model` component pops the last command from the stack and calls the `undo()` method of the command.

These operations are exposed in the `Model` interface as `Model#pushToUndoStack()` and `Model#undoAddressBook()`.

Given below is an example usage scenario and how the undo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The undo stack is initialised as empty.

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command is pushed onto the undo stack.

Step 3. The user executes `add n/David …​` to add a new person. The `add` command is also pushed onto the undo stack.

<box type="info" seamless>

**Note:** If a command is not undoable or fails its execution, it will not be pushed onto the undo stack.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will pop the last command from the undo stack and call its `undo()` method. The `undo()` method of the command will then reverse the effects of the command.

<box type="info" seamless>

**Note:** If the undo stack is empty, then there are no commands to undo. The `undo` command checks if this is the case. If so, it will return an error to the user.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will not be pushed to the undo stack. Thus, the undo stack remains unchanged.

Step 6. The user executes `clear`, which is pushed to the undo stack.

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo executes:**

* **Alternative 1:** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2 (current implementation):** Individual command knows how to undo by itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### Clean feature

#### Implementation

The `clean` command extends `ConcreteCommand`. The `clean` command deletes the contacts whose `GradYear` field is earlier
than the current year, deleting contacts who have graduated from the address book.
The `clean` command is undoable.

Given below is an example usage scenario and how the `clean` command behaves at each step.

Step 1. The user executes `clean` in 2024.

<box type="info" seamless>

**Note:** The `clean` command checks if there are contacts with `GradYear` 2023 or earlier. If there are none, it will return an error message to the user.

</box>


Step 2. The `clean` command deletes all contacts with `GradYear` 2023 or earlier.


The following sequence diagram shows how a `clean` command goes through the `Logic` component:

<puml src="diagrams/CleanSequenceDiagram.puml" alt="CleanSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** There are no destroy markers (X) for `CleanCommand` and `GradYearPredicate` as they are preserved in the `undo` command stack.

</box>

The following activity diagram summarizes what happens when a user executes a `clean` command:

<puml src="diagrams/CleanActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: UI display when `clean` executes after a `find` command:**

* **Alternative 1:** Display all contacts.
    * Pros: Shows users the full result of `clean`.
    * Cons: Forgets the results of the `find` command.

* **Alternative 2 (current implementation):** Retain the search results of `find` and only display those contacts. 
    * Pros: Allow users to retain their serach results from `find`.
    * Cons: Users cannot see the full extent of `clean` until they return to the default view with `list`.

### Update Find Command

#### Implementation

* the findCommand is enhanced by new Predicates 
* RoomNumber predicates, PhonePredicate, and TagContainsKeywordsPredicate allows
the a wider range of searching based on more features

Below is a detailed process illustration using a sequential diagram:

Step 1. The user issues a `find` command followed by specific parameters, 
for example: `t/friends n/Alex r/08-0805 p/9124 6892`, searches for a profile with a 
tag of friends, a name called Alex, a roomNumber of 08-0805, and a phone call of 9124 6842
 
**Note** : These parameters can be combined in any sequence, allowing for versatile parameter configurations. 

Step 2. The parser interprets the user command and constructs corresponding predicates for the `FindCommand` object.

Step 3. The `FindCommand` get executed and updates the filteredPersonList within the model, reflecting the search 

results based on the specified criteria.

<puml src="diagrams/FindSequenceDiagram.puml" alt="FindSequenceDiagram" />

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**: University dormitory manager (teacher residents or admins at Dorm Halls)

* has a need to manage a significant number of student contacts
* prefer desktop apps over other types
* has experience and prefers using CLI apps

**Value proposition**: Provide fast and centralised access to vital resident information.

* keeps track of residents' contact details
* manages room number and roles of residents
* allows for quick input of details and querying by different conditions for dorm managers of large dorms.

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                   | I want to …​                                                 | So that I can…​                                                        |
|----------|---------------------------|--------------------------------------------------------------|------------------------------------------------------------------------|
| `* * *`  | forgetful dorm manager    | see usage instructions                                       | refer to instructions when I forget how to use the App                 |
| `* * *`  | dorm manager              | add a new contact                                            | keep track of students in my dorm                                      |
| `* * *`  | dorm manager              | delete a contact                                             | remove entries when residents leave the dorm to prevent clutter        |
| `* * *`  | dorm manager              | view contacts                                                |                                                                        |
| `* * *`  | dorm manager              | see emergency contacts of my residents                       | act quickly in the event of an emergency                               |
| `* * *`  | strict dorm manager       | know my residents' room number                               | go check on them                                                       |
| `* *`    | bustling dorm manager     | find a contact by their details (eg. name, room)             | locate contacts without having to go through the entire list           |
| `* *`    | dorm manager              | edit a contact                                               | update contact details                                                 |
| `* *`    | clumsy dorm manager       | undo my actions                                              | restore information if I accidentally delete them                      |
| `* *`    | dorm manager              | give roles (responsibilities) to my residents                | foster communal living                                                 |
| `* *`    | forgetful dorm manager    | keep track of the roles of my residents (eg. RA, CCA leader) | know their responsibilities                                            |
| `* *`    | dorm manager              | find residents with certain roles (eg. RA, CCA leader)       | view and contact them as a group                                       |
| `* *`    | impatient dorm manager    | load / save all resident details to a file                   | avoid typing in each resident's details                                |
| `*`      | dorm manager              | sort contacts by name, room number                           | locate a contact easily                                                |
| `*`      | forgetful dorm manager    | search by partial matches                                    | find contacts without memorising their full name                       |
| `*`      | neat dorm manager         | group residents by block, floor, cluster, year               | keep contacts organised in specific groups                             |
| `*`      | dorm manager              | filter search results by roles or groups                     | find contacts in specific groups quickly                               |
| `*`      | dorm manager              | view a summary of resident groups                            | get an overview of the dorm population                                 |
| `*`      | dorm manager              | keep track of demerit points of my residents                 | evict them when needed                                                 |
| `*`      | anxious dorm manager      | know the phone, email and home address of my residents       | contact residents through multiple channels in case they don't respond |
| `*`      | thoughtful dorm manager   | know the major of my residents                               | provide care and support during stressful periods                      |
| `*`      | wholesome dorm manager    | know the clubs of my residents                               | support their arts showcases / sports competitions                     |
| `*`      | enthusiastic dorm manager | know the food preferences of my residents                    | prepare welfare packs for them                                         |
| `*`      | dorm manager              | know the medical conditions of my residents                  | provide timely medical assistance                                      |
| `*`      | dorm manager              | know the nationality of my residents                         | better respect their culture                                           |
| `*`      | thoughtful dorm manager   | add a small description of each resident                     | note the quirks and interests of each resident                         |
| `*`      | dorm manager              | keep the commands short and powerful                         | use it effectively with CLI experience                                 |


### Use cases

(For all use cases below, the **System** is `DorManagerPro`, and the **Actor** is the `user` who refers to university dormitory managers unless specified otherwise.)

**Use Case: UC01 - Add a profile**

**MSS:**

1. User requests to add a specific profile, specifying name and contact number.
2. DorManagerPro adds the profile.

   Use case ends.

**Extensions:**

* 1a. DorManagerPro detects an error in the command format.
    * 1a1. DorManagerPro requests for the correct command format.
    * 1a2. User enters command again.

  Steps 1a1-1a2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1c. DorManagerPro detects that the specified profile already exists.
    * 1c1. DorManagerPro informs the user and asks for another profile to add.
    * 1c2. User specifies another profile to add.

  Steps 1c1-1c2 are repeated until a valid profile is indicated.
  Use case resumes from step 2.

* 1d. DorManagerPro detects invalid parameters specified by user.
    * 1d1. DorManagerPro requests for valid parameters.
    * 1d2. User re-supplies parameters.

  Steps 1d1-1d2 are repeated until the parameters are valid.
  Use case resumes from step 2.

* *a. At any time, User chooses to stop adding a profile.
  Use case ends.

**Use Case: UC02 - Add room number to profile**

**Precondition: There is at least one profile added into DorManagerPro**

**MSS:**

1. User requests to add room number information to a specific profile.
2. DorManagerPro updates the profile to include the room number.

   Use case ends.

**Extensions:**

* 1a. DorManagerPro detects an error in the command format.
    * 1a1. DorManagerPro requests for the correct command format.
    * 1a2. User enters command again.

  Steps 1a1-1a2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1b. DorManagerPro cannot find the specified profile to update.
    * 1b1. DorManagerPro requests for a profile that exists to update.
    * 1b2. User specifies profile again.

  Steps 1b1-1b2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1c. DorManagerPro detects that the room capacity is already full.
    * 1c1. DorManagerPro requests for a room number that is not already occupied.
    * 1c2. User specifies room number again.

  Steps 1c1-1c2 are repeated until a valid room number is provided.
  Use case resumes from step 2.

* 1d. DorManagerPro detects invalid parameters specified by user.
    * 1d1. DorManagerPro requests for valid parameters.
    * 1d2. User re-supplies parameters.

  Steps 1d1-1d2 are repeated until the parameters are valid.
  Use case resumes from step 2.

* *a. At any time, User chooses to stop adding a room number.
  Use case ends.

**Use Case: UC03 - Add emergency contact to profile**

**Precondition: There is at least one profile added into DorManagerPro**

**MSS:**

1. User requests to add emergency contact information to a specific profile.
2. DorManagerPro updates the profile to include the emergency contact.

   Use case ends.

**Extensions:**

* 1a. DorManagerPro detects an error in the command format.
    * 1a1. DorManagerPro requests for the correct command format.
    * 1a2. User enters command again.

  Steps 1a1-1a2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1b. DorManagerPro cannot find the specified profile to update.
    * 1b1. DorManagerPro requests for a profile that exists to update.
    * 1b2. User specifies profile again.

  Steps 1b1-1b2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1c. DorManagerPro detects invalid parameters specified by user.
    * 1c1. DorManagerPro requests for valid parameters.
    * 1c2. User re-supplies parameters.

  Steps 1c1-1c2 are repeated until the parameters are valid.
  Use case resumes from step 2.

* *a. At any time, User chooses to stop adding an emergency contact.
  Use case ends.

**Use Case: UC04 - View profiles**

**Precondition: There is at least one profile added into DorManagerPro**

**MSS:**

1. User requests to view profiles.
2. User requests to view certain profiles based on the profiles features (tags, roomNumber, number, name)
3. DorManagerPro displays all profiles with all attached information.

   Use case ends.

**Extensions:**

* 1a. DorManagerPro detects an error in the command format.
    * 1a1. DorManagerPro requests for the correct command format.
    * 1a2. User enters command again.

  Steps 1a1-1a2 are repeated until the command is correct.
  Use case resumes from step 2.

* *a. At any time, User chooses to stop viewing a profile.
  Use case ends.

**Use Case: UC05 - Delete a profile**

**Precondition: There is at least one profile added into DorManagerPro**

**MSS:**

1. User requests to delete a specific profile.
2. DorManagerPro asks if user to confirm they want to delete the profile.
3. User confirms.
4. DorManagerPro deletes the profile.

   Use case ends.

**Extensions:**

* 1a. DorManagerPro detects an error in the command format.
    * 1a1. DorManagerPro requests for the correct command format.
    * 1a2. User enters command again.

  Steps 1a1-1a2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1b. DorManagerPro cannot find the specified profile to delete.
    * 1b1. DorManagerPro requests for a profile that exists to delete.
    * 1b2. User specifies profile again.

  Steps 1b1-1b2 are repeated until the command is correct.
  Use case resumes from step 2.

* 1c. DorManagerPro detects invalid parameters specified by user.
    * 1c1. DorManagerPro requests for valid parameters.
    * 1c2. User re-supplies parameters.

  Steps 1c1-1c2 are repeated until the parameters are valid.
  Use case resumes from step 2.

* 3a. DorManagerPro detects an error in the confirmation message sent by the User
    * 3a1. DorManagerPro asks the user for confirmation to delete the profile again.
    * 3a2. User confirms again.

  Steps 3a1-3a2 are repeated until the confirmation is correct.
  Use case resumes from step 4.

* 3b. User expresses they do not want to delete the profile after all.
    * 3b1. DorManagerPro acknowledges the rejection.

  Use case ends.

* *a. At any time, User chooses to stop deleting a profile.
  Use case ends.

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2. Should work on Mac as long as javafx and java '17' both installed.
3. Should be able to hold up to 1000 persons without noticeable sluggishness in performance for typical usage.
4. A user with above average typing speed for regular English text (i.e., not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
5. Should take less than 1000 milliseconds to finish every command operation.
6. Should take less 1 gigabyte of storage.
7. Should have an upper limit of 10000 contacts on the list.
8. Should take no more than one contact for each person.
9. Should take no more than 1 emergency contact for each person.

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Dorm**: A university or college hall of residence / hotel for students and teachers
* **Dorm resident**: Student and / or teacher currently staying in a dorm
* **Dorm manager**: User of Dormanager Pro that has to keep track of the residents in their dorm
* **Profile**: Collection of information related to a resident that serves as a block of interrelated data in Dormanger Pro. Consists of name, contact number, room number, and emergency contact.
* **Emergency contact**: Person to contact when the resident related to said contact gets into an emergency (injury, immigration related issues etc.). Consists of a name and contact number.
* **Dorm room**: Rooms of the dorm where residents stay in. Has a room number and upper limit of

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
