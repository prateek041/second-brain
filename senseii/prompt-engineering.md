# Strategies for Prompt engineering

## Medprompt

This technique comprises of three other techniques.
- Few shot selection
- Self-generated chain of thoughts
- choice shuffle ensembling

> TODO: Ask lokesh to get people generate diet plans for free.

### Few Shot selection

Make domain experts create exemplers for your use-case.

Out of multiple examples, we pull out the most relevant one with our use-case.

#### Process
- Convert the examples into embedding using OpenAI's embedding model
- Using K-nn, fetch n (1x, 2x, 3x...nx) test examples that are matching to the use-case of the user.
- Embed the most similar ones to the Prompt itself.

### Self generated chain of thoughts

Include instructions like "Think about it step by step" to make the model generate response by thinking it through and generating intermediate steps.

### Majority Vote ensembling

