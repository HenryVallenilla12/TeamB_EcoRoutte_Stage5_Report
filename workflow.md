# 2. Workflow & Draft Report

*Output of the system prompt in `methodology.md` (Chain-of-Thought block removed).*

## 2.1 Overview

EcoRoute guides a user from an origin to a destination along the route with the lowest estimated emissions. The workflow covers route request, emission calculation, ranking, user confirmation, navigation and the logging of trip emissions, including the error and cancellation paths that end the flow early.

## 2.2 Workflow steps

| ID | Symbol | Step | Description | Next step |
|---|---|---|---|---|
| T1 | Terminator | Start | Entry point of the workflow. | IO1 |
| IO1 | Data I/O | Enter origin and destination | User provides the trip origin and destination. | A1 |
| A1 | Action | Fetch route options | System retrieves candidate routes. Data source: INSUFFICIENT_DATA. | A2 |
| A2 | Action | Calculate route emissions | System estimates emissions per route. Method: INSUFFICIENT_DATA. | D1 |
| D1 | Decision | Route found? | Checks whether at least one valid route exists. | A3 / IO2 |
| IO2 | Data I/O | Display error message | Informs the user that no route was found. | IO1 (Re-enter data) |
| A3 | Action | Rank routes by emissions | Orders routes from lowest to highest emissions. | A4 |
| A4 | Action | Present greenest route | Shows the lowest-emission route to the user. | D2 |
| D2 | Decision | User accepts route? | User confirms or rejects the proposed route. | A6 / A5 |
| A5 | Action | Adjust route preferences | User changes preferences; routes are fetched again. | A1 |
| A6 | Action | Start navigation | Turn-by-turn guidance begins. | D3 |
| D3 | Decision | User cancels trip? | Checks whether the user cancels during the trip. | A7 / A8 |
| A7 | Action | Cancel navigation | Navigation stops at the user's request. | T2 |
| A8 | Action | Log trip emissions | Stores the emissions of the completed trip. Storage: INSUFFICIENT_DATA. | D4 |
| D4 | Decision | Emissions logged? | Checks whether logging succeeded. | T2 / IO3 |
| IO3 | Data I/O | Display logging error | Informs the user that the trip could not be logged. | T2 |
| T2 | Terminator | End | Exit point of the workflow. | — |

## 2.3 Decision logic

| ID | Question | Yes path | No path |
|---|---|---|---|
| D1 | Route found? | Rank routes by emissions | Display error message → Re-enter data |
| D2 | User accepts route? | Start navigation | Adjust route preferences → Fetch route options |
| D3 | User cancels trip? | Cancel navigation → End | Log trip emissions |
| D4 | Emissions logged? | End | Display logging error → End |

## 2.4 Exception and error handling

- **No route found (D1 = No):** an error message is displayed and the user must re-enter origin and destination. The loop is user-driven; no automatic retry is modelled.
- **Route rejected (D2 = No):** the user adjusts route preferences and the system fetches new route options. No limit on iterations is defined.
- **User cancellation (D3 = Yes):** navigation is cancelled and the flow ends; no emissions are logged for the cancelled trip.
- **Logging failure (D4 = No):** a logging error is displayed and the flow ends; the trip data is not retried or stored elsewhere.

## 2.5 Data inputs and outputs

- **Inputs:** origin and destination (IO1); route preferences (A5).
- **Outputs:** greenest route (A4); error message (IO2); logging error (IO3); trip-emissions record (A8).

## 2.6 Assumptions, open questions and INSUFFICIENT_DATA items

- Route data source for "Fetch route options": INSUFFICIENT_DATA.
- Emission-calculation method and emission factors: INSUFFICIENT_DATA.
- Storage target for "Log trip emissions": INSUFFICIENT_DATA.
- Order of steps: emissions are calculated before "Route found?" is checked. The team should confirm whether the check should come directly after "Fetch route options".
- System failure while fetching routes (e.g. service unavailable) is not distinguished from "no route found".
- Cancellation is checked once after "Start navigation"; cancellation at earlier steps is not modelled.
- No maximum number of retries is defined for the two loops.
