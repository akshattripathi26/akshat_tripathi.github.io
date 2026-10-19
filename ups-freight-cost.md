---
layout: default
title: UPS Parcel Freight Cost Analysis
---

[← Back to portfolio](../)

# UPS Parcel Freight Cost Analysis

**Where does the shipping money go, and where are the savings? A descriptive analysis of a year of UPS invoices.**

| | |
|---|---|
| **Domain** | Supply chain analytics · Cost engineering · Descriptive analytics |
| **Data** | UPS Parcel Dataset 2025: shipment-level freight invoices with 70+ fields (charges, surcharges, weights, zones, services, dates) covering a domestic lane (Plymouth, MI → US) and an international import lane (Belgium → Australia / UK) |
| **Tools** | Python, pandas, NumPy, matplotlib, seaborn, Google Colab |
| **Context** | Rice University course project |
| **Code** | [View the notebook](https://github.com/akshattripathi26/akshat_tripathi.github.io/blob/main/notebooks/ups_freight_cost_analysis.ipynb) |

## Results at a glance

| Top 2 services | Surcharges & fees | Far zone vs near zone | Spend concentration |
|:---:|:---:|:---:|:---:|
| **≈95% of spend** | **≈25% of spend** | **≈2.6× cost per lb** | **A small set of addresses = 80% of domestic spend** |

---

## The business problem

Parcel spend usually shows up as one large invoice line, and nobody can say what is driving it. The goal was to break the bill down into **cost drivers a procurement or logistics manager can act on**: which services, which surcharges, which destinations, and which shipping habits.

## Data preparation

- Loaded the `Freight-Detail` sheet, skipping its decorative header rows.
- Dropped columns that were 100% empty in this extract (delivery date, days in transit, on-time flag), converted money, weight, and dimension fields from text to numbers, parsed dates, and removed duplicate rows.
- Engineered business metrics:
  - **Lane:** domestic vs international
  - **Accessorials:** net charge − freight − fuel
  - **Fuel %:** fuel surcharge as a share of base freight
  - **Cost per lb:** net charge ÷ chargeable weight
  - **Invoice lag:** days from ship date to invoice date
  - **Residential-fee flag** and calendar breakdowns
- **Data-quality decisions:** an on-time (service-level) analysis wasn't possible because those fields were empty. A FedEx comparison column was about 10× lower than the UPS charge and clearly not a like-for-like rate, so I excluded it rather than drawing a misleading comparison.

## Analysis performed

Twelve views, each ending in a quantified result:

1. Monthly spend trend by lane
2. Spend by mode of transport and service
3. Bill composition: base freight vs fuel vs accessorials
4. Fuel surcharge rate over time
5. Domestic cost per lb by UPS zone
6. Cost distribution per shipment by service (outliers)
7. Destination concentration (Pareto curve)
8. Top destination states and residential-fee exposure
9. Weekday × month shipping heatmap
10. Same-day, same-address consolidation opportunity
11. International landed cost per shipment (freight + duties/fees)
12. Billing lag from ship date to invoice

## Findings and recommendations

| # | Theme | Finding | Recommendation |
|---|---|---|---|
| 1 | Spend concentration | Two services account for ≈95% of spend | Negotiate rates on these two services first |
| 2 | Surcharges | Fuel and accessorials are ≈¼ of spend, and the fuel rate rose through the year | Push for a fuel-surcharge cap and accessorial waivers in the contract |
| 3 | Distance | Zone 108 costs ≈2.6× more per lb than zone 102 | Ship far zones from a closer origin, or use ground service when time allows |
| 4 | Customers | A small set of addresses drives 80% of domestic spend | Set up account-level shipping agreements with those customers |
| 5 | Consolidation | Repeat Next Day Air boxes go to the same address on the same day | Batch orders per customer per day (the savings estimate is an upper bound) |
| 6 | Imports | Australia is the single most expensive lane once duties and fees are included | Compare Worldwide Saver against deferred or ocean options |

## What I'd do next

- Bring in delivery dates to measure on-time performance against cost, so savings don't come at the expense of service.
- Add automated outlier detection for unexpectedly high charges, to support invoice auditing.
- Model the net savings of consolidation after accounting for heavier combined boxes.

[← Back to portfolio](../)
