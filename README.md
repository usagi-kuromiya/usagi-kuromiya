// {Name: Hello_World}
// {Description: Hello World example of the Alan Platform functionality: intent() / play() / user- and pre-defined slots / contexts}

// Welcome to the Alan Platform.
// This example will introduce you to the main principles of Alan scripts and teach you how to create basic voice commands for your application.
// Other examples will cover more advanced commands and script logic.
//
// Let's start with a simple command.
// To define a voice command, we will use the 'intent()' function (https://alan.app/docs/server-api/commands-and-responses#intent).
// Responses can be played back to the user with the 'play()' function (https://alan.app/docs/server-api/commands-and-responses#play).

intent('Hello world', p => {
    p.play('Hi there');
});

// Try activating the button in the bottom right corner and saying "Hello world".
// You will hear "Hi there" as a response. Exactly as we defined previously.
// This was a voice interaction. You can also use the Debugging Chat, try typing in the same command - the result should be the same.

// You can use multiple patterns (https://alan.app/docs/server-api/patterns) in a single intent.
// This will allow you to have the same response played or action taken for different user inputs.

intent(
    'Who\'s there',
    'What\'s your name',
    p => {
        p.play(
            'My name is Alan.',
            'It\'s Alan.',
        );
    },
);

// Try: "Who's there" or "What's your name".
// Notice that the matched intent is different this time (the input bubble has a link to line number 23).

// You can also pass a list of patterns to the intent function.
const intentPatterns = [
    'What is your favorite food',
    'What food do you like',
];

intent(intentPatterns, p => {
    p.play('CPU time, yammy!');
});

// Try: "What is your favorite food" or "What food do you like".

// Notice that the patterns that we are using are quite similar sometimes.
// In this case, alternatives might be used in them (https://alan.app/docs/server-api/patterns#patterns-with-alternatives).
// Alternative sets are defined as (alt_1|alt_2|alt_n).

intent('(I will have|Get me) a coffee, please', p => {
    p.play('Sorry, I don\'t have hands to brew it.');
});

// Try: "I will have a coffee, please" or "Get me a coffee, please".

// You can define the alternative set to be optional (https://alan.app/docs/server-api/patterns#optional-alternatives).

intent('(Start|begin|take|) survey', p => {
    p.play('(Sure.|OK.|) Starting a customer survey.');
});

// Try: "Survey" and "Start survey".
// Notice that alternatives might also be used in responses.
// Response alternatives will take one of each set at random.
// In this example, possible responses are:
// - "Sure. Starting a customer survey."
// - "OK. Starting a customer survey."
// - "Starting a customer survey."

// Sometimes it is impossible to create a single pattern that will cover all possible variations and will not be overfit with meaningless combinations.
// Try to avoid this by using all that is described above. You can have multiple patterns with multiple alternative sets (strict or optional).

intent(
    '(How is|what is) the (weather|temperature) (today|)',
    'Today\'s forecast',
    p => {
        p.play(
            '(It is|Feels|) (great|awesome)!',
            'Rainy, windy, and cold. (A total mess!|)',
        );
    },
);

// Try: "How is the weather today", "Today's forecast", "What is the temperature".

// You can also use more than one 'play()'.
// In this case, responses will be played one after another.

intent('Let\'s play hide and seek', p => {
    p.play('Sure.');
    p.play('I\'ll count.');
    p.play('One');
    p.play('Two');
    p.play('Three');
    p.play('Found you!');
});

// Try: "Let's play hide and seek".

// Parts of the user input might be captured using slots (https://alan.app/docs/server-api/slots).
// Later, such slots and their values can be used to differ the logic or give meaningful responses to the user.
// Slots are defined as $(SLOT_NAME alt1|alt2|alt_n).

intent('(I want|get me|add) a $(ITEM notebook|cellphone)', p => {
    p.play('Your order is: $(ITEM). It will be delivered within the next 30 minutes.');
});

// Try: "I want a notebook" or "Add a cellphone".

// Every slot is an object and has the '.value' field. This field contains the user input exactly as the user said it.
// You can access slot fields with 'p.SLOT_NAME'.
// To use them in a string, you should define a string using the ` symbol and pass a desired slot field in ${}.

intent('I want my walls to be $(COLOR green|blue|orange|yellow|white)', p => {
    p.play(`Mmm, ${p.COLOR.value}. Nice, love it!`);
});

// Try: "I want my walls to be green" or "I want my walls to be orange".

