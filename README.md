# **PJM 5-Bus Optimal Power Flow: ED, DC-OPF and AC-OPF**

Economic dispatch (ED), DC optimal power flow (DC-OPF) and AC optimal power flow (AC-OPF) on the PJM 5-bus test case, built in Python with Pyomo. The project shows what transmission congestion costs, how it changes the generation dispatch, and how it produces different electricity prices (locational marginal prices, LMPs) at different buses, first with a linear lossless model and then with full AC physics. Every Pyomo result is cross-checked against an independent PYPOWER solution.

## **Key results**

![][image1]

*Figure 1\. DC-OPF on the PJM 5-bus network. Bus colours and labels show locational marginal prices (LMPs); line colours and thickness indicate loading relative to thermal limits; arrows show power-flow direction.*

![][image2]

*Figure 2\. AC-OPF on the same network. Bus colour and the first number are the active LMP, and the second is the voltage magnitude. Line colour and thickness show apparent-power loading against the MVA limit, and arrows show the direction of active power flow.*

**Table 1\. ED, DC-OPF and AC-OPF compared**

| Result | ED | DC-OPF | AC-OPF |
| ----- | ----- | ----- | ----- |
| Operating cost | \$14,810/h | \$17,479.90/h | \$17,551.89/h |
| Increase over ED | n/a | \+\$2,669.90/h (+18.0%) | \+\$2,741.89/h (+18.5%) |
| Network losses | not modelled | neglected | 5.19 MW |
| Binding line | none (no network) | Branch 6 (240 MW), 100% loaded | Branch 6 (240 MVA), 100% loaded |
| LMP range (\$/MWh) | 30.00 (one price) | 10.00 to 39.94 | 10.00 to 39.71 |
| Branch 6 shadow price | n/a | \$62.32/h per MW | \$61.31/h per MVA |

**Table 2\. Active-power LMPs by bus (\$/MWh)**

| Model | Bus 1 | Bus 2 | Bus 3 | Bus 4 | Bus 5 |
| ----- | ----- | ----- | ----- | ----- | ----- |
| ED | 30.00 | 30.00 | 30.00 | 30.00 | 30.00 |
| DC-OPF | 16.98 | 26.38 | 30.00 | 39.94 | 10.00 |
| AC-OPF | 16.94 | 26.55 | 30.00 | 39.71 | 10.00 |

![][image3]

*Figure 3\. ED, DC-OPF and AC-OPF compared: (a) generator dispatch, (b) nodal prices, (c) line loading, with Branch 6 at its limit, and (d) AC-OPF reactive output against its limits.*

### **Key findings**

* **Congestion raises operating cost and splits the price.** One binding line (Branch 6\) forces about 133 MW of cheap Gen 5 output to be replaced by more expensive Gen 3\. Cost rises by about 18% over ED, and the single ED price of \$30/MWh becomes nodal prices from \$10 to about \$40/MWh.  
* **Transmission capacity has a marginal value.** The dual of the binding line gives the saving from one more unit of capacity: \$62.32/h per MW (DC-OPF) and \$61.31/h per MVA (AC-OPF). These are local sensitivities at the current operating point.  
* **AC physics is a small correction here.** AC-OPF costs \$72.00/h (0.4%) more than DC-OPF, mostly to cover 5.19 MW of losses. The same line binds and LMPs differ from DC-OPF by at most \$0.23/MWh.  
* **AC-OPF adds information DC-OPF cannot.** Three generators reach their reactive limits, Bus 3 sits at its 1.10 p.u. voltage bound, reactive LMPs are positive at Buses 1 to 3, and reactive flow raises line loading (Branch 3 is at 66.7% in AC-OPF versus 53.2% in DC-OPF).

## **Approach**

| Model | Problem type | Tools | Outputs |
| ----- | ----- | ----- | ----- |
| ED | Linear program | Pyomo \+ HiGHS | Dispatch, system price |
| DC-OPF | Linear program | Pyomo \+ HiGHS | Dispatch, flows, LMPs, line shadow price |
| AC-OPF | Non-convex nonlinear program | Pyomo \+ IPOPT (flat start) | Dispatch, voltages, losses, active and reactive LMPs, line shadow price |

**Validation.** DC-OPF and AC-OPF are solved again with PYPOWER, and the dispatch, cost, LMPs and shadow prices agree with the Pyomo results. The ED solution is also derived by hand with Lagrange multipliers and the KKT conditions in the [Optimization Foundations note](https://vesuvius-khatiwada.super.site/optimization-foundations-for-energy-system-modelling), which gives the same system price ($30/MWh)$ and cost (14,810/h) as the solver.

## **What is in this repo**

```
pjm5-opf/
├── README.md
├── requirements.txt
├── notebooks/                            # four analysis and reference notebooks
└── images/
```

Suggested reading order: `01` then `03`. The two PYPOWER notebooks are the reference solutions. The Pyomo notebooks contain their reference values, and the PYPOWER notebooks regenerate them if you change the case data.

## **Theory**

The notebooks cover implementation and results. The derivations live in two notes on my website:

* [ED, DC-OPF, and AC-OPF](https://vesuvius-khatiwada.super.site/ed-dc-opf-and-ac-opf): the three formulations, congestion, and how to read duals as LMPs and line shadow prices.  
* [Optimization Foundations for Energy System Modelling](https://vesuvius-khatiwada.super.site/optimization-foundations-for-energy-system-modelling): convexity, Lagrangians, KKT conditions, and a hand-worked economic dispatch.

The full project write-up, with figures, is on the [project page](https://vesuvius-khatiwada.super.site/economic-dispatch-dc-opf-and-ac-opf-on-the-pjm-5-bus-network).

## **How to run**

```sh
git clone https://github.com/vjkhtw/pjm5-opf.git
cd pjm5-opf
pip install -r requirements.txt
idaes get-extensions        # downloads IPOPT, needed for the AC-OPF notebook
jupyter notebook notebooks/01_dcopf_pyomo.ipynb
```

The DC-OPF notebooks only need HiGHS (installed with `highspy`). The AC-OPF notebook needs IPOPT, which `idaes get-extensions` provides.

## **Limitations**

This is a single-period model with linear generation costs and base-case line limits. There are no N-1 contingencies, reserves, unit commitment or bid-based offers, so the prices are simplified day-ahead-style LMPs, not PJM's actual prices. The DC-OPF model is lossless and ignores reactive power. AC-OPF is non-convex, so IPOPT returns a local optimum from a flat start; it matches the PYPOWER solution here, but that does not prove global optimality.

## **Data**

[IEEE PES Power Grid Library](https://github.com/power-grid-lib/pglib-opf), `pglib_opf_case5_pjm` (v23.07), based on F. Li and R. Bo, "Small Test Systems for Power System Economic Studies," IEEE PES General Meeting, 2010\. Licensed CC BY 4.0.

## **Learning resources**

See the reading list at the end of the [ED, DC-OPF, and AC-OPF](https://vesuvius-khatiwada.super.site/ed-dc-opf-and-ac-opf) note.

## **Author**

Vesuvius Khatiwada, [portfolio](https://vesuvius-khatiwada.super.site/)




[image1]: images/dc_opf_network.png
[image2]: images/ac_opf_network.png
[image3]: images/comparison.png
