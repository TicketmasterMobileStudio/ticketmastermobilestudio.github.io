---
title: “Creating Conversational Interfaces: Highlights from Google I/O 2017”
author: Marcelle Gibble
categories: Product Design
tags: [Google I/O, voice interfaces, conversational interfaces]
---

Natural language processing has improved exponentially over the last few years. As expected, one of the clear themes of Google I/O 2017 was building conversational interfaces.
 
As part of a team invited to create one of the first Actions on Google with transactional and multi-modal capabilities, I paid particular attention to sessions that focused on designing user experiences for voice interactions. While all the speakers had something unique to offer, I felt like all the messages boiled down to one maxim: 
 
> **Always imagine the whole experience you are creating as a conversation between two people.**


### Make It A Conversation

One of the unmistakable benefits of voice interactions is how efficiently they can understand and execute tasks: while a GUI might take several clicks and some typing, a voice command can be understood and executed in a matter of seconds. Leverage this advantage heavily, but use caution. The voice interface experience you design must be more than some way to bypass the typing and clicking involved in a traditional GUI. We tend to use the terms “voice” and “conversation” interchangeably (i.e. “voice user interface” equals “conversational user interface”). However, to truly reap the benefits of a voice interface, design your experience so the user feels like it is a conversation with another human.

### Create Your Personality

Give your product a real human voice.
 
> “Exactly _why_ do I need to make the interaction sound like a personal conversation?” asks the analytical, dispassionate engineer. 
 
The act of conversation has the inherent side effect of being more personal and intimate. After all, it is the natural human reaction to personify things that speak (didn’t you enjoy the talking clock and candlestick in Disney’s _Beauty and the Beast_?). Use this to your advantage and set the tone, mood, and style to pervade the experience. 
 
Start with relatable language. Conversation is intuitive and you shouldn’t have to train people how to use your product. Do not strive for the automated teller parallel (e.g., “say ‘help’ if you need help, say ‘next’ if you want to hear more options”). Rather, be an expert in your specific area and direct the conversation accordingly. When you create the right persona, users won’t expect an all-knowing computer, just a proficient personality that is an expert in her field.

### Be Concise 

Unlike a GUI, where users can read and decide on options at their own pace, once a verbal response is given, users must rely on their memory of the spoken response to understand the next step. Be concise and structure your requests for easy recall. Additionally, skimming a paragraph isn’t an option with a voice interface, which will frustrate a user who wants to advance quickly. To prevent this, be concise with your responses and remember foremost to answer the question the user asked. The whole experience must be easier than the alternative, or your users won’t be around for long.
 
### Prevent Frustration

Sometimes there are tough moments when users either didn’t say anything, or their response was misunderstood. Users have less tolerance for error in voice interfaces, because the root cause can be less obvious (you _know_ you made a mistake in a GUI when you fat-fingered something!). How do you recover gracefully from these situations? 
 
Acknowledge the ambiguity as soon as possible, since it will be more difficult to unravel later. Attempt to handle the correction using normal conversation. This can be accomplished by building a list of response prompts and randomizing them. Simply restating the exact question will come off as rude and impatient. Additionally, consider breaking the question into parts, or helping the user understand why you need more information. Understanding their progress through the experience will make them feel more comfortable answering the questions.
 
Finally, as a general rule, if you have unsuccessfully asked the question three times (because presumably you must have the information to continue), exit the experience gracefully and hope it goes more smoothly next time.


### Design a Great Experience
 
Viewed through the lens of social conventions, all these tips are fairly obvious when you think about how you would like a real person to respond. 
 
That being said, as I listened to all of these special cases, the engineer in me started to panic while mentally tabulating the growing lines of code required. This sounds like a LOT of extra work! Indeed it is. But to ensure my users have a great experience, I’m motivated to develop and design for that extra effort.
 
I hope you’ve found this primer helpful. If you’d like more detailed information from the experts, here are some great sessions from Google I/O 2017 on developing conversational user interfaces:

[Finding the Right Voice Interactions for your App](https://www.youtube.com/watch?v=0PmWruLLUoE&list=PLOU2XLYxmsIKC8eODk_RNCWv3fBcLvMMy&index=78){:target='_blank'}

[In Conversation, There Are No Errors](https://www.youtube.com/watch?v=oOLo071Pj1U&index=101&list=PLOU2XLYxmsIKC8eODk_RNCWv3fBcLvMMy){:target='_blank'}

[PullString: Storytelling in the Age of Conversational Interfaces](https://www.youtube.com/watch?v=pz8EQHMRr6Y&index=133&list=PLOU2XLYxmsIKC8eODk_RNCWv3fBcLvMMy&t=1s){:target='_blank'}

[Applying Built-in Hacks of Conversation to Your Voice UI](https://www.youtube.com/watch?v=wuDP_eygsvs&index=151&list=PLOU2XLYxmsIKC8eODk_RNCWv3fBcLvMMy){:target='_blank'}