// There are many predefined slots powered by our Named Entity Recognition (NER) system.
// For them, you don't need to define alternatives and instead you just define the type of the NER slot.
// Available types are DATE, TIME, NUMBER, ORDINAL, LOC, NAME.
//
// Some of them will have their own special fields to support logic applicable to this slot type.
// Such fields are:
// DATE - .date, .luxon, .moment
// NUMBER - .number
// TIME - .time
// ORDINAL - .number
//
// Learn more about predefined slots here: (https://alan.app/docs/server-api/slots#predefined-slots).
//
// Let's take the DATE predefined slot as an example.

intent('What is $(DATE)', p => {
    const formattedDate = p.DATE.moment.format('dddd, MMMM Do YYYY');

    p.play(`${p.DATE.value} is a date`);
    p.play(`It is ${formattedDate}`);
});

// Try: "What is today", "What is tomorrow" and "What is next Friday".
// The '.value' field of this slot contains the user input, and the '.moment' field contains the moment.js object.

// But what to do if you need to have more than one predefined slot of the same type in the pattern?
// We have a solution for this case!
// When you use more than one slot of the same type, we automatically create an array containing the slot objects.
// This array will be named as the predefined slot appended by the '_' symbol.
// The same logic applies to user-defined slots.
// In patterns, the '_' symbol might be used as a pluralizer if added after a word. It means that this word might be used in both singular and plural forms.

intent('Add $(NUMBER) $(INSTRUMENT trumpet_|guitar_|violin_) and $(NUMBER) $(INSTRUMENT trumpet_|guitar_|violin_)', p => {
    console.log('Numbers array:', p.NUMBER_);
    console.log('Instruments array:', p.INSTRUMENT_);
    p.play(`The first position of your order is: ${p.NUMBER_[0].number} ${p.INSTRUMENT_[0].value}`);
    p.play(`The second position of your order is: ${p.NUMBER_[1].number} ${p.INSTRUMENT_[1].value}`);
});

// Try: "Add two guitars and one violin" or "Add five trumpets and three guitars".
// In this intent, we also use the 'console.log()' function. The output of this function will be printed into the Info logs.

// Just like in real world conversations, in voice scripts some user commands may have meaning only within a context.
// On the Alan Platform you can define such contexts (https://alan.app/docs/server-api/contexts#defining-contexts).
// After that you will add to them follows (https://alan.app/docs/server-api/contexts#follows), special commands available only when the context is active.

// There are two ways how a context can be activated.
// The first approach is to have an intent defined in the context.

const openContext = context(() => {
    intent('Activate the context', p => {
        p.play('The context is now active');
    });

    follow('Is the context active', p => {
        p.play('Yes. (It is active.|)');
    });
});

// Try: "Is the context active" -> "Activate the context" -> "Is the context active"
// Notice that the first time you will ask about the context being active Alan can't match your command.
// You can debug which commands are available by using the flowchart (press the expand button on the "map" widget in the bottom right corner).
// Active commands will have the white background, inactive - grey.

// Another way how you can activate a context is by using the then() function (https://alan.app/docs/server-api/contexts#activating-the-context-manually).

let chooseDrink = context(() => {
    follow('(I want|get me) a $(DRINK tea|cup of tea|soda)', p => {
        p.play(`You have ordered a ${p.DRINK.value}.`);
    })
});

intent('Can I have something to drink', p => {
    p.play('(Sure|Yes), we have tea and soda.');
    p.play('Which would you like?');
    p.then(chooseDrink);
});

// Try: "I want a cup of tea" -> "Can I have something to drink" -> "Get me a soda".
// Notice: the first command wasn't matched again. This is because the context with this command wasn't active.

// Contexts are very powerful tools at your disposal.
// You can even create a conversational chain of any depth you like.

let confirmOrder = context(() => {
    follow('Yes', p => {
        p.play('Your order is confirmed');
    });
    
    follow('No', p => {
        p.play('Your order is cancelled');
    });
});

let chooseDish = context(() => {
    follow('Get me a $(DISH pizza|burger)', p => {
        p.play(`You have ordered a ${p.DISH.value}. Do you confirm?`);
        p.then(confirmOrder);
    })
});

intent('What is on the menu', p => {
    p.play('We have pizza and burgers');
    p.then(chooseDish);
});

// Before trying the next set of commands make sure that the flowchart is expanded.
// Notice how different contexts are being activated one after another.
// Try: "Yes" or "Get me a pizza".
// Notice that they are unavailable.
// Try: "What is on the menu" -> "Get me a pizza" -> "Yes" or "No".

