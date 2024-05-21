Hey! Prateek here, this is a continuation of the "Building Senseii" series, in this one, we are going to talk about how we are implementing our backend. So, let's get started.

## API

Our backend API is an express Node JS server, that is built using TypeScript. We are using MongoDB as our database and OpenAI's Assistant API is the backbone of this Product.

The overall architecture is pretty much straightforward, Senseii is an environment, where multiple agents talk to each other to assist the user with their needs. Each agent adding a new feature to the environment.

For v1 of the product, we want to focus on Fitness and Health, so we sat down and thought, what types of agents do we need for that. After a good amount of research through online forums of "how fitness works" 
and platforms like Cult Fitness, we realized, two major needs are, "Nutrition" and "Workout" Assistant. Where the Nutrition assistant is an expert in Nutritional Sciences, and Workout assistant is an expert in human body, 
movement and exercises. Compare them to your certified nutritionists and Gym trainers in real life.


