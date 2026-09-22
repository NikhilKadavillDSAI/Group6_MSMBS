# Assignment 3 — Boolean Networks: Cancer-Causing Mutations

This assignment studies how different mutations affect a Boolean network that models cancer-related cell behaviour. The network has 8 nodes: DNA damage, p53, MYC, CDK2, MDM2, p21, Growth, and Death. All 256 possible starting states were tested for each network.

A state is called **cancer-like** when:

* DNA damage = 1
* Growth = 1
* Death = 0

This means the cell keeps growing even though its DNA is damaged and it does not die.

## Mutations

| Mutation | Change               | Meaning                 |
| -------- | -------------------- | ----------------------- |
| Normal   | No change            | Normal network          |
| A        | p53 always OFF       | p53 knockout            |
| B        | MYC always ON        | MYC overexpression      |
| C        | MDM2 always ON       | MDM2 overexpression     |
| D        | DNA damage always ON | Chronic DNA damage      |
| E        | Growth always ON     | Permanent growth signal |

Mutations D and E were added to test chronic DNA damage and permanently active growth.

## Results

### Scenario analysis

For a healthy cell, all networks generally show normal growth without death.

For a stressed cell with DNA damage:

* **Normal:** the cell stops growing and dies.
* **A, B, C:** the cell keeps growing instead of dying.
* **D:** the cell responds normally and dies.
* **E:** the cell keeps growing because Growth is forced ON.

### Cancer-like states

| Network                 | Cancer-like states | Percentage |
| ----------------------- | -----------------: | ---------: |
| Normal                  |            8 / 256 |      3.12% |
| A — p53 knockout        |          128 / 256 |     50.00% |
| B — MYC amplification   |          128 / 256 |     50.00% |
| C — MDM2 overexpression |          128 / 256 |     50.00% |
| D — Chronic DNA damage  |           24 / 256 |      9.38% |
| E — Growth locked ON    |          128 / 256 |     50.00% |

Mutations A, B, C and E increase the cancer-like states from **3.12% to 50%**. Mutation D increases them to **9.38%**.

## Q1: Which mutation is most dangerous?

A, B, C and E all produce a **50% cancer-like basin** in this model, compared with 3.12% for the normal network.

A, B and C disrupt the p53 control system, while E directly forces Growth ON. D has a smaller effect, with 9.38% cancer-like states.

## Q2: Role of the feedback loop

The network contains a feedback loop:

**p53 → MYC → MDM2 → p53**

p53 reduces MYC, MYC increases MDM2, and MDM2 reduces p53.

Mutations A, B and C break this control system in different ways but produce the same result: **50% cancer-like states**. Mutation E bypasses the loop by forcing Growth ON. Mutation D is outside the loop and has a smaller effect.

## Q3: Limitations

The model has three main limitations:

1. **Only 0 or 1:** It cannot show different levels of protein activity.
2. **Synchronous updates:** It assumes all biological processes happen at the same time.
3. **Very small network:** Real cancer pathways contain many more genes and interactions than these 8 nodes.

## Conclusion

The results show that disrupting **p53, MYC, MDM2, or Growth** can greatly increase cancer-like behaviour in this model. Chronic DNA damage has a smaller effect because the p53 control system can still respond to the damage.