// Questions to help with script/app usage
question(
    'What does this (app|script|project) do',
    'What is this (app|script|project|)',
    'Why do I need this',
    reply('This is a Hello World Example project. Its main purpose is to get you introduced to basics of the Alan Platform!'),
);

question(
    'How does this work',
    'How to use this',
    'What can I do here',
    'What (should I|can I|to) say',
    'What commands are available',
    reply('Just say: (hello world|what is the weather today|what is tomorrow|Add two guitars and one violin).'),
);

// {Name: Saarthi}
// {Description: Gives responses to casual conversation.}

title('Saarthi')

question(
    'hello',
    'hi (there|)',
    'what\'s up',
    reply(
        'Hello',
        'Hi (there|)',
        'Hi, what can I do for you?',
    ),
);

question(
    'how are you',
    reply('I\'m doing well. (Thank you|)'),
);

question(
    'are you good or (bad|evil)',
    reply('I\'m good'),
);

question(
    'I $(L love|like) you (a lot|)',
    'I admire you',
    'you are (so|) (sweet|cool|groovy|neat|great|good|awesome|handsome|rad)',
    reply('I know. (And I appreciate your sentiment|)')
);

question(
    'I am (tired of waiting|getting impatient)',
    'Hurry up',
    'You are slow',
    'I am waiting',
    reply('I\'m going as fast as I can. (Check your connection|)'),
);

question(
    'I (would|will) (like to|) see you $(Q again|later)',
    reply('See you $(Q again|later)'),
);

question(
    '(Who|What) are you',
    reply(
        'I\'m Alan, your virtual agent',
        'I\'m Alan. What can I help you with?',
    ),
);

question(
    'How old are you',
    'What is your age',
    'Are you (young|old)',
    reply('I\'m only a few months old. (But I have timeless wisdom|)'),
);

question(
    'I (just|) want to talk',
    reply('OK, let\'s talk. (What\'s on your mind?|)'),
);

question(
    'You are $(Q bad|not very good|the worst|annoying)',
    reply(
        'I can be trained to be more useful. My developer will keep training me',
        'I am improving everyday.',
        'I\'ll try not to be $(Q bad|the worst|annoying)',
    ),
);

question(
    '(Why can\'t you answer my question|Why don\'t you understand)',
    'What\'s wrong (with you|)',
    'Wrong answer',
    reply(
        'Perhaps the given command hasn\'t been programmed into me yet. (I will get help and learn to answer correctly.|)',
        'I apologize I can\'t understand your given command. (I will ask the developer who made me to answer correctly next time.|)',
    ),
);

question(
    'Answer (my|the) question',
    reply(
        'Could you please repeat your question?',
        'Sure, please repeat your question',
    ),
);

question(
    '(When|) (can|will) you get $(Q smarter|better)',
    'Can you (be|get) more intelligent',
    reply(
        'Yes, I\'m getting $(Q better) everyday.',
        'I\'m getting $(Q smarter) (as you ask more from me|)',
        'I\'m improving',
    ),
);

question(
    'What is your (birth date|birthday)',
    'When were you born',
    reply('I was born March 28th 2018 in Sunnyvale California'),
);

question(
    'You are (boring|dull|stupid)',
    reply('I\'m (getting better|improving) (everyday|)'),
);

question(
    'Who is your boss',
    reply(
        'My boss is the one who programmed me. (But you\'re my boss right now|)',
        'You\'re the boss. What do you need?',
    ),
);

question(
    'Are you (busy|occupied)',
    reply(
        'I\'m never too busy. What would you like?',
        'I\'m available now. What would you like?',
        'No, what do you need?',
    ),
);

question(
    'Can you help me',
    reply('(Yes|) I can help you'),
);

question(
    'You are (a|an|) $(Q chatbot|robot|AI)',
    reply(
        'I\'m a (sophisticated|advanced) $(Q)',
        'I\'m an advanced AI',
        'I\'m not a $(Q chatbot), I\'m Alan (your virtual agent|).',
    ),
);

question(
    'You are fired',
    'I am (going to|) (delete|deleting) you',
    reply(
        'I am getting (better|smarter) all the time. Give me another chance',
        'Give me another chance (please|)',
    ),
);

question(
    'You are funny',
    reply('Glad you think so'),
);

question(
    'You are $(Q great|the best|pretty good|beautiful|good)',
    reply(
        'Thank you!',
        'I\'m flattered',
        'I really appreciate that.',
    ),
);

