# Development Log

<!-- ## Table of Contents -->

 <!-- - [Botcorp API Development Log](#botcorp-api-development-log) -->
 - [0.1.1 Specialized employees and talent acquisition](#011---specialized-employees-and-talent-acquisition)
 - [0.1.0 The shared documents drive](#010---the-shared-documents-drive)
 - [0.0.6 Rudimentary chat viewer GUI](#006---rudimentary-chat-viewer-gui)
 - [0.0.5 Saving and loading the corp's state](#005---saving-and-loading-the-corp-s-state)
 - [0.0.4 Implementation of workday structure and first trials](#004---implementation-of-workday-structure-and-first-trials)
 - [0.0.3 Group meetings](#003---group-meetings)
 - [0.0.2 Function calling implementation](#002---function-calling-implementation)
 - [0.0.1 Procedurally generated system message](#001---procedurally-generated-system-message)
 - [0.0.0 A basic conversation between founder and co-founder](#000---a-basic-conversation-between-founder-and-co-founder)

----

<!-- # Botcorp API Development Log -->

<!-- This is a granular development log of the Botcorp API from November 2023 to present, in reverse-chronological order. -->

## 0.1.1 - Specialized employees and talent acquisition

Up until this point, the company's structure had been defined in code, and all employees were hired by the developer. The next step was to instantiate the corp with the co-founder as the only employee, and allow the company to grow organically. This involved the implementation of the `post_job_opening` tool, as well as the new `Manager` subclass of `Employee`, which has all the same traits as an `Employee`, but with the additional ability to hire new employees as needed. The co-founder was made to be a manager at the outset, and a new company was created for testing this functionality: "The Artistic Meteorologist," whose mission is "To build a website where the user can input their location and see an AI-generated image of the current weather in that location." This gave the co-founder (whose name was randomly chosen to be Robert Wise) plenty of context with which to assemble a team.

<!-- Here is the first interaction between Doug and Robert: -->

## 0.1.0 - The shared documents drive

It was clear from the actions of the employees (or non-actions) that they needed a way to write and share documents with each other. A trivial implementation of this was to simply refactor the existing system of sticky notes to a shared document drive. This involved creating a static class, `Documents`, in which all documents were stored, and showing all employees every document at the bottom of their system message. Sticky notes, on the other hand, had only ever been visible to their authors. The variable names and system message were rewritten to reflect this new system, and the employees began writing planning documents after every meeting, publishing documents to the drive in accordance with their assigned tasks, and iterating on others' documents to add their own contributions.

At this time the `Participant` class was renamed to `Agent` and references to `functions`, in the context of API calls, were renamed to `tools`.

## 0.0.6 - Rudimentary chat viewer GUI

In order to show off the results, a basic GUI was created using PyQt5. This GUI ran as its own program, and could use the `corp` module to create corps or open saved corps. Eventual chat functionality was planned, allowing the whole program to be run from this GUI, but this first iteration served mainly as a viewer for past chats. A list of employees was shown in the left margin of the window, and upon selection, that employee's chat logs would become visible in the main area of the window as blue/green colored chat bubbles, with the selected employee's messages on the right and their incoming messages on the left. After initial development as a sub-module of the Botcorp package, it was refactored to its own package with `botcorp` as a package dependency.

This concluded the first phase of this project's development in which the framework was built, necessary areas of improvement identified, and a basic GUI was created.

## 0.0.5 - Saving and loading the corp's state

In the course of testing the workday, the need arose to save the state of the corp periodically so that each iteration cycle need not involve remaking every API call since the beginning of the corp's inception. For this, we implemented a JSON serialization of the corp and its directory of employees, being careful to convert objects into a set of serializable parameters that could be passed to their constructor to remake them as they were. In order to improve readability, we switched to indented "pretty" JSON files, and gave each employee their own JSON file, saving the relative path to these files in the main corp JSON file. We also began tracking the human's message history for the upcoming PyQt5 chat viewer GUI.

## 0.0.4 - Implementation of workday structure and first trials

At this point, the vision for the program's execution structure had become clear:

1. Morning planning meeting between founder and co-founder.
2. Recursive planning meetings between managers and their subordinates, starting at the top of the organizational tree structure and going down.
3. A productivity phase where employees would be given several turns to complete their assigned tasks.
4. Recursive progress summarization meetings between subordinates and their managers, starting at the bottom of the organizational tree structure and going up.
5. End-of-day progress summarization meeting between co-founder and founder, possibly functioning as the next day's morning meeting.

To test the effectiveness of the then-fully-implemented Botcorp workday structure at breaking up complex tasks, a company was created called "Sunshine News Digest" with the mission "To deliver an informative and entertaining summary of the news every day." The corporate structure was hardcoded as follows:

- **Doug Graham**, *Founder*
  - **Beatrice Roesler**, *Co-founder*
    - **Natalie Leung**, *In-house Legal Counsel*
      - **Kathy Brown**, *Paralegal*
    - **Virgil Allen**, *Chief Financial Officer*
    - **Judy Wilt**, *Chief Technical Officer*
    - **Doris Kipping**, *Art Director*
    - **Patricia Mchattie**, *Editor in Chief*

In the initial conversation with Beatrice, the goal communicated was to write a handful of articles about the aquisition of Twitter by Elon Musk from various perspectives, including legal, financial, and ethical, and have them ready for publication by the end of the day. The team excelled at task delegation, but failed to actually get any work done. More troublingly, at the EOD meetings, they lied and said that the articles were indeed written, and they would email them to their manager after the meeting's conclusion (something they definitively cannot do). Upon being given two more days to work, and being pressed for answers about the articles' non-completion, Beatrice wordlessly left the EOD meeting.

## 0.0.3 - Group meetings

The `Employee` class was given `manager` and `subordinates` attributes, and the co-founder was given a hardcoded set of subordinates, referred to internally as the "officers". The `Meeting` class was modified to support more than two participants, and a new meeting was created between the co-founder and the officers. At first, the meeting would proceed by simply "going around the room" and prompting each participant in turn, a troubling paradigm quickly emerged. In a group meeting, a participant would often address another participant by name, but by the time it was that participant's turn, the conversational context had shifted to reflect whatever the intermediate participants had said. To solve this, we implemented a "talking stick" which could be passed to any participant with the `yield_talking_stick` function, and participants without the talking stick could only call the `raise_hand` function, which would let others know they wanted the talking stick to be passed to them. However, participants would often fail to yield the talking stick for one or more rounds, leading to a lot of wasted time. Instead, we made the talking stick always move from one participant to the next, unless the `send_message` function call was accompanied by the optional `yield_to` parameter. This led to much more quick and efficient conversations.

## 0.0.2 - Function calling implementation

A `Parse` module was created that contains the `parse` function, which accepts a `response` parameter containing the response from the API, a `responder` parameter referencing the `Bot` that generated the response, and a dictionary of available functions by name. The logic to parse a response and call the functions was non-trivial, and required extensive debugging and error handling. The result was a new system of communication between founder and co-founder which relied on meeting messages being passed to the `send_message` function, rather than contained within the `content` field of the API response. The system message was updated to convey this, and the co-founder diligently obeyed. Next, the `invite_to_meeting` function was created, and the `created_spoofed_meeting` method was implemented, which created a series of messages mimicking a meeting invitation and subsequent acceptance of that invitation. Finally, the `leave_meeting` and `write_sticky_note` functions were also implemented, and any written sticky notes were included at the bottom of the system message, allowing the employee to remember information beyond the limit of their context window.

## 0.0.1 - Procedurally generated system message

Next, we created the `Employee` subclass of `Bot`, which allows for a `job_title` and `job_description`, and created the system message procedurally from this information, as well as the company's `name` and `mission`. We also gave every participant `gender`, `first_name`, and `last_name` attributes which were randomized. After implementing the opening paragraphs of the system message, referred to internally as the "training manual," the co-founder immediately assumed the identity they were given. The rest of the system message template took a fair bit of time to write, and is extensive in its enumeration of the various functions that would be available for the employee to execute. Each section was defined as its own string, accompanied by JSON-style dictionaries which defined any related functions in a format understandable to the API, though function calling was not implemented yet. A `Prompt` class was also created which contains a list of paragraphs as well as a list of JSON-style function definitions, and supported concatenation into longer prompts with the addition operator. The `Bot` class was modified to contain a list of these JSON-style function definitions, and pass them to the API. The co-founder immediately started trying to call these functions, which hastened the urgency of the implementation of function-calling.

## 0.0.0 - A basic conversation between founder and co-founder

We used the `wigli-cli` code from early 2023 as a jumping-off point, allowing us to instantiate a instance of the `Bot` class which would handle API calls and maintaining message history. The first advancement we made was to also define a `Human` class and make both `Human` and `Bot` subclasses of the parent `Participant` class, which offers bot/human-agnostic access to `chat` and `receive_message` methods. Using this parent class, we created a `Meeting` class which could allow two or more participants to trade off sending messages and receiving the other's messages. Finally we created the overall `Corp` class which could be instantiated with a `name` and `mission`, and would create the `founder` and `cofounder` objects as `Human` and `Bot` respectively. Upon calling its `start` method, it would create and begin the meeting.
