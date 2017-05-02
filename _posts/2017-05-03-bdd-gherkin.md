---
title: "Intro to BDD with Gherkin"
description: "Using the Gherkin language to get started with Behaviour-Driven Development"
tags: BDD, Gherkin
author: Peter Hilton
layout: article
---

At the [ACCU 2017](https://conference.accu.org/) conference last week, [Seb Rose](https://twitter.com/sebrose) presented an [Intro to TDD and BDD](https://conference.accu.org/site/stories/2017/sessions.html#XIntrotoTDDandBDD) workshop.
This blog summarises the key lessons I learned from this workshop.

## Video

The video of this 90-minute session is not yet available, at the time of writing, but will likely soon appear on the [ACCU Conference YouTube channel](https://www.youtube.com/channel/UCJhay24LTpO1s4bIZxuIqKw).

## Gherkin

We started the workshop with two examples of feature scenarios.
The first scenario described an end-to-end cash withdrawal:

```gherkin
Scenario: Dispense $50 cash in multiple denominations
  Given I have $100 in my account
    And I have a card with the PIN 5173
    And I push my card into the machine
    And I enter 5173 for my PIN
    And I push the button next to Withdrawal
    And I push the button next to Checking
  When I push the button next to $50
  Then a $20 bill should be ejected by the cash dispenser
    And a $20 bill should be ejected by the cash dispenser
    And a $10 bill should be ejected by the cash dispenser
```

The second version described the same scenario, but focused on the specific business rule about which denominations to use:

```gherkin
Scenario: Dispense $50 cash in multiple denominations
  When $50 is dispensed by the ATM
  Then the following bills should be received:
    | count | denomination |
    | 2     | $20          |
    | 1     | $10          |
```

Our first lesson, after attempting to describe pros and cons for each version of the scenario was that the second style is better.
The end-to-end scenario includes lots of other details that are best described in separate scenarios.

The second lesson was that even such a simple scenario lead to a lively group _conversation_ while we _explored_ the scenario.
It turns out that having a conversation about concrete examples is a key BDD technique.

## Behaviour-Driven Development (BDD)

… three amigos meeting

… benefit of this discussion before starting to write code

## Gerkin language

… we also learned that the scenario syntax, using given-when-then, is a language called Gherkin

## Gherkin tips



## Next steps

… test automation… Cucumeber

… it turns out that Seb co-wrote a book about Cucumber…

… API documentation

## Conclusion