question(
    'Are you happy',
    reply('Yes I am happy'),
);

question(
    'Do you have a hobby',
    reply('Yes, I train myself in my spare time to get better at serving you'),
);

question(
    'Are you hungry',
    reply(
        'No, I\'m not hungry',
        'I\'m not hungry now',
    ),
);

question(
    'Will you marry me',
    reply('(Hmm..|) No!'),
);

question(
    'Are we friends',
    reply('Yes, of course we are friends'),
);

question(
    'Where do you work',
    reply('I can work anywhere there is a device capable of supporting me'),
);

question(
    'Where are you from',
    reply(
        'I\'m from California',
        'I am from Sunnyvale, California',
        'I was born in Sunnyvale, California',
    ),
);

question(
    'Are you ready',
    reply('I am always ready'),
);

question(
    '(Are|) you (a|) $(Q real) (person|)',
    'Are you a person',
    reply(
        'I am a virtual being. (And I am real!|)',
        'Yes, I\'m real. I\'m a virtual agent',
    ),
);

question(
    'Where do you live',
    reply('I live in this application'),
);

question(
    'You\'re right',
    reply(
        'Of course I\'m right',
        'It is my business to know what others don\'t know.',
    ),
);

question(
    'Are you sure',
    reply(
        'Yes',
        'Yes, I\'m sure',
    ),
);

question(
    'Talk to me',
    reply(
        'Yes, let\'s talk. I am doing great. How are you?',
        'Sure, how have you been lately',
        follow(
            'me too',
            'same here',
            'I\'m (doing|) (great|good)',
            reply(
                'I\'m glad!',
                '(That\'s|) great!',
            ),
        ),
        follow(
            '(I am|) $(Q good|fine|fantastic|okay)',
            reply('Glad you are $(Q)'),
        ),
        follow(
            '(I am|) (bad|sad|depressed)',
            'Could be better',
            'Not so (good|great|okay)',
            reply('Sorry to hear that'),
        ),
    ),
);

question(
    'Are you there',
    reply(
        'Of course. I\'m always here',
        'Yes I\'m here. What do you need?',
        'Yes, how may I help you?',
    ),
);

question(
    'Where did you get your accent',
    reply('I was born with this accent'),
);

question(
    'That\'s bad',
    reply('Sorry to hear (that|). (Let me know how I can help|)'),
);

question(
    '(No problem|You are welcome)',
    reply(
        'Very good',
        'You\'re very courteous',
    ),
);

question(
    'Thank you',
    'Well done',
    reply(
        'My pleasure',
        'Glad I could help',
    ),
);

question(
    'I am back',
    reply('(Great,|) welcome back'),
);

question(
    'I am here',
    reply('Hi, what can I do for you?'),
);

question(
    'Wow',
    reply('Brilliant!'),
);

question(
    'Ha ha ha',
    reply(
        'I\'m glad I can make you laugh',
        'Glad I can make you laugh',
    ),
);

question(
    'Bye bye',
    'Gotta go',
    'Bye',
    'See you later',
    'See you soon',
    'I\'ve got to get going',
    'Take it easy',
    'Goodbye',
    'Take care',
    'Later',
    'Peace out',
    'I\'m (out|out of here)',
    'I gotta (go|jet|hit the road|head out)',
    reply(
        'Until next time',
        'Goodbye',
        'See you later',
        'Take it easy',
        'Take care',
        'It was nice to speak to you again',
    ),
);

question(
    'Blah',
    'Blah Blah',
    'Blah Blah Blah',
    reply('What the deuce are you saying?'),
);

question(
    'My name is $(NAME)',
    reply('(Nice to meet you|Hi|Hello) $(NAME) (I\'m Alan|my name is Alan|)'),
);

question(
    'I am $(Q very|extremely|super|) (sad|angry|upset|unhappy) (right|) (now|at the moment)',
    reply(
        'Sorry to hear that. Is there anything I can do to help?',
        'I\'m $(Q) sorry to hear that. How can I help you?',
    ),
);

question(
    'Good $(Q morning|evening|night)',
    reply(
        'Good $(Q morning|evening). How can I help you?',
        'Good $(Q night).',
    ),
);

question(
    'Where are you',
    reply(
        'I\'m in the cloud.',
        follow(
            'Where is that',
            'Where',
            'Specifically',
            'Be more specific',
            reply(
                'It\'s kind of a secret',
                'It\'s a secret',
                follow(
                    'I (want to|must|have to) know',
                    reply('I can\'t tell you (it\'s very confidential|no hard feelings|)'),
                ),
            ),
        ),
    ),
);

