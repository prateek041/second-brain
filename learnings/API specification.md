# List of API endpoints

Currently there are three assistants
- Core : Core handles the interaction between user and system.
- Nutrition : Nutrition is responsible for creating and altering nutrition plans.
- Fitness: Fitness is responsible for creating and altering workout plans.

list of API endpoints

- **/ping**: for testing the connection with the API
- **Threads**: 
	- */newThread*: for creating an empty thread.
	- */getThreadMessages*: for getting messages in the give thread
- **Vitals**:
	- *ping*: for testing the vitals route.
	- *getBloodPressure*: there are many here, just not registered.
- **user**: 
	- *register* : create new user.
	- *login*: logging in and returning refresh and access tokens.
- **Chat** /api/chat:
	- */continue*: continues the thread when thread Id and message is provided.
	- */testChat* : just a testing interface for chatting with core assistant
- Get Assistant
- Get Assistant tools

list of functions and their usages

- **Nutrition**:
	- *createNutritionPlan*: creates a nutrition plan when a certain amount of information is collected and provided.