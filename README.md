**Digi**

Meet your new ServiceNow Digital Developer, ‘Digi’.

**What Digi does**:\
  Digi gives ServiceNow developers a github co-pilot like experience directly on the ServiceNow platform to assist with generating code.
**Why Digi was made**:\
  Open Source provides advantages to innovate however we want, collectively as a ServiceNow ecosystem. This is especially important when it comes to AI and our precious data. We should all be able to choose what data to integrate with and where the security perimeter is. And moreover, there should not be a price tag associated with it.
\\
**How Digi works**:\
It is more than a code generator; Digi has a framework which provides a flexible and configurable AI future on ServiceNow (paradigm details are in the repo). Some examples:\
* Integrate with popular models or AI engines (codex, bard, your private Starcoder stack).\
* Configure parameters for the chosen model.\
 * Configure prompts, prefixes and postfixes for messages sent to the AI engine.\
* Have a full trace log of everything that AI does on your platform.\
* Validate changes prior to allowing Digi to directly modify your environment.\
When a user types ‘@digi’ in a ServiceNow script field followed by a written statement then hits tab, Digi will send that statement (and any configured prompts) to the configured AI engine and provide the script on the screen for the developer. The developer then accepts the changes and Digi will add the new code to the script field.\
Head to the repo, take Digi for a spin, collaborate with the project team and create a pull request. Let’s make Digi better together and make AI work for us in many ways.\
\\
\\
**Database Architecture**\
![image](https://github.com/rapdev-io/digi/assets/34044520/96c1abb8-f29a-4161-b6cf-a894e03f247a)

**Request**
Defines a request made by a user to an AI engine. 
  * A transactional table.
  * A row represents the transactional-request data between the user and the AI engine.
  * When complete,  it becomes an audit/history of the transaction.
    Important data items: 
      * Action - REF to the Action table.  Identifies what type of request is being made.
      * AI Engine -  REF to AI Engine table.  Identifies which AI engine to be used.
      * AI Model - REF to AI Model table.  Identifies which model of the AI Engine to be used.
      * User Text - The user’s question or instruction (the “Prompt”) to the AI engine.
      * Source Reference - The script or document (if any) to which the Prompt refers.  Ex: If the user wants AI to modify a script, the script to be modified should be put here. 
      * AI Request -  The actual text sent to the AI.  It may include additional prompts added by Digi to facilitate better responses from the AI. 
      * AI Answer - The response from the AI.
      * AI Answer Block - The script or document extracted out of the AI’s response.  Ex: If the AI returns text with an embedded script example, the script example would be extracted and placed here.
      * Status Message - If an error occurs, the text of the error will appear here.  Warnings may also show here. 
**Related tables**: 
  Request Log - log messages directly related to this Request. 



AI Engine
Defines an AI Engine which will be answering user Requests.

A configuration table.
A row represents a specific version of an AI Engine which will answer Requests.
Important Data Items:
Name - The generic name of the AI Engine, like ChatGPT or OpenAI.
Version - The version number of the AI Engine.
Engine ID - Calculated field combining the name and version.Should be unique.
API Key - the API key required to access the AI.
REST Message - REF to a row on SN’s OOB Rest Message table.
REST Method - REF to a row on SN’s OOB Rest Method table.
Endpoint - the API endpoint URL.
API Component Name - The name of a script include or Flow which handles the physical API call to the AI.
API Component Type - Whether the API Component Name is a Flow or a Script Include.
Related tables: 
AI Model - specifies various models available to this AI Engine, like code, text, etc.
AI Prompts - specifies prefix/postfix/formatting prompts used to sandwich the user request text prior to sending to the AI Engine.

AI Model
Defines the various language models used by an AI Engine.  Ex:  OpenAI has several models like text-davinci-002, text-davinci-003, code-davinci-003.
A configuration table.
A row represents a model for a particular AI Engine.
Important Data Items:
AI Engine - REF to a row on the AI Engine table.
Model - the name of the model.
Type - the type of model (text / code).
Max Tokens - the maximum number of tokens allowed by this model.
Default - checked if this row is the default model for its AI.  Only one row can be checked for a given model.  This is enforced by a Business Rule.
Related Tables:
AI Model Params - lists the various REST parameters and their values needed to successfully do an API call to the AI.

AI Model Params
Defines the various REST parameters which form the body of the JSON payload sent to the AI.  Other parameters may be added here with a different Param Type for other purposes, but use-cases for this are TBD.
A configuration table.
A row represents a single parameter and its value.
Important Data Items: 
AI Engine Model - REF to a row in the AI Engine Model table.
Param name - the physical name required by the AI in its JSON payload.  
Param value - the value for the Param name. 
Param data type - the data type for the param value (string, integer, etc).
Param type - the purpose for this parameter (REST, …).
Related Tables: 
none.
AI Prompt
Defines the various prompts used by Digi to amend a user’s prompts prior to engaging the AI.

A configuration table.
A row represents the textual phrase used to augment the user’s request text so that the AI engine can return a better answer. 
A given user request may be sandwiched by multiple prefix, postfix, and formatting prompts.
Important Data Items: 
AI Engine - REF to a row in the AI Engine table. (Wondering if this table should reference the AI Model table instead).
Request Action - (optional) REF to a row in the Action table.  The nature of the users request may impact what prompts are amended to the user’s prompt.  If empty this row applies to ALL actions!   See “Configuring” notes below.
Type - the type of amendment (prefix, postfix, format, source).  
Prefix - put these before the user’s prompt.
Postfix - put these after the user’s prompt.
Format - put these after the Postfix prompts.
Source - where the Request record’s “source reference” data should be added to the prompt.
Sequence - the order in which prompts should amend the user’s text.
Note:  the sequence number on a row which specifies an action will override a row with the same sequence number but which has no action specified.  See the “Configuring” discussion below. 
Text - the actual text of the amendment.   Note that this may include embedded <functionName> markers.  These are function names in the script include DigiApplication which will substitute values for the markers.
Related tables:
None
Configuring: 
Digi will select all rows where the Action is empty or equal to the Action on the Request.  It sequences the rows by the sequence number in order to build the AI’s final prompt.  
This allows you to configure a “universal” set of prompts which apply to all actions, without having to specify the same prompts for every different action.
If two rows appear with the same sequence number (ex:  one “universal” prompt and one for “Write a script”) then Digi will use the more specific one–the one which specifies an Action.
So when you build prompts for a particular Action, filter the list so it includes rows with an empty Action.  Then you can sequence them by the sequence number and add any additional prompts (not already in the “universal” list) that are required by the Action. 
In the example below, notice how the “universal” and the specific prompts are woven together based on the sequence number.

Action
Defines the types of questions or instructions Digi can process.
A configuration table.
A row represents a single type of question or instructions available to the user.
Important Data Items: 
Name - A short moniker describing the action, like “Ask a question,” “Write a script,”, or “Compose a KB article.”
Instances - the list of ServiceNow instances in which this Action is allowed.  Ex: “Write a script” may not be allows in the production instance. 
Roles - the list of roles in which this Actionis allowed.  Ex:  “Write a script” will only be allowed for admin users.
Related Tables
none.
Log
Similar to the system log, but tied directly to a Request, it houses transactional information logged here during the lifecycle of a Request.
A transactional table.
Captures various debug, info, warning, and error messages logged by various digi components.
Important Data Items:
Request - REF to a row in the Request table.
Level - the type of message (debug, info, warn, error).
Source - the Digi component that wrote the message.
Message - the text of the message
Timestamp - the time (with milliseconds!) in which the message was logged.
Related tables: 
None



What Digi does:
Digi gives ServiceNow developers a github co-pilot like experience directly on the ServiceNow platform to assist with generating code.
Why Digi was made:
Open Source provides advantages to innovate however we want, collectively as a ServiceNow ecosystem. This is especially important when it comes to AI and our precious data. We should all be able to choose what data to integrate with and where the security perimeter is. And moreover, there should not be a price tag associated with it.
Where Digi is:
Digi lives on github, here:
How Digi works:
It is more than a code generator; Digi has a framework which provides a flexible and configurable AI future on ServiceNow (paradigm details are in the repo). Some examples:
·         Integrate with popular models or AI engines (codex, bard, your private Starcoder stack).
·         Configure parameters for the chosen model.
·         Configure prompts, prefixes and postfixes for messages sent to the AI engine.
·         Have a full trace log of everything that AI does on your platform.
·         Validate changes prior to allowing Digi to directly modify your environment.
When a user types ‘@digi’ in a ServiceNow script field followed by a written statement then hits tab, Digi will send that statement (and any configured prompts) to the configured AI engine and provide the script on the screen for the developer. The developer then accepts the changes and Digi will add the new code to the script field.
Head to the repo, take Digi for a spin, collaborate with the project team and create a pull request. Let’s make Digi better together and make AI work for us in many ways.
The Future for Digi:
1.       Pre-training data collation
a.       Ready to run jobs that will take relevant information in your ServiceNow instance and the publicly available ServiceNow information to prepare structured pre-training data for your models.
2.       Starcoder in a box
a.       IaC for a plug and play Starcoder stack that you can push to your safest subnet and integrate with to train your sensitive data behind your very own firewalls.
3.       Test Driven Development:
a.       Give Digi an ATF Test and have it freely build a solution on the platform until the test passes.
4.       AI Actions
a.       Digi will do more than write code. Build a workflow, work a ticket, create new records, analyze an incident to provide real time context in the activity/work notes based on all the relevant data at its disposal.
There are many more amazing things Digi will do. Digi is going to be a big part of how the AI future works on ServiceNow, by putting flexibility, options, and openness first.
We want anyone interested to be a part of it.
How to get involved:
Star the repo and sign up to join the quarterly architectural board meeting (all details are in the repo). RapDev will host this meeting and invite people to speak about where we can collectively take AI on ServiceNow, together.






**The Future for Digi**:
1.       Pre-training data collation
a.       Ready to run jobs that will take relevant information in your ServiceNow instance and the publicly available ServiceNow information to prepare structured pre-training data for your models.
2.       Starcoder in a box
a.       IaC for a plug and play Starcoder stack that you can push to your safest subnet and integrate with to train your sensitive data behind your very own firewalls.
3.       Test Driven Development:
a.       Give Digi an ATF Test and have it freely build a solution on the platform until the test passes.
4.       AI Actions
a.       Digi will do more than write code. Build a workflow, work a ticket, create new records, analyze an incident to provide real time context in the activity/work notes based on all the relevant data at its disposal.
There are many more amazing things Digi will do. Digi is going to be a big part of how the AI future works on ServiceNow, by putting flexibility, options, and openness first.
We want anyone interested to be a part of it.
How to get involved:
Star the repo and sign up to join the quarterly architectural board meeting (all details are in the repo). RapDev will host this meeting and invite people to speak about where we can collectively take AI on ServiceNow, together.