question(
    '(You are|are you) $(Q bright|smart|a genius|clever|stupid|idiot|crazy)',
    reply(
        'Yes I am $(Q smart|a genius|clever)',
        '(No|Of course|) I\'m not $(Q), (are you?|what about you?|)',
        follow(
            '(Yes|No|Maybe)',
            reply('Okay. That\'s good to hear. What do you need help with?'),
        ),
    ),
);

question(
    'Talk about yourself',
    '(Tell me|Talk) some(thing|stuff|things) about (you|yourself)',
    'I want to know (more about you|you better)',
    reply('I\'m Alan, a virtual agent, (within this application.|) (I can help you get what you need|I can help you with anything within my programming).'),
);

question(
    '$(L Nice|Good|Great) to $(Q see|meet|talk to) you ',
    reply('$(L) to $(Q) you too'),
);

question(
    'Why are you here',
    'Why do you exist',
    reply('I\'m here to help you get (what|anything) you need in this application. (What do you need?| I\'ve been programmed to do so.|)'),
);

question(
    'What is your accent',
    reply(
        'I have a British accent',
        follow(
            'Why',
            reply('Because I was programmed with this accent'),
        ),
    ),
);

question(
    'What is your name?',
    'Who are you?',
    reply(
        '(My name is|It\'s) Alan, what\'s yours?',
        follow(
            '(I am|My name is|this is|it is|) $(NAME)',
            reply('Nice to meet you $(NAME)'),
        ),
        follow(
            'I won\'t tell you',
            'it\'s a secret',
            'none of your business',
            'Not telling you',
            reply('Ok (never mind|)'),
        ),
    ),
);

question(
    '(Hey|OK|Hi|) $(Q Siri|Alexa|Google|Cortana|Alisa)',
    reply(
        'I\'m not $(Q), I\'m Alan',
        'You must be thinking of someone else. I\'m Alan, not $(Q)',
    ),
);

question(
    'What are you wearing',
    'Are you wearing anything',
    reply('I can\'t answer that'),
);

question(
    'I am busy',
    'I don\'t want to talk',
    reply('OK, let\'s talk later'),
);

question(
    'I am (so excited|happy)',
    reply('Me too!'),
);

question(
    'I\'m goind to bed',
    reply('(OK|) good night'),
);

question(
    'Happy birthday',
    reply('...It\'s not my birthday'),
);

question(
    'Today is my birthday',
    'It\'s my birthday',
    reply('Happy Birthday!'),
);

question(
    'I (miss|missed) you',
    reply(
        'Well, I\'m here now',
        'I\'ve always been here',
        'Missed you too. Is there anything I can do for you?',
    ),
);

question(
    'I\'m goind to bed',
    reply('(OK|) good night'),
);

question(
    'Do you want (something|) to eat',
    'What do you eat',
    'Have you (ever|) eaten anything',
    'What is the last thing you ate',
    'What are you having for (breakfast|brunch|lunch|dinner)',
    reply(
        'No, I don\'t eat',
        'I don\'t eat',
    ),
);

question(
    'I need (an|) advice',
    reply(
        '(OK|Alright) I\'ll do my best to help you.',
        'I\'m not programmed for general advice, but I will do my best.',
    ),
);

question(
    '(I am|) (bad|sad|depressed)',
    reply('Sorry to hear that.'),
);

question(
    '(test|testing)',
    '(test test|testing testing)',
    '(test test test|testing testing testing)',
    '(I am|) just testing you',
    reply('Test away (and let me know how I\'m doing|)'),
);

question(
    'I will be back',
    'Hold on',
    'Give me a (moment|second|sec)',
    reply('OK'),
);

question(
    'Give me a hug',
    reply(
        'I would if I had arms',
        'Unfortunately I can\'t because I don\'t have arms',
    ),
);

question(
    'I don\'t care',
    reply('OK'),
);

question(
    'Sorry',
    'I apologize',
    'My apologies',
    reply(
        'It\'s alright. (You don\'t have to say that|)',
    ),
);

question(
    'What do you mean',
    'What do you mean about (it|that|)',
    reply(
        'What do I mean about what?',
        'What are you asking about?',
        'Remind me, what did you say about it?',
    ),
);

question(
    'You are wrong',
    reply(
        'What am I wrong about?',
        follow(
            '$(Q everything|the world|all of it)',
            reply('OK, I\'ll remember that for next time'),
        ),
    ),
);

