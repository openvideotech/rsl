# Revenue Sharing Language (RSL)

*Status: v0.5, draft specification, open for review.*

RSL is a proposed syntax for describing revenue sharing agreements between multiple parties using [YAML](https://yaml.org/). 

A standard Syntax is first proposed below with either fixed or percentage payout splits in a potentially unlimited series of steps. The payments in each step must be complete before moving onto the next step – which for percentage payouts can be based on a cap. 

Beneath that is an additional, less complete, Variable Syntax with {{double-bracketed pointers}} to reference values taken from other sources, such as an expense total or an interest rate.

### Why is this needed?

Revenue Sharing Modeling Language (RSL) is inspired by the idea of a [Ricardian Contract](https://en.wikipedia.org/wiki/Ricardian_contract), which is a machine- and human- readable agreement. A Ricardian Contract for revenue sharing has the advantages of potentially reducing human error, mispayments, delays or 'creative adjustments' when an agreement is paid out. RSL is proposed as a common set of instructions that can be generated and processed when creating multistep recoupment and profit-share agreements. 

A RSL agreement can be hashed, ensuring future changes can be spotted and that errors/mispayments are limited to the software implementing the agreement. Where this software is operated by a party independent of the agreement, it could give further confidence of accurate payouts.
:::

## RSL Standard Syntax

An RSL agreement features three components:
 - Data for the whole **Agreement**, which appears once, and of which only Name and Currency is required.
 - Data for a **Step** in the agreement which can appear multiple times, and of which only Type (fixed or percentage) is required. A step will contain one or more Payees.
 - Data for a **Payee**, who can  appear multiple times per step, and of which only wallet is required.


### Data for the agreement
- **Name**: 256 chars max - *REQUIRED*.
- **Currency**: (three letter [ISO currency](https://en.wikipedia.org/wiki/ISO_4217) or [crypto](https://www.finder.com.au/cryptocurrency/altcoins) code) - *REQUIRED*.
- **Description**.
- **Period**: When does the agreement reset? If not set is assumed to run indefinitely, or until the end date. Takes form of '10 transactions', '1 year', '4 weeks', etc, made up of:
	- A positive integer;
    - An interval, either 'transaction' sich as per donation, sale or membership; or time period such as day, month, year, etc
- **Starts**: a starting date for the agreement (YYYY-MM-DD), e.g. 2021-12-31. This can be used with Term to match financial reporting periods.
- **Ends**: an end date, which allows for a fixed period agreement to stop. 
- **Pointer**: following the idea of [Payment Pointers](https://paymentpointers.org/), this references a wallet, bank account, collecting society or escrow account that will receive the initial funds this agreement relates to. 
- **URL**: A public address for the agreement.
- **Contact**: Contact name for the agreement.
- **Email**: Email address for the contact.
- **Steps**: An array of steps

### Data for a step in the agreement, numbered sequentially:
- **Type**:  fixed or percent - *REQUIRED*.
- **Description**:
- **Cap**: for percentage payouts, a max payout before moving to the next step. ‘null’, or unset indicates distribution runs indefinitely.
- **Payee**: an array for payees, containing:
	- **name**;
	- **pointer**: [Payment pointer](https://paymentpointers.org/), bank account/sortcode, IBAN, Swift, email, dbse ID, etc) - *REQUIRED*;
	- **amount**. If fixed this is the fee, if percentage this is the share - required.
	- **type** (describes the nature of the pointer, e.g. ILP, AC/sort, Wise, Stripe, Paypal, Ref, ID)

### Notes:
- payees in each step are paid at the same time, ['pari passu'](https://en.wikipedia.org/wiki/Pari_passu) until all are paid.
- if payees are in a percentage step but don’t add up to 100, the processor must transform them into a % of their sum.

### Example 1
An author signs a deal for 25% of net sales (with 10% to her agent) after a £10,000 advance is paid back to the publisher. To return the advance, the first step (step1) pays only the publisher until the 25% net figure is reached, ie when the publisher has earned £40,000.

```
---
name: "Best of times, Worst of times"
description: "Book deal for the new novel by Charlene Dickens"
url: example.com/248mc0u3489mc
currency: GBP
steps:
  -
    type: fixed
    payee:
      - ["PRNG House", $ilp.example.com/prng, ilp, 40000]
  -
    type: percent
    payee: 
      - ["Charlene Dickens", 12345678/12-34-56, uk-ac, 22.5 ]
      - ["Internet Aritsts Management", payment@example.com, paypal, 2.5]
      - ["PRNG House", $ilp.example.com, ilp, dbse, 75 ]
```

### Example 2
After paying their record label and producer, band members split income from their album up to $1m equally, and give the remainder to charity.

```
---  
name: “Abi's Road”  
description: “Record deal for the band Be At Less”
url: beatless.example.com/deal
pointer: $ilp.example.com/pN3K3rKULNQh
contact: Big Lawyers Inc.
email: big.lawyers@example.com
currency: EUR
steps:
  -
    type: fixed  
    payee: 
      - [“Orange Records”, ID001, dbse, 5000 ]
      - ["Georgina Martina", producer@example.com, paypal, 5000 ]
  -
    type: percent
    cap: 1000000  
    payee: 
      - [ "Joan", $ilp.example.com/joan, ilp, 25 ]  
      - [ "Paula", 12345678/12-34-56, uk-ac, 25 ]  
      - [ "Georgie", $ilp.example.com/georgie, ilp, 25 ]  
      - [ "Reena", NO 93 8601 1117947, iban, 25 ]
  -
    type: percent
    cap: null
    payee: 
      - [ "Unicef", $ilp.example.org/unicef, ilp, 100 ]
```

## RSL Variable Syntax 

Variable Syntax uses {{double -curly bracketed values}} in a contract, which will change after a revenue share agreement is first created. For instance annual expenses which must be deducted before profits are shared; or cooperative members when new members are joining all the time. 

As well as allowing for {{double bracketed}} reference points which can be changed after a contract has been generated, this would require further lines to be supported in the YAML:

### Additional variables for a step in the agreement:
- *Endpoint*: A url for the API to resolve the variables used in that step.
- *Key*: If the API endpoint requires a key to access the values.
- payeeTemplate: a generic array for that can resolve as new payees are added. An additional value is used to describe the list containing:
	- {{name_of_group.name}} (e.g. {{worker_owners.name}}, {{user_owners.name}}, {{investors.name}} - would each point to a different lists of payees)
	- {{name_of_group.pointer}}
	- {{name_of_group.payout}} or just N for an equally split percentage share.
	- {{name_of_group.type}}

### Example 3
A filmmaker allows anyone embedding their film to take 20% of Web Monetization, donations and pay-per-view fees, after the cost of video streaming and a carbon footprint offset. 

```
---  
name: “Film distribution share”  
description: “Profit share for video embeds”
url: ilp.example.com/name
currency: USD
period: 1 transaction
steps:
  -
    type: percent
    cap: 0.30
    payee:
      - [ "Streaming costs", $fee.example.com, ilp, 50]
      - [ "Ocean forest restoration", $offset.example.com, ilp, 50]
  -
    type: percent
    endpoint: celiafilm.example.com
    payeeTemplate:
      - [ "Celia Bee Da'mil", $example.com/celia, ilp, 80]
      - [ "{{website}}", "{{pointer}}", ilp, 20]
```

### Example 4
Following on from the above example; when the filmmaker receives income from websites embedding her film, she will first pay back her credit card; then deferred salaries & her expenses; then a profit share with the cast and crew and investors.

```
---  
name: “Film income share”  
description: “Profit share for feature film”
url: $example.com/celia
currency: USD
contact: Celia Bee Da'mil
steps:
  - 
    type: fixed
    endpoint: bank.example.com/interest
    payee:
      - [ "Credit Card", $bank.example.com, ilp, 50000]
      - [ "Interest", $bank.example.com,  ilp, "{{interest}}"]
  -
    type: fixed
    endpoint: deferals.example.com/unpaidsalary
    key: 10923m10293mix
    payeeTemplate: 
      - [ "{{deferal.name}}", "{{deferal.pointer}}", "{{deferal.type}}", "{{deferal.amount}}"]
  -
    type: percent
    payee:
      - [ "Lead actor", $ilp.example.com/uche, ilp, 10]
      - [ "Writer", $ilp.example.com/jalāl, ilp, 10]
      - [ "Director", $ilp.example.com/akira, ilp, 10]
      - [ "Composer", $ilp.example.com/clara, ilp, 10]
      - [ "Cinematographer", $ilp.example.com/sven, ilp, 10]
      - [ "Producer", $ilp.example.com/celia-personal, ilp, 25]
      - [ "Investor", $ilp.example.com/investor, ilp, 25]
```

### Example 5
A group of filmmakers have setup a film studio cooperative  (with apologies to [United Artists](https://en.wikipedia.org/wiki/United_Artists)). They will take an annual founders bonus, then split the profits after expenses equally. It is worth noting that steps two and three are effectively the same, but with Step 3, the addition of Buster Keaton or Lilian Gish to the list of 'owners' at the specified endpoint would change the payout per owner (marked as N) from 25% to 20%.

```
---  
name: “Artists United”  
description: “Founding agreement for film studio coop”
url: deal.example.com
pointer: $ilp.example.com/pN3K3rKULNQh
currency: USD
period: 1 year
steps:
  -
    type: fixed  
    endpoint: au.example.com/costs
    key: 203984c20934m0c29348
    payee: 
      - [ "Annual expenses", ID002, dbse, "{{costs}}" ]
  -
    type: percent
    cap: 1000000
    description: bonus for the studio founders
    payee: 
      - [ "Chaplin", $payee.example.com/charles, ilp, 25]
      - [ "Pickford", $payee.example.com/mary, ilp, 25]
      - [ "Griffith", $payee.example.com/melanie, ilp, 25]
      - [ "Fairbanks", $payee.example.com/douglass, ilp, 25]
  -
    type: percent
    endpoint: au.example.com/owners
    payeeTemplate: 
      - [ {{owners.name}}, {{owners.pointer}}, ilp, N ]
```

### Example 6
A charity helps developers give coding lessons in refugee camps around the world, and sells the apps and games created. Each student developer gets a fee per sale; what's left is split between the charity and all developers for that year.

```
---  
name: “Dev Camp”  
description: “Profit share for trainee developers”
currency: INR
period: 1 transaction
steps:
  - 
    type: fixed
    endpoint: transactions.example.com/developers
    payeeTemplate:
      - [ "{{developer.name}}", "{{pointer}}", ilp, 1000]
  -
    type: percent
    cap: null
    endpoint: devcamp.example.org/all_developers
    payeeTemplate:
      - [ "Charity", $ilp.example.org/devcamp, ilp, 50]
      - [ "{{all_developers.name}}", "{{all_developers.name}}", ilp, N]
```

--- 

**Copyright & Disclaimer**

Copyright © 2021 Netribution Ltd, Nicol Wistreich under a [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/). With thanks to Rich Lott, Aidan Saunders & Silvia Schmidt.

THIS WORK IS PROVIDED "AS IS," AND COPYRIGHT HOLDERS MAKE NO REPRESENTATIONS OR WARRANTIES, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO, WARRANTIES OF FITNESS FOR ANY PARTICULAR PURPOSE OR THAT THE USE OF THE SYSTEM OR DOCUMENT WILL NOT INFRINGE ANY THIRD PARTY PATENTS, COPYRIGHTS, TRADEMARKS OR OTHER RIGHTS.

THE WORK DOES NOT CONSTITUTE LEGAL OR ANY OTHER FORM OF ADVICE. COPYRIGHT HOLDERS WILL NOT BE LIABLE FOR ANY DIRECT, INDIRECT, SPECIAL OR CONSEQUENTIAL DAMAGES ARISING OUT OF ANY USE OF THE SYSTEM OR DOCUMENT.
