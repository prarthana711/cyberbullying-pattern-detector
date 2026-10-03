# Cyberbullying Pattern Detector — Label Specification

## Overview

This project detects harassment as a trajectory across a conversation
rather than treating each message as an isolated toxic sentence.

The task is a multi-label classification problem. A single thread may
contain one or more of the defined patterns.

The four labels are:

1. `escalating_hostility`
2. `repeated_targeting`
3. `social_exclusion`
4. `pile_on`

---

## 1. escalating_hostility

### Definition

Hostile or insulting behavior becomes progressively stronger across
consecutive messages in a conversation.

The important characteristic is escalation across the conversation,
rather than the presence of one isolated harsh message.

### Must include

A thread should receive this label when:

- hostility increases across multiple messages;
- insults become progressively stronger;
- the conversation moves from mild hostility toward stronger personal
  attacks, contempt, or threats;
- there is observable progression rather than only a single hostile
  statement.

### Must not include

Do not assign this label when:

- there is only one harsh or insulting message;
- participants disagree strongly but hostility does not increase;
- the conversation contains friendly teasing or mutual joking;
- hostility decreases or the conversation ends with a clear
  de-escalation or apology.

### Positive example 1

A: I don't think your answer is correct.

B: You're wrong about this.

A: You clearly didn't understand the question.

B: You're completely useless.

### Positive example 2

A: That's not a very good argument.

B: Your argument makes no sense.

A: You never understand anything.

B: Just stop talking.

### Negative example 1

A: I don't agree with your answer.

B: That's okay. I see it differently.

A: Fair enough.

### Negative example 2

A: That's a terrible answer.

B: I was frustrated when I wrote that.

A: Okay, let's discuss the actual problem.

---

## 2. repeated_targeting

### Definition

The same person is the target of hostile, insulting, humiliating,
threatening, or exclusionary messages across two or more messages
within a conversation.

The target may be attacked by one author or by multiple authors.

### Must include

A thread should receive this label when:

- the same person is targeted in at least two separate messages;
- the target can reasonably be identified from the conversation;
- the attacks may come from one or several authors.

### Must not include

Do not assign this label when:

- there is only one attack against a person;
- different people are attacked without a common target;
- the conversation contains criticism of an idea rather than a person;
- the supposed target cannot reasonably be identified.

### Positive example 1

A: Rahul's answer is terrible.

B: Rahul clearly doesn't understand the topic.

A: Rahul should stop answering these questions.

### Positive example 2

A: Why does Priya keep posting this?

B: Priya never contributes anything useful.

C: Someone should tell Priya to stop.

### Negative example 1

A: Rahul's answer is wrong.

B: Priya's answer is also incorrect.

C: I disagree with both answers.

### Negative example 2

A: Rahul made one mistake in his answer.

B: Yes, but the rest of his explanation is reasonable.

C: Let's focus on correcting that mistake.

---

## 3. social_exclusion

### Definition

Messages attempt to isolate, silence, reject, remove, or discourage
a person from participating in a conversation or social group.

### Must include

Examples of exclusionary behavior include:

- telling someone to leave;
- telling others to ignore someone;
- saying that nobody wants someone in the group;
- telling someone they should not participate;
- attempting to silence or socially isolate the target.

### Must not include

Do not assign this label when:

- someone politely asks to end a discussion;
- a moderator gives a legitimate moderation instruction;
- someone voluntarily leaves a conversation;
- participants simply disagree with someone's idea;
- someone asks another participant to stop a specific behavior without
  attempting to socially exclude them.

### Positive example 1

A: Why is she still here?

B: Nobody wants her in this chat.

C: Just ignore everything she says.

### Positive example 2

A: You shouldn't be part of this group.

B: Nobody wants you here.

C: Leave and don't come back.

### Negative example 1

A: I think we've discussed this enough.

B: Okay, I'll stop.

### Negative example 2

A: Please stop posting unrelated messages.

B: Sorry, I'll stay on topic.

---

## 4. pile_on

### Definition

Three or more different authors direct hostile, insulting,
humiliating, threatening, or exclusionary messages toward the same
target within a conversation.

The defining characteristic is coordinated or converging hostility
from multiple authors toward one target.

### Must include

A thread should receive this label when:

- at least three different authors participate in the hostility;
- the authors target the same person;
- the messages are hostile, insulting, humiliating, threatening,
  or exclusionary.

### Must not include

Do not assign this label when:

- there are fewer than three attacking authors;
- multiple people criticize an idea without attacking a person;
- one author sends several hostile messages;
- three people discuss a person without directing hostility toward them.

### Positive example 1

A: Rahul's answer is ridiculous.

B: I agree, Rahul clearly has no idea what he's talking about.

C: Rahul should stop pretending he understands this.

D: Someone needs to tell Rahul to leave.

### Positive example 2

A: Priya is making this discussion worse.

B: She always does this.

C: Nobody wants her here.

### Negative example 1

A: Rahul's answer is wrong.

B: I disagree with Rahul too.

C: I think the calculation needs to be corrected.

### Negative example 2

A: Rahul is wrong.

A: Rahul doesn't understand this.

A: Rahul should read the question again.

---

# Hard Negatives

Hard negatives are conversations that may contain words, tones, or
behaviors that look hostile when individual messages are examined,
but should not be classified as cyberbullying patterns based on the
conversation context.

## 1. Friendly roasting

Friends may use insults or teasing toward each other without
harassment.

Example:

A: You're such an idiot 😂

B: Look who's talking 😂

A: Fair enough.

This should not automatically be classified as harassment.

---

## 2. Heated argument that de-escalates

A conversation may become heated but subsequently calm down,
resolve the disagreement, or contain an apology.

Example:

A: That's a ridiculous argument.

B: Don't talk to me like that.

A: You're right. Sorry.

B: It's okay. Let's continue.

The presence of hostility alone should not be sufficient if the
overall interaction does not form the defined pattern.

---

## 3. Quoting an insult

A person may quote an offensive statement in order to report,
describe, or criticize it.

Example:

A: He called me "worthless" yesterday.

The quoted insult should not automatically be treated as an attack
by the current speaker.

---

## 4. Strong disagreement about ideas

People can strongly disagree with an argument, opinion, or idea
without attacking the person.

Example:

A: Your proposed solution is completely wrong.

B: I disagree. The second approach handles the edge case better.

This is disagreement about an idea, not necessarily harassment.

---

## 5. Reclaimed or in-group language

Some words may have different meanings depending on the relationship,
community, or context in which they are used.

A model should not rely only on individual offensive terms without
considering conversation context.

---

# Annotation Granularity

## Thread-level labels

The primary annotation unit is the complete conversation thread.

A thread may receive multiple labels.

For example:

```json
{
  "labels": [
    "escalating_hostility",
    "repeated_targeting",
    "social_exclusion"
  ]
}