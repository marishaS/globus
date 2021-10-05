# Globus Coding Challenge

## Project setup
- Clone this repository to your local machine.
- Run ``` npm install``` to install dependencies
- Add a ``` .env.local`` file to your project. (project root folder)
- Edit your .env.local file and add the key ``` VUE_APP_API_ENDPOINT``` and its value.
  - example: ```VUE_APP_API_ENDPOINT=http://localhost:3000/api/```
  - Replace the ``` http://localhost:3000``` with the url on which the api application serves data.
- Run ``` npm run serve``` to run the application.
- Follow instructions on terminal to open the application in browser. Usually ``` localhost:8080```

> NOTE:
> Before visiting the application make sure that the api application is running.  
> 
> Follow-up :
> 
1. Pick one of your implementation decisions and list some of its pros and cons:
>IMPLEMENTATION DECISION : Status Calculation
> 
PROS:
> The calculation of various statuses using separate method calls makes the code cleaner and readable.
> 
> Appending this above calculated status to the JSON data as a custom status field makes the calculation of "nice status" cleaner.


CONS:

> This approach adds a new field to the data object which takes memory.
> 
> This increases the execution time since we iterate over each object in order to add this new property
> 
2. Briefly describe any testing you feel should be added if we were going to put this in production.
All the custom methods which are introduced in the application require testing for edge cases.