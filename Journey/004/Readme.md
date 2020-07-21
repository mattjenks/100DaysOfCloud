## Cloud Research

- Decided to create a very simple foundation chatbot to try tying AWS Lex to a lambda function. I had not used AWS Lex before.
- Familiarized myself with the [Lex Lingo](https://docs.aws.amazon.com/lex/latest/dg/how-it-works.html)
- Created an intent using the Lex UI to ask for help for a specific department
  - This included sample utterances, such as
  - How do I get help with RD
  - How do I get help with Research
  - Research
  - Help with Research
  - Help me with Research
- created a Lex Slot for the department
  - Actually added a few like Research and HR
- Created a default output if the slot was not understood
- Hooked the slot to the Lambda function to process the slot and give static output from the lambda function.
- Stopped here for now, but will revisit and add a database and restful api to allow for creating more dynamic content.

## Social Proof

✍️ Show that you shared your process on Twitter or LinkedIn

[Hooking Lambda to Lex](link)
