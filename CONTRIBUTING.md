# Contribution Guidelines

If you are interested in contributing to Substrate Documentation, please follow these contribution 
guidelines.

* [PR Checklist](#pr-checklist)
* [Documentation Standards](#documentation-standards)
* [Documentation Style](#documentation-style)

## PR Checklist

- [ ] Are the audience and objective of the document clear? E.g. a document for developers that 
  should teach them about transaction fees.
- [ ] Is the writing:
  - **Clear**: No jargon
  - **Precise**: No ambiguous meanings
  - **Concise**: Free of superfluous detail
- [ ] Does it follow our [style guide](#documentation-style)?
- [ ] If this is a new page, does the PR include the appropriate infrastructure, e.g. adding the
  page to a sidebar?
- [ ] Build the page. Does it render properly? E.g. no funny lists or formatting.
- [ ] Do links go to rustdocs or devhub articles rather than code?
- [ ] If this PR addresses an issue in the queue, have you referenced it in the description?

## Documentation Standards

A document about what to document and how to document it for people who create things that need
documentation.

### Why document?

Substrate is a new framework for building blockchains. Effective documentation allows new people to join and grow the project by having all necessary information.

### Who is the audience?

The documentation described here is intended to address a technical audience, i.e. those who expect
to implement or exercise APIs or understand the internal dynamics of the blockchain system. These
standards are not intended for end-user product documentation.

The knowledge base houses instructions, conceptual overviews, and procedural documentation for 
developers who are working on Substrate. This directory includes documentation on how to get 
started, build, run, and test Substrate. You should organize the content that you create by 
specific activities, such as testing, getting started, or by workflow topic.

### What should I document?

Document protocols, introduce essential concepts, and explain how everything fits together.

- Conventions: e.g. this document about documentation, code style
- System Design: e.g. network stack, consensus, runtime, assumptions
- APIs: e.g. JSON-RPC, runtime functions
- Protocols: e.g. schemas, encodings, configuration files
- Tools: e.g. `subkey`, `build-spec`, `purge-chain`
- Workflows: e.g. environment set up, test methodologies, where to find various
  parts, how to get work done

### What documentation style guidelines should I follow?

It is important to follow documentation style guidelines to ensure that the documentation
created by a large number of contributors can flow together. See
[Documentation Style Guide](#documentation-style).

### How can I link to other docs in my documentation?

Use absolute paths starting with `/` and ending with `.md`, like `/overview/getting-started.md`.

### How can I link to source code in my documentation?

Use the rust-docs to link to specific parts of the Substrate source code, for example a module's `trait`. When referencing specific lines of code, copy those lines into the documentation.

DO NOT link to specific lines of code -- Substrate changes often and links that reference specific locations in the code become inaccurate fast and are not easily maintainable.

**Showing code can make an abstract idea concrete. It is now, however, a shortcut to writing a 
clear explanation about how something works.**

## Documentation Style

It is important to create documentation that follows similar guidelines. This allows documentation
to be clear and concise while allowing users to easily find necessary information.

These are general style guidelines that can help create clearer documentation:

- **Write in plain U.S. English.** You should write in plain U.S. English and try to avoid over-complicated words when you describe something.

- **Avoid using pronouns such as "I" or "we".** These can be quite ambiguous when someone reads the
  documentation. It is better to say "You should do…." instead of "We recommend that you do….". It
  is OK to use "you" as this allows the documentation to speak to a user.

- **Know your relative pronouns.**

	- "That" introduces a restrictive clause. It isn't preceded by a comma.
	- "Which" introduces a nonrestrictive clause and is preceded by a comma.
	- Quick tip that works 90% of the time: Every time you write "which", put a comma in front of 
	  it and see if it makes sense. If it does, then leave the comma. If it doesn't, change it to 
	  "that".
	- See: https://www.quickanddirtytips.com/education/grammar/which-versus-that

- **If you plan on using acronyms, you should define them the first time you write about them.** For
  example, looks good to me (LGTM). Don't assume that everyone will understand all acronyms. You do
  not need to define acronyms that might be considered industry standards such as TCP/IP.

- **In most cases, avoid future tense.** Words such as "will" are very ambiguous. For example "you
  will see" can lead to questions such as "when will I see this?". In 1 minute or in 20 minutes? In
  most cases, assume that when someone reads the documentation you are sitting next to them and
  reading the instructions to them.

- **Use active voice.** You should always try to write in the active voice since passive voice can
  make sentences very ambiguous and hard to understand. There are very few cases where you should
  use the passive voice for technical documentation.

  - Active voice - the subject performs the action denoted by the verb.

    - "The operating system runs a process." This sentence answers the question on what is
      happening and who/what is performing the action.

  - Passive voice - the subject is no longer _active_, but is, instead, being acted upon by the
    verb - or passive.

    - "A process is being run." This sentence is unclear about who or what is running the process.
      You might consider "a process is run by the operating system", but the object of the action
      is still made into the subject of the sentence, which indicates passive voice. Passive voice
      tends to be wordier than active voice, which can make your sentence unclear.

- **Do not list future plans for a product/feature.** "In the future, the product will have no
  bugs." This leads to the question as to when this would happen, but most importantly this is not
  something that anyone can guarantee will actually happen.

- **Do not talk about how certain features work behind the covers unless it is absolutely necessary.**
  Always ask yourself, "Is this text necessary to understand this concept or to get through these
  instructions?" This also leads to shorter (less maintenance) and more concise (happier readers)
  documentation.

- **Avoid using uncommon words, highly technical words, or jargon that users might not understand.**
  Also, avoid using idioms such as "that's the way the cookie crumbles". While it might make sense
  to you, it may not translate well into another language. Keep in mind that a lot of users are
  non-native English speakers.

- **Use compound words correctly.** Use the compound words that give the correct meaning.
  For example, "set up" (verb)  and "setup" (noun) have different meanings.

- **Avoid using words such as "best" or "great" since these are all relative terms.** How can you
  prove that "this operating system is the best?"

- **Avoid referencing proprietary information.** This can refer to any potential terminology or
  product names that may be trademarked or any internal information (API keys, machine names, etc…).

- **Avoid starting a sentence with "this" since it is unclear what "this" references.**

  - For example: "The operating system is fast and efficient. This is what makes it well designed."
    Does "this" refer to fast, efficient, or operating system? Consider using: "The operating system
    is well designed because it is fast and efficient."

- **Avoid ambiguous pronouns.** Pronouns are generally evil and not to be trusted. When you use 
  "it", "they", "them", etc., make sure it is 100% clear what the pronoun refers to. Always err on 
  the side of re-writing the noun. Example: "The cat crept down the stairs towards the mouse, and 
  it shrieked." The cat or the mouse?

- **Keep sentences fairly short and concrete.** Using punctuation allows your reader to follow
  instructions or concepts. If by the time you read the last word of your sentence, you can't
  remember how the sentence started, it is probably too long. Also, short sentences are much easier
  to translate correctly.

- **Know your audience.** It is good practice to know your audience before you write documentation.
  Your audience can be, for example, developers, end-users, integrators, etc. and they can have varying
  degrees of expertise and knowledge about a specific topic. Knowing your audience allows you to
  understand what information your audience should be familiar with. When a document is meant for a
  more advanced audience, it is best practice to state it up front and let the user know
  prerequisites before reading your document.

- **Adverbs are evil.** Using an adverb is almost always a sign that you haven't done enough work 
to set the correct context. You should try using a different verb or setting more context.

- **Do not use exclamation marks.** Some newspaper editors only let their writers use one 
exclamation mark _per year._ Just remove it, and you will see that the sentence is better.

- **Use markdown.** You must create documentation in markdown (.md) and keep the markdown file
  wrapped to a 100 character column size.

- **Be respectful**