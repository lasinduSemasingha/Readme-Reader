# Overall Customer Risk Rating - Full Technical Trace

This document explains every component used to derive:

- `OverallCustomerRiskScore`
- `OverallCustomerRiskRating`
- `OverallCustomerRiskTolerance`

for Customer Risk Assessment in AMLFinOps.

---

## Plain-language summary (for non-IT readers)

This is how the system decides a customer's overall risk:

1. The officer selects 7 customer attributes:
  - Location
  - Type of Product
  - Business/Occupation
  - Source of Funds
  - Anticipated Transaction Volume
  - Customer Type
  - Payment Mode
2. For each selected attribute, the system reads a predefined **Risk Rating** and **Risk Score** from master setup tables.
3. The system adds all 7 scores to get one **Total Risk Score**.
4. The total score is matched to a predefined row in the **Risk Tolerance table**.
5. That matched row gives:
  - **Overall Customer Risk Rating** (for example Low / Medium / High)
  - **Overall Customer Risk Tolerance**

### How customers are categorized

- Customers are categorized by the **final mapped rating** from the Risk Tolerance table.
- In other words, the category is not hardcoded in this calculation file; it depends on the score-to-rating setup maintained in `Tbl_Mas_RiskTolerance`.

### Special rule: PEP customers

If customer is treated as PEP in this workflow, the system forces:

- Overall Customer Risk Score = **21**
- Overall Customer Risk Rating = **High**
- Overall Customer Risk Tolerance = **20**

So PEP customers are automatically categorized as high risk in this implementation.

### Simple example

If the 7 selected factors produce scores: `3, 2, 3, 2, 3, 2, 1`

- Total Risk Score = `3+2+3+2+3+2+1 = 16`
- System looks up score `16` in Risk Tolerance setup
- The row for `16` decides the final category/rating and tolerance

---

## A. Components involved in the calculation

## 1) UI screens and script

- `AML/Views/CustomerRiskAssessment/Create.cshtml`
- `AML/Views/CustomerRiskAssessment/Edit.cshtml`
- `AML/Scripts/amsjs/CustomerRiskAssessment.js`

The two forms render all risk factor inputs and include `CustomerRiskAssessment.js`, where the final overall score is computed.

## 2) Server endpoints used by the script

In `AML/Controllers/CustomerRiskAssessmentController.cs`:

- `JsonResult RiskData(int Id, int Type)`
- `JsonResult RiskToleranceData(int Id)`

`RiskData` supplies per-factor risk score/rating.
`RiskToleranceData` maps total score to overall rating + tolerance.

## 3) Master entities that provide individual factor risk values

Each of these entities stores `Riskrating` and `RiskScore`:

- `DCL/RiskLocation.cs`
- `DCL/TypeOfProduct.cs`
- `DCL/BusinessType.cs`
- `DCL/SourceOfFund.cs`
- `DCL/AnticipatedVolum.cs`
- `DCL/CustomerType.cs`
- `DCL/PaymentMode.cs`

These are the seven source dimensions used in the overall calculation.

## 4) Master entity for total-score-to-rating mapping

- `DCL/RiskTolerance.cs` (table `Tbl_Mas_RiskTolerance`)

Key fields:

- `RiskScore`
- `RiskrRating`
- `RiskTolerancePercentage`

This table defines what overall rating/tolerance corresponds to each final summed score.

## 5) Transaction entity where final values are stored

- `DCL/CustomerRiskAssessment .cs` (table `Tbl_Trn_CustomerRiskAssessment`)

Final persisted fields:

- `OverallCustomerRiskScore`
- `OverallCustomerRiskRating`
- `OverallCustomerRiskTolerance`

---

## B. Exact runtime flow (step-by-step)

## Step 1: User opens Create or Edit

`CustomerRiskAssessmentController` prepares dropdown lists via:

- `GetCreateData(...)`
- `GetEditCustomerRiskAssessment(...)`

Dropdown data sources:

- `_RiskLocationFacade.GetActive()`
- `_TypeOfProductFacade.GetActive()`
- `_BusinessTypeFacade.GetActive()`
- `_SourceOfFundFacade.GetActive()`
- `_AnticipatedVolumFacade.GetActive()`
- `_CustomerTypeFacade.GetActive()`
- `_PaymentModeFacade.GetActive()`

So each selected dropdown option corresponds to a master row with a predefined `Riskrating` and `RiskScore`.

## Step 2: User changes a risk factor

In `CustomerRiskAssessment.js`, each dropdown has a `change` handler:

- `#Location` -> Type `1`
- `#Typeofproduct` -> Type `2`
- `#BusinessType` -> Type `3`
- `#Sourceoffunds` -> Type `4`
- `#Anticipatedtransaction` -> Type `5`
- `#CustomerType` -> Type `6`
- `#PaymentMode` -> Type `7`

Each handler calls:

```http
POST /CustomerRiskAssessment/RiskData
```

Payload:

```json
{ "Id": <selected master id>, "Type": <1..7> }
```

Server (`RiskData`) loads the corresponding master entity row and returns:

```json
{ "Riskrating": "...", "RiskScore": <int> }
```

The script writes these values into factor-specific fields, for example:

- `#LocationRiskrating`, `#LocationRiskScore`
- `#TypeofproductRiskrating`, `#TypeofproductRiskScore`
- etc.

## Step 3: UI computes the total score

After each factor update (for non-PEP flow), script function `riskcalculation()` runs.

It reads all seven score fields (defaulting to `0` when empty), then calculates:

```text
riskcalculation =
  parseInt(LocationRiskScore)
+ parseInt(TypeofproductRiskScore)
+ parseInt(BusinessTypeRiskScore)
+ parseInt(SourceoffundsRiskScore)
+ parseInt(AnticipatedtransactionRiskScore)
+ parseInt(CustomerTypeRiskScore)
+ parseInt(PaymentModeRiskScore)
```

So the algorithm is a direct arithmetic sum of the 7 dimension scores.

## Step 4: UI asks server for total-score mapping

The script then calls:

```http
POST /CustomerRiskAssessment/RiskToleranceData
```

Payload:

```json
{ "Id": <sum_of_7_scores> }
```

Server method `RiskToleranceData(int Id)` does:

- `_RiskToleranceFacade.Get().Where(x => x.RiskScore == Id).FirstOrDefault()`

Returned response shape:

```json
{
  "Riskrating": "<RiskrRating>",
  "RiskScore": <RiskScore>,
  "RiskTolerancePercentage": "<RiskTolerancePercentage>"
}
```

UI then sets:

- `#OverallCustomerRiskScore`
- `#OverallCustomerRiskRating`
- `#OverallCustomerRiskTolerance`

## Step 5: Save operation persists UI-computed values

In `Create(CustomerRiskAssessment CustomerRiskAssessment)` and `Edit(CustomerRiskAssessment CustomerRiskAssessment)`, the posted model is inserted/updated.

Important: the controller does not recompute overall risk server-side during save; it stores what the form posts.

---

## C. PEP path (special/override behavior)

PEP is controlled by dropdown bound to `IsActive` in Create/Edit forms and function `IsPEP()` in script.

## 1) Controller-side PEP setup on Create

When `CustomerRiskAssessment.IsActive == true` in `Create(...)`:

- Model errors for factor fields are cleared.
- The controller fetches rows where `Description == "PEP" && IsActive == false` from each master table.
- It auto-assigns those PEP IDs to all seven factor fields:
  - `Location`
  - `Typeofproduct`
  - `BusinessType`
  - `Sourceoffunds`
  - `Anticipatedtransaction`
  - `CustomerType`
  - `PaymentMode`
- Sets `IsPepStatus = true`.

## 2) Script-side PEP override

`IsPEP()` computes/requests tolerance but then force-sets:

- `OverallCustomerRiskScore = 21`
- `OverallCustomerRiskRating = "High"`
- `OverallCustomerRiskTolerance = 20`

So in PEP mode, the displayed/stored overall values are explicitly overridden to fixed high-risk values.

## 3) BusinessType-triggered PEP toggle

In `#BusinessType` change handler, if selected value is `16`, script does:

- `$('#IsActive').val('true').trigger('change');`

This can auto-switch into PEP behavior from a business type selection.

---

## D. Full factor map used in overall score

The seven exact factor score columns in transaction model (`CustomerRiskAssessment`) that feed the final sum:

1. `LocationRiskScore`
2. `TypeofproductRiskScore`
3. `BusinessTypeRiskScore`
4. `SourceoffundsRiskScore`
5. `AnticipatedtransactionRiskScore`
6. `CustomerTypeRiskScore`
7. `PaymentModeRiskScore`

The corresponding rating columns are also populated and saved:

- `LocationRiskrating`
- `TypeofproductRiskrating`
- `BusinessTypeRiskrating`
- `SourceoffundsRiskrating`
- `AnticipatedtransactionRiskrating`
- `CustomerTypeRiskrating`
- `PaymentModeRiskrating`

---

## E. API contract details used by the UI

## 1) `RiskData` input/output

Input:

- `Id` = selected master record ID
- `Type` = factor selector (1..7)

Output object (`RiskCollection`):

- `Riskrating` (string)
- `RiskScore` (int)

## 2) `RiskToleranceData` input/output

Input:

- `Id` = total summed score

Output object (`RiskTolerance` inner class in controller):

- `Riskrating` (mapped from master `RiskrRating`)
- `RiskScore`
- `RiskTolerancePercentage`

Fallback on missing mapping/exception:

- blank `Riskrating`
- `RiskScore = 0`
- blank `RiskTolerancePercentage`

---

## F. What ultimately determines the overall rating

At runtime, the overall customer risk rating is determined by three data layers:

1. **Master factor tables**: provide each dimension's `RiskScore` (7 inputs).
2. **Client calculation**: sums those 7 scores.
3. **Risk tolerance master table**: maps total score to final rating/tolerance.

If PEP is enabled, final value is then overridden to fixed high-risk values in the script.

---

## G. One-line formula summary

For non-PEP flow:

```text
OverallCustomerRiskScore = sum(7 factor risk scores)
OverallCustomerRiskRating, OverallCustomerRiskTolerance = lookup by score in Tbl_Mas_RiskTolerance
```

For PEP flow:

```text
OverallCustomerRiskScore = 21
OverallCustomerRiskRating = High
OverallCustomerRiskTolerance = 20
```

---

## H. Mathematical formulation

Let the seven factor scores be:

- $s_1$ = LocationRiskScore
- $s_2$ = TypeofproductRiskScore
- $s_3$ = BusinessTypeRiskScore
- $s_4$ = SourceoffundsRiskScore
- $s_5$ = AnticipatedtransactionRiskScore
- $s_6$ = CustomerTypeRiskScore
- $s_7$ = PaymentModeRiskScore

Then the non-PEP overall score is:

$$
S = \sum_{i=1}^{7} s_i = s_1+s_2+s_3+s_4+s_5+s_6+s_7
$$

Where in the script empty values are treated as zero before summation.

Define risk tolerance master mapping function:

$$
M(S) = (R, T)
$$

such that:

- $R$ = overall risk rating (Low / Medium / High etc.)
- $T$ = risk tolerance percentage/value

and $M$ is implemented as exact DB lookup in `Tbl_Mas_RiskTolerance` by key `RiskScore = S`.

So for non-PEP:

$$
  ext{OverallCustomerRiskScore} = S
$$
$$
  ext{OverallCustomerRiskRating} = R
$$
$$
  ext{OverallCustomerRiskTolerance} = T
$$

with $(R,T)=M(S)$.

If no mapping row exists for score $S$, fallback behavior is:

$$
  ext{OverallCustomerRiskScore}=0,\quad
  ext{OverallCustomerRiskRating}=\varnothing,\quad
  ext{OverallCustomerRiskTolerance}=\varnothing
$$

For PEP override path:

$$
  ext{OverallCustomerRiskScore}=21,\quad
  ext{OverallCustomerRiskRating}=\text{High},\quad
  ext{OverallCustomerRiskTolerance}=20
$$

Piecewise final output can be written as:

$$
(\text{Score},\text{Rating},\text{Tolerance})=
\begin{cases}
(21,\text{High},20), & \text{if PEP mode is enabled} \\
(S,R,T), & \text{otherwise, where }(R,T)=M(S)
\end{cases}
$$
