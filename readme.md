# Revenue Sharing Language (RSL)

Nothing written in this document is offered as legal or financial advice: use of these ideas and work is at your own risk. 

*v1.0 RC* 

Revenue Sharing Language (RSL) is a domain-specific Language (DSL) for describing multi-step revenue sharing agreements between multiple parties. It is proposed as a common vocabulary for such agreements that both humans and software can understand. It is designed to support unlimited steps and payees, loops, caps and either pro rata or pari passu type payouts at each step. 

RSL is not designed to implement or process payouts, but to provide the instructions for systems that do. It is also not designed to replace a recoupment contract, but to provide a machine-readable description of a payout structure, recoupment schedule or waterfall that could be appended to a contract, in a way that lawyer or judge could understand. RSL has been tested in implementations described at the bottom of this file.

Background: RSL is structured as a [YAML](https://yaml.org/) subset, which was chosen over JSON for easier legibility. It is inspired by Ian Grigg's idea of a [Ricardian Contract](https://en.wikipedia.org/wiki/Ricardian_contract), a machine- and human- readable agreement. A benefit of a machine readable contract is that a cryptographic [hash](https://en.wikipedia.org/wiki/SHA-1) can be generated from it, which would change if the agreement later changes, acting as a check against changes that haven't been agreed to by a contract's parties. This is somewhat similar to a [Smart Contract](https://en.wikipedia.org/wiki/Smart_contract), which also describes a machine-parsable legal agreement and was also first discussed in 1996 – with the important difference a human should also be able to read a Ricardian contract. 

## Syntax

An RSL agreement is made from three components:
 - **Header**: appears once for the whole *Agreement*. Only Name and Currency is required. It is made up of… 
 - **Step(s)**, which can appear unlimited times, and be fixed, percentage or ratio. Steps are paid out *sequentially*, one after the other. A step is made up of…
 - **Payee(s)**, who can appear unlimited times per step, and need only specify amount and an account reference. Payees are paid out *concurrently*, at the same time.

For e.g. in the example below, an author signs a deal for 25% of net sales (with 10% to her agent) after a £10,000 advance is paid back to her publisher.

```
---
name: "Best of times, Worst of times"
rsl version: 1.0
description: "Book deal for the new novel by Charlene Dickens"
currency: EUR
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

## Full syntax

### Header
- **Name**: 256 chars max - *REQUIRED*.
- **Currency**: (three letter [ISO currency](https://en.wikipedia.org/wiki/ISO_4217) or [crypto](https://www.finder.com.au/cryptocurrency/altcoins) code) - *REQUIRED*.
- **Decimals**: comma (,), period (.) or none. Determines if decimal places in currency ammounts are denoted with '.' or ',' or (as is the case with currencies such as Yen) no decimal places. If not set, period (.) is assumed.
- **RSL**: which RSL version is used (e.g. 0.6, 1.0). This allows for future changes in RSL syntax to not break on older RSL files.
- **Description**.
- **Pointer**: following the idea of [Payment Pointers](https://paymentpointers.org/), this references a wallet, bank account, collecting society or escrow account that will receive the initial funds this agreement relates to. 
- **URL**: A public address for the agreement. This should include the values for any variable items.
- **Contact**: Contact name for the agreement.
- **Email**: Email address for the contact.
- **Jurisdiction**: Legal jurisdiction for the agreement, ie which country the agreement is operated from / answerable to. Could be country, region or state.
- **Period**: When does the agreement reset? If not set, is assumed to run indefinitely, or until the end date. Takes form of '10 transactions', '1 year', '4 weeks', etc, made up of:
	- A positive integer;
	- An interval, either 'transaction' sich as per donation, sale or membership; or time period such as day, month, year, etc
- **Starts**: a starting date for the agreement (YYYY-MM-DD), e.g. 2021-12-31. This can be used with Term to match financial reporting periods.
- **Ends**: an end date, which allows for a fixed period agreement to stop. 
- **Steps**: An array of one or more steps, each including an array of one or more payees.

### Steps:
- **Type**:  fixed, percent or ratio. Ratios are an alternative to percent, helpful for splits with recurring decimals when expressed as a % - *REQUIRED*.
- **Description**:
- **Cap**: for percentage payouts, a max payout before moving to the next step. ‘null’, or unset indicates distribution runs indefinitely.
- **Payee**: an array for payees, 

### Payees
An unlabelled, inline array of:
- **name**;
- **pointer**: [Payment pointer](https://paymentpointers.org/), bank account/sortcode, IBAN, Swift, email, dbse ID, etc) - *REQUIRED*;
- **amount**: If fixed this is the fee, if percentage it is the mount (without a percentage symbol). Ratios are expressed only as the numerator (ie '1' for an equal 3-way split) - required.
- **type** (describes the nature of the pointer, e.g. ILP, AC/sort, Wise, Stripe, Paypal, Ref, ID)

#### NB: The difference between fixed (pari passu) and %/ratio (pro rata) payouts:

Payees in each step are paid at the same time but these can be handled in two different ways. If the step is percentage or ratio, each payout is paid out proportionately, or pro rata, until the cap is met. If fixed, each payee is paid an equal amount, or pari passu, with each payout, until all are paid what's owed. Fixed steps mean some payees can be fully repaid before others.

| payee | % | ratio | fixed |
|-------|---|-------|-------|
|A - owed €5 | 33.33 | 1 | €5 |
|B - owed €10 | 66.66 | 2 | €10 |
|cap | €15 | €15 | - |

If only €6 is received, in the percentage or ratio step, A gets €2 and B gets €4. With a fixed step, both would get €3. 
If only €12 was received, the ratio/% payout would give A €4 and B €8, and the fixed payout would give A €5 and B €7.

### Variable Syntax - proposed specification

Variable Syntax is a proposed extension of RSL to use {{double-curly bracketed values}} in a contract to reference values that can be assigned after a revenue sharing agreement is agreed and hashed. It is not yet supported in any of the implementions described below, but would allow for:

1. payouts to be defined after a contract is agreed, for instance expenses that aren't known in advance. E.g. Marketing costs up to 1000 can be deducted in in the step below:

```
steps:
  -
    type: fixed  
    cap: 1000
    payee: 
      - [“Marketing costs”, 12345678 / 12-34-56, Wise, {{marketingCosts}} ]
```

2. payees to be defined after a contract is agreed, for instance cooperative members, resellers or distributors. E.g. below 40% of income is split equally between all investors, and 60% between all workers, regardless how many new investors and workers join after the contract is first agreed.

```
steps:
  -
    type: %  
    payee: 
      - [{{investor.name}}, {{investor.pointer}}, {{investor.type}}, 40 / N ]
      - [{{worker.name}}, {{worker.pointer}}, {{worker.type}}, 60 / N ]
```
 
These variables can then be defined through a URL specified in the header as a JSON object, ie:

```
{
  "marketingCosts": "850",
  "worker": [
    {
      "name": "Flik",
      "pointer": "12345678",
      "type": "IBAN"
    },
    {
      "name": "Atta",
      "pointer": "name@example.org",
      "type": "paypal"
    },
  ],
  "investor": [
    {
      "name": "PT Flea",
      "pointer": "ilp.example.org/123456",
      "type": "ILP"
      }
   ]
}
```

## Examples

### Example 1: record publishing deal
After paying their record label and producer, band members split income from their album up to $1m equally, and give the remainder to charity.

```
---  
name: “Abi's Road”  
rsl: 1.0
description: “Record deal for the band Be At Less”
url: beatless.example.com/deal
pointer: $ilp.example.com/pN3K3rKULNQh
contact: Big Lawyers Inc.
email: big.lawyers@example.com
currency: EUR
decimals: comma
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

### Example 2: distribution agreeement
A filmmaker allows anyone embedding their film to take 30% of Web Monetization, donations and pay-per-view fees, after the cost of video streaming and a carbon footprint offset, which are split 3:4 up to 50 cents. 

```
---  
name: “Film distribution share”
rsl: 1.0
description: “Profit share for video embeds”
url: deal.example.com/name
currency: USD
period: 1 transaction
jurisdiction: California, USA
steps:
  -
    type: ratio
    cap: 0.40
    payee:
      - [ "Streaming costs", $fee.example.com, ilp, 3]
      - [ "Ocean forest restoration", $offset.example.com, ilp, 4]
  -
    type: percent
    endpoint: celiafilm.example.com
    payeeTemplate:
      - [ "Celia Bee Da'mil", $example.com/celia, ilp, 70]
      - [ "{{website.name}}", "{{website.pointer}}", {{website.type}}, 30]
```

### Example 3: film profit share
Following on from the above example; when the filmmaker receives income from websites embedding her film, she will first pay back her credit card; then deferred salaries & her expenses; then a profit share with the cast and crew and investors.

```
---  
name: “Film income share”
rsl: 1.0
description: “Profit share for feature film”
url: $deal.example.com/celia
currency: USD
contact: Celia Bee Da'mil
jurisdiction: California, USA
steps:
  - 
    type: fixed
    payee:
      - [ "Credit Card", 12346678 12345678, IBAN, 50000]
      - [ "Interest", 12346678 12345678, IBAN, "{{interest}}"]
  -
    type: fixed
    endpoint: deferals.example.com/unpaidsalary
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
      - [ {{investor.name}}, {{investor.pointer}}, {{investor.type}}, 25 / {{investor.share}} ]
```

### Example 4: filmmaker cooperative
A group of filmmakers have setup a film studio cooperative (with apologies to [United Artists](https://en.wikipedia.org/wiki/United_Artists)). They will take an annual founders bonus, then split the profits after expenses equally. It is worth noting that steps two and three are effectively the same, but with Step 3, the addition of Buster Keaton or Lilian Gish to the list of 'owners' at the specified endpoint would change the payout per owner (marked as N) from 25% to 20%.

```
---  
name: “Artists United”  
rsl: 1.0
description: “Founding agreement for film studio coop”
url: deal.example.com
pointer: $ilp.example.com/pN3K3rKULNQh
currency: USD
period: 1 year
steps:
  -
    type: fixed  
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
    payeeTemplate: 
      - [ {{owners.name}}, {{owners.pointer}}, {{owners.type}}, N ]
```

### Example 5: per sale redistribution
A charity helps developers give coding lessons in refugee camps around the world, and sells the apps and games created. Each student developer gets a fee per sale; what's left is split between the charity and all developers for that year.

NB, in this example, an additional 'this' term is added to the first step payee to indicate a single payee relvent to *this* specific transaction. In the second step, all developers are included.

```
---  
name: “Dev Camp”  
rsl: 1.0
description: “Profit share for trainee developers”
url: transactions.example.com/developers
currency: JPY
decimals: none
period: 1 transaction
steps:
  - 
    type: fixed
    payeeTemplate:
      - [ "{{this.developer.name}}", "{{this.developer.pointer}}", {{this.developer.type}}, 1000]
  -
    type: percent
    cap: null
    payeeTemplate:
      - [ "Charity", 12346678 12345678, IBAN, 50]
      - [ "{{developer.name}}", "{{developer.pointer}}", {{developer.type}}, 50 / N]
```

## Implementations

Cascade is an RSL builder:

* **[Cascade Native](https://github.com/openvideotech/cascade-native)** Minimal and framework-free, built by [Mark Boas](https://github.com/maboa). [Demo](https://openvideo.tech/cascade/).

* **[Cascade Svelte](https://github.com/openvideotech/cascade-svelte)** (standalone & civi), realtime responsive (ie RSL generates as you type), building on Native, compiled in Svelte, with unit tests, built by [Rich Lott](https://github.com/artfulrobot/). It compiles to two versions for CiviCRM and for standalone. [Demo](https://openvideo.tech/cascade-svelte/).

CiviSplit is an alpha level extension sutie for [CiviCRM](https://civicrm.org) (a comprehensive CRM for WordPress, Drupal, Backdrop & Joomla)

* **[CiviSplit Agreement Builder](https://lab.openvideo.tech/civisplit/agreement-builder)** - extension to create, save and generate income sharing agreements in with reporting.
    
* **[CiviSplit Processor](https://lab.openvideo.tech/civisplit/processor)** - extension to calculate and process amounts owed to different parties, handling incomplete payments.

* **CiviSplit Uphold** - payment processor integrating with Uphold.

--- 

**Copyright & Disclaimer**

Copyright © 2022 Nicol Wistreich under a [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/). With thanks to Rich Lott, Mark Boas, Aidan Saunders, Matthew Wire & Silvia Schmidt.

THIS WORK IS PROVIDED "AS IS," AND COPYRIGHT HOLDERS MAKE NO REPRESENTATIONS OR WARRANTIES, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO, WARRANTIES OF FITNESS FOR ANY PARTICULAR PURPOSE OR THAT THE USE OF THE SYSTEM OR DOCUMENT WILL NOT INFRINGE ANY THIRD PARTY PATENTS, COPYRIGHTS, TRADEMARKS OR OTHER RIGHTS.

THE WORK DOES NOT CONSTITUTE LEGAL OR ANY OTHER FORM OF ADVICE. COPYRIGHT HOLDERS WILL NOT BE LIABLE FOR ANY DIRECT, INDIRECT, SPECIAL OR CONSEQUENTIAL DAMAGES ARISING OUT OF ANY USE OF THE SYSTEM OR DOCUMENT.
