# Requirements Document

## Introduction

Moneymize is a comprehensive personal finance dashboard built with Next.js (App Router) and TypeScript. It provides users with a single, clean interface to track income and expenses, set and monitor financial goals, plan major purchases with jurisdiction-aware benefit calculators, analyse investments with live market data, chart a path to financial independence through FIRE planning, and calculate Canadian take-home pay with RRSP optimisation.

User data is persisted securely in AWS — authentication via Amazon Cognito, data in DynamoDB (single-table design), and the application hosted on AWS Amplify. The infrastructure is 100% AWS-native, serverless, and scales to zero.

---

## Glossary

- **System**: The Moneymize Next.js application as a whole.
- **App**: Synonym for System.
- **User**: An authenticated person using the application.
- **Dashboard**: The `/dashboard` page displaying the overview of a user's financial position.
- **Transaction**: A single income or expense record belonging to a user.
- **Goal**: A named savings target with a target amount and target date.
- **Holding**: A portfolio position representing shares of a ticker symbol owned by a user.
- **Calculator**: Any of the four interactive financial calculators (Buy vs Wait, Home Purchase, Investment, Take-Home Pay).
- **FIRE**: Financial Independence, Retire Early — a planning methodology.
- **FIRE_Dashboard**: The `/fire` page where users plan their path to financial independence.
- **BenefitRegistry**: The static, data-driven registry of Canadian government home-purchase benefit programs.
- **TaxBracketRegistry**: The versioned, data-driven store of federal and provincial Canadian tax brackets.
- **MarketDataProvider**: The provider-agnostic adapter that fetches live ticker data from an external market data API.
- **Repository**: A typed data-access layer that abstracts DynamoDB operations.
- **ServerAction**: A Next.js Server Action that executes server-side logic in response to client events.
- **Middleware**: The Next.js middleware that verifies JWT tokens on every request.
- **Cognito**: Amazon Cognito User Pool used for authentication and user management.
- **DynamoDB**: Amazon DynamoDB used as the primary database with a single-table design.
- **Amplify**: AWS Amplify Hosting used to host and serve the Next.js application.
- **JWT**: JSON Web Token issued by Cognito after successful authentication.
- **RRSP**: Registered Retirement Savings Plan — a Canadian tax-advantaged savings account.
- **FHSA**: First Home Savings Account — a Canadian tax-free savings account for first-time home buyers.
- **HBP**: Home Buyers' Plan — allows withdrawal from RRSP for a first home purchase.
- **CMHC**: Canada Mortgage and Housing Corporation — provides mortgage insurance for low-down-payment purchases.
- **LTT**: Land Transfer Tax — a provincial tax on real estate purchases.
- **CPP**: Canada Pension Plan — mandatory employee and employer contributions.
- **EI**: Employment Insurance — mandatory employee premium deductions.
- **MinorUnits**: Integer representation of a monetary amount in cents (e.g., $10.00 = 1000).
- **DesignTokens**: CSS custom properties and Tailwind configuration values derived from the money-green seed colour.
- **MonthlyAggregate**: A pre-computed DynamoDB item storing per-month income, expense, and cash-flow totals.
- **CalculatorPlan**: A persisted snapshot of calculator inputs and results saved by a user.
- **FIREScenario**: One of five FIRE planning variants: Lean, Regular, Fat, Coast, or Barista.
- **PayFrequency**: The cadence at which a user is paid: weekly, biweekly, semi-monthly, or monthly.
- **CanadianProvince**: One of the 13 Canadian provinces and territories (AB, BC, MB, NB, NL, NS, NT, NU, ON, PE, QC, SK, YT).

---

## Requirements

### Requirement 1: User Authentication

**User Story:** As a new or returning user, I want to register, verify my email, sign in, and recover my password, so that my financial data is protected and accessible only to me.

#### Acceptance Criteria

1. WHEN a user submits a valid email address and a password of at least 8 characters containing at least one uppercase letter and one number on the sign-up page, THE System SHALL create a new Cognito user account and send a verification email via SES.
2. WHEN a user submits the correct 6-digit verification code on the verify page within the code's validity window, THE System SHALL mark the Cognito account as confirmed and redirect the user to `/login`. IF the code is invalid or expired, THE System SHALL display an error message and allow the user to request a new code.
3. WHEN a user submits valid credentials on the login page, THE System SHALL exchange credentials with Cognito, receive JWT tokens, and store them in httpOnly, Secure, SameSite=Strict cookies.
4. WHEN a user navigates to any protected route, THE Middleware SHALL verify the JWT signature against Cognito's JWKS endpoint and redirect unauthenticated requests to `/login`.
5. WHEN the AccessToken has less than 5 minutes remaining, THE Middleware SHALL silently exchange the RefreshToken with Cognito for a new AccessToken and update the cookie without requiring the user to re-authenticate. IF the RefreshToken is expired or invalid, THE System SHALL clear the cookies and redirect the user to `/login`.
6. WHEN a user submits a password reset request with a registered email address, THE System SHALL trigger a Cognito password reset email containing a confirmation code.
7. WHEN a user submits a valid confirmation code and a new password meeting the password policy on the reset page, THE System SHALL update the Cognito account password. IF the code is invalid or expired, THE System SHALL display an error message.
8. IF a user submits an incorrect password 5 or more consecutive times, THEN THE System SHALL lock the account for 15 minutes and display a message indicating the lockout duration.
9. THE System SHALL never store raw passwords — all credential handling SHALL be delegated to Cognito.
10. WHEN a ServerAction executes, THE System SHALL extract the userId exclusively from the verified JWT and never from client-supplied input.

---

### Requirement 2: Dashboard Overview

**User Story:** As an authenticated user, I want to see a summary of my financial position on the dashboard, so that I can quickly understand my net worth, spending, goals, and cash flow at a glance.

#### Acceptance Criteria

1. WHEN an authenticated user loads `/dashboard`, THE System SHALL render the page server-side and display the Net Worth Widget, Spending Summary Widget, Goals Progress Widget, Recent Transactions Widget, Monthly Cash Flow Chart, and FIRE Progress Widget.
2. WHEN the Dashboard loads, THE System SHALL fetch transaction summaries, goals, recent transactions, and monthly aggregates using the Repository layer.
3. THE Net_Worth_Widget SHALL display the user's total net worth and a directional trend indicator showing the percentage change compared to the previous calendar month.
4. THE Spending_Summary_Widget SHALL display a doughnut chart of spending broken down by category for the current calendar month, along with total spent. WHERE the user has set a monthly budget, THE Spending_Summary_Widget SHALL also display budget utilisation as a percentage.
5. THE Cash_Flow_Chart SHALL display a bar chart of monthly income versus expenses over the trailing 12 calendar months using pre-computed MonthlyAggregate items.
6. THE Goals_Progress_Widget SHALL display up to three goals where the target date has not passed and `currentAmount` is less than `targetAmount`, each with a progress bar and target date.
7. THE Recent_Transactions_Widget SHALL display the five most recent transactions with category, amount, and date.
8. IF a widget fails to load data, THEN THE System SHALL display a fallback card reading "Unable to load data" without affecting other widgets.

---

### Requirement 3: Transaction Management

**User Story:** As a user, I want to create, view, edit, delete, and filter my income and expense transactions, so that I can maintain an accurate record of my financial activity.

#### Acceptance Criteria

1. WHEN a user submits a valid transaction form, THE System SHALL persist the transaction to DynamoDB and atomically update the corresponding MonthlyAggregate in a single transactional write.
2. WHEN a transaction is created, THE System SHALL assign a unique identifier, record creation and update timestamps, and return the created Transaction to the client.
3. WHEN a user edits a transaction, THE System SHALL update the stored transaction and adjust the affected MonthlyAggregate by subtracting the old amount and adding the new amount.
4. WHEN a user deletes a transaction, THE System SHALL remove the transaction record and decrement the corresponding MonthlyAggregate by the transaction's amount.
5. WHEN a user applies filters on the transactions page, THE System SHALL return only transactions matching all specified criteria: type, category, date range, search text (matched against description and notes fields), tags, and amount range. IF no transactions match the active filters, THE System SHALL display an empty state message.
6. IF a transaction form is submitted with any of the following conditions, THEN THE System SHALL return validation errors keyed to the invalid field names without persisting any data: missing type, missing or non-positive amount, amount exceeding 999,999,999.99 in major units, missing category, missing description, missing date, description exceeding 255 characters, notes exceeding 1,000 characters, more than 10 tags, or any individual tag exceeding 50 characters.
7. THE Transaction_Form SHALL support the fields: type (income or expense), amount (0.01–999,999,999.99 in major units), category, description (up to 255 characters), date, tags (up to 10 items, each up to 50 characters), and optional notes (up to 1,000 characters).
8. WHEN a transaction is created or updated, THE System SHALL store the amount in MinorUnits.
9. WHEN the transaction list contains 100 or more transactions, THE Transaction_List SHALL use virtualised rendering.

---

### Requirement 4: Financial Goals

**User Story:** As a user, I want to create and track savings goals with target amounts and dates, so that I can plan and monitor progress toward specific financial objectives.

#### Acceptance Criteria

1. WHEN a user submits a valid goal form with a name (1–100 characters), a target amount (0.01–9,999,999.99 in major units), a current amount (0–target amount), a target date (today or later), a category, and a colour, THE System SHALL persist the goal to DynamoDB and return the created Goal.
2. WHEN a user adds a positive fund amount to a goal, THE System SHALL increment `currentAmount` by the specified amount and update the modification timestamp.
3. IF a user attempts to add funds that would cause `currentAmount` to exceed `targetAmount`, THEN THE System SHALL display a confirmation dialog showing the overage amount. IF the user confirms, THE System SHALL cap `currentAmount` at `targetAmount`. IF the user cancels, THE System SHALL make no changes.
4. WHEN a user edits a goal, THE System SHALL update the stored goal with the new values.
5. WHEN a user deletes a goal, THE System SHALL remove the goal from DynamoDB.
6. THE Goal_Card SHALL display the goal name, a progress bar showing `currentAmount / targetAmount`, current amount, target amount, target date, and projected completion date.
7. WHEN projecting goal completion, THE System SHALL use the compound growth formula with the user-supplied monthly contribution (0–99,999.99 in major units) and annual return rate (0–50%).
8. WHEN the projected completion date is after the target date, THE Goal_Card SHALL display an off-track indicator and the shortfall expressed in whole months.

---

### Requirement 5: Buy vs Wait Calculator

**User Story:** As a user, I want to compare buying an item now versus saving up for it, so that I can make an informed decision that accounts for opportunity cost and price inflation.

#### Acceptance Criteria

1. WHEN a user enters item price (0.01–9,999,999.99), current savings (0–9,999,999.99), monthly savings contribution (0–99,999.99), expected annual return (0–100%), and inflation rate (0–100%), THE System SHALL compute and display the Buy vs Wait result within 300 milliseconds without a page reload.
2. THE Buy_vs_Wait_Calculator SHALL calculate the number of months required to save the inflation-adjusted price.
3. THE Buy_vs_Wait_Calculator SHALL calculate the investment gain foregone if the user waits (opportunity cost of saving).
4. THE Buy_vs_Wait_Calculator SHALL produce a recommendation of "buy-now" or "wait" based on whether the price appreciation exceeds the investment gain from waiting.
5. IF the goal is unreachable at the current savings rate within 600 months, THEN THE System SHALL display a message stating the goal is unreachable at the current savings rate and SHALL NOT display a recommendation.
6. IF a user enters a non-numeric value or a value outside the valid range for any input field, THEN THE System SHALL display an inline validation message below that field and SHALL NOT perform a calculation until all inputs are valid.
7. WHEN any input value changes, THE System SHALL recompute results within 300 milliseconds without requiring an explicit submit action.

---

### Requirement 6: Home Purchase Calculator

**User Story:** As a Canadian user planning to buy a home, I want to calculate the total cash required including down payment, closing costs, and applicable government benefits, so that I can plan my savings timeline accurately.

#### Acceptance Criteria

1. WHEN a user enters property price (0.01–9,999,999.99), province, first-time buyer status, annual income (0–9,999,999.99), current savings (0–9,999,999.99), monthly savings (0–99,999.99), amortisation years (5–30), and mortgage rate (0.01%–25%), THE System SHALL compute the full HomePurchaseResult.
2. THE Home_Purchase_Calculator SHALL calculate the minimum down payment according to CMHC rules: 5% for properties up to $500,000; 5% on the first $500,000 plus 10% on the remainder for properties between $500,000 and $999,999; and 20% for properties at or above $1,000,000.
3. THE Home_Purchase_Calculator SHALL calculate the CMHC mortgage insurance premium at 3.15% for down payments below 10%, 2.80% for 10%–14.99%, and 2.40% for 15%–19.99%, and zero for down payments of 20% or more.
4. THE Home_Purchase_Calculator SHALL calculate provincial land transfer tax using province-specific bracket schedules.
5. THE Home_Purchase_Calculator SHALL calculate the monthly mortgage payment using the standard amortisation formula.
6. THE Benefit_Toggle_Panel SHALL render all applicable benefits from the BenefitRegistry filtered by province and first-time buyer status.
7. WHEN a user toggles a benefit, THE System SHALL recompute the HomePurchaseResult with the updated set of active benefits.
8. THE Home_Purchase_Calculator SHALL calculate `netCashRequired` as down payment plus total closing costs minus the sum of all active benefit offsets.
9. THE Home_Purchase_Calculator SHALL calculate the number of months required to save `netCashRequired` using compound growth with the user's current savings and monthly savings contribution.
10. WHEN a benefit has an `effectiveTo` date in the past, THE Benefit_Toggle_Panel SHALL render it with a greyed-out "Ended [date]" badge and disable the toggle.
11. WHEN a new benefit is added to the BenefitRegistry, THE Benefit_Toggle_Panel SHALL display it without requiring changes to any component code.
12. WHEN a user saves a calculator plan with a name between 1 and 100 characters, THE System SHALL persist the plan to DynamoDB. IF the save operation fails, THE System SHALL display an error toast and retain the current calculator state.

---

### Requirement 7: Investment / Seed Money Calculator

**User Story:** As a user, I want to project the growth of an initial investment with optional recurring contributions, so that I can understand the long-term impact of compound growth.

#### Acceptance Criteria

1. WHEN a user enters seed amount (0–999,999,999 in major units), monthly contribution (0–99,999 in major units), annual return rate (0–100%), years (1–50), and compounding frequency, THE System SHALL compute and display the InvestmentResult.
2. THE Investment_Calculator SHALL compute final value, total contributed, total growth, growth percentage, and a yearly breakdown showing year number, balance, cumulative contributed, and growth for each year.
3. WHEN a user enters a ticker symbol, THE System SHALL call the market quote endpoint and populate the seed amount field with the live price. IF the ticker symbol is not found, THE System SHALL display an inline error message and leave the seed amount field unchanged.
4. WHEN the market data API is unavailable, THE System SHALL display "Quote unavailable — enter price manually" and allow the user to enter the price manually.
5. THE Market_Data_API SHALL serve cached quote responses for up to 60 seconds before fetching a fresh quote from the external provider.
6. THE Investment_Calculator SHALL support compounding frequencies of monthly, quarterly, and annually.
7. WHEN a user saves a calculator plan with a name between 1 and 100 characters, THE System SHALL persist the plan to DynamoDB. IF the save operation fails, THE System SHALL display an error toast and retain the current calculator state.

---

### Requirement 8: Take-Home Pay Calculator

**User Story:** As a Canadian employee, I want to calculate my net take-home pay including all federal and provincial taxes, CPP, EI, and optional RRSP contributions, so that I can understand my true after-tax income.

#### Acceptance Criteria

1. WHEN a user enters employment income (0–9,999,999 in major units), province, tax year (2020 to current year + 1), and pay frequency, THE System SHALL compute the full TakeHomeResult using the TaxBracketRegistry.
2. THE Take_Home_Calculator SHALL apply federal and provincial income tax brackets progressively using the rates registered for the selected tax year and province.
3. THE Take_Home_Calculator SHALL calculate CPP1 contributions, CPP2 contributions, and EI premiums using the versioned CPP and EI rate tables for the selected tax year.
4. THE Take_Home_Calculator SHALL apply the federal and provincial Basic Personal Amount non-refundable credits.
5. THE Take_Home_Calculator SHALL apply Ontario and PEI surtax when the selected province is ON or PE.
6. THE Take_Home_Calculator SHALL support optional inputs: bonus amount, RSU vest amount with adjusted cost basis, other income, RRSP contribution, union dues, professional fees, childcare expenses, employer RRSP match, and group benefits premium.
7. THE Take_Home_Calculator SHALL compute a per-paycheque breakdown of gross pay, federal tax withheld, provincial tax withheld, CPP withheld, EI withheld, RRSP deducted, and net pay.
8. THE Take_Home_Calculator SHALL compute `netIncome` as `grossIncome - totalTax - totalDeductions`.
9. THE Take_Home_Calculator SHALL compute `totalTax` as `netFederalTax + netProvincialTax + cppContributions + eiPremiums`.
10. THE Take_Home_Calculator SHALL compute `marginalCombinedRate` as `marginalFederalRate + marginalProvincialRate`.
11. WHEN a user selects a tax year for which no bracket data exists, THE System SHALL fall back to the most recent available year and display a banner indicating the approximation.
12. WHEN a user clicks "Optimize RRSP", THE System SHALL compute the optimal RRSP contribution that minimises total tax by targeting the nearest lower bracket boundary.
13. THE RRSP_Optimizer SHALL produce `optimalContribution` that is less than or equal to `availableRRSPRoom`. IF `availableRRSPRoom` is zero or the user is already in the lowest bracket, THE System SHALL display a message indicating no further optimisation is possible.
14. THE RRSP_Optimizer SHALL produce `taxRefundEstimate` that is greater than or equal to zero.
15. THE RRSP_Optimizer SHALL produce contribution scenarios for $5,000, $10,000, $15,000, $20,000, the optimal amount, and the maximum available room.
16. WHEN a user enters an RRSP room figure exceeding the CRA annual maximum for the selected tax year as defined in the TaxBracketRegistry, THE System SHALL display an inline warning without blocking the calculation.
17. THE Take_Home_Calculator SHALL support all 13 Canadian provinces and territories.
18. THE TaxBracketRegistry SHALL be versioned such that adding a new tax year requires only a new data entry with no algorithm changes.

---

### Requirement 9: FIRE Dashboard

**User Story:** As a user pursuing financial independence, I want to calculate my FIRE number, project the years to financial independence, and analyse withdrawal sustainability across multiple scenarios, so that I can plan my path to early retirement.

#### Acceptance Criteria

1. WHEN a user enters current age (1–100), current savings (0–99,999,999 in major units), annual income (0–9,999,999), annual expenses (0.01–9,999,999), annual savings contribution (0–9,999,999), expected annual return (0–50%), inflation rate (0–50%), and withdrawal rate (0.01–20%), THE System SHALL compute the full FIREResult.
2. THE FIRE_Dashboard SHALL compute `fireNumber` as `annualExpenses / targetWithdrawalRate`.
3. THE FIRE_Dashboard SHALL compute `yearsToFI` by simulating annual compound growth plus contributions until the portfolio reaches `fireNumber`.
4. THE FIRE_Dashboard SHALL compute `projectedFIAge` as `currentAge + yearsToFI`.
5. THE FIRE_Dashboard SHALL compute `coastFIRENumber` as `fireNumber / (1 + expectedAnnualReturn) ^ yearsToRetirement` where retirement age is 65.
6. THE FIRE_Dashboard SHALL simulate a 30-year withdrawal phase and produce a year-by-year projection showing portfolio value, annual withdrawal, inflation-adjusted withdrawal, and whether the portfolio survives. A year is considered to survive when the portfolio value is greater than zero at the end of that year.
7. THE FIRE_Dashboard SHALL record milestones at 25%, 50%, and 75% of the FIRE number during the accumulation phase.
8. THE FIRE_Scenario_Selector SHALL support five scenarios: Lean FIRE, Regular FIRE, Fat FIRE, Coast FIRE, and Barista FIRE.
9. WHEN a user selects a different FIRE scenario, THE System SHALL recompute the FIREResult without requiring additional user action.
10. WHEN a user saves a FIRE plan with a name between 1 and 100 characters, THE System SHALL persist the plan to DynamoDB. IF the save operation fails, THE System SHALL display an error toast and retain the current calculator state.

---

### Requirement 10: Investment Portfolio

**User Story:** As a user, I want to track my investment holdings with live market prices, so that I can monitor my portfolio value and performance.

#### Acceptance Criteria

1. WHEN a user adds a holding with a valid ticker (1–10 characters), name (1–100 characters), shares greater than zero, and average cost basis of at least 0.01 in major units, THE System SHALL persist it to DynamoDB.
2. WHEN the portfolio page loads, THE System SHALL enrich each holding with the current price, current value (shares × current price), gain/loss ((current price − average cost basis) × shares), and gain/loss percentage ((current price − average cost basis) / average cost basis × 100) from the MarketDataProvider.
3. WHEN a user edits a holding, THE System SHALL update the stored holding with the new values.
4. WHEN a user deletes a holding, THE System SHALL remove it from DynamoDB.
5. THE Portfolio_Holdings_Table SHALL display ticker, name, shares, average cost basis, current price, current value, gain/loss, and gain/loss percentage for each holding, sorted by current value descending by default.
6. WHEN the market data API is unavailable for a holding, THE System SHALL display the holding with a "Price unavailable" indicator and continue displaying other holdings.

---

### Requirement 11: Market Data API

**User Story:** As a user of the investment and calculator features, I want live ticker quotes to be fetched reliably and efficiently, so that my calculations reflect current market prices.

#### Acceptance Criteria

1. WHEN a client requests a quote via the market quote endpoint with a valid ticker symbol, THE System SHALL return the live quote including price, change, change percent, name, currency, and exchange.
2. THE Market_Data_API SHALL serve cached quote responses for up to 60 seconds before fetching a fresh quote from the external provider.
3. WHEN the external market data provider returns an error or does not respond within 5 seconds, THE System SHALL return HTTP 502 with `{ error: "Quote unavailable" }`.
4. THE Market_Data_Provider SHALL be swappable by changing the `MARKET_DATA_PROVIDER` environment variable without modifying consumer code.
5. WHEN a client requests a ticker search, THE System SHALL return results including ticker symbol, name, exchange, and asset type.
6. WHEN a client requests historical prices for a valid ticker and date range, THE System SHALL return a list of daily price records.
7. THE System SHALL store market API keys in AWS Secrets Manager and never hardcode them in source code.

---

### Requirement 12: Data Persistence and Repository Layer

**User Story:** As a developer, I want all data access to go through typed repository interfaces backed by DynamoDB, so that the data layer is consistent, testable, and replaceable.

#### Acceptance Criteria

1. THE Repository_Layer SHALL expose typed interfaces for transactions, goals, holdings, calculator plans, and user profiles.
2. WHEN a transaction is created, THE System SHALL atomically write the transaction item and update the MonthlyAggregate item in a single DynamoDB TransactWrite operation. IF the TransactWrite operation fails, THE System SHALL return an error and make no partial changes.
3. THE System SHALL store all monetary amounts in MinorUnits (integer cents) in DynamoDB.
4. THE DynamoDB_Table SHALL use a single-table design with composite keys (`PK` + `SK`) supporting all defined access patterns without full-table scans.
5. THE System SHALL use DynamoDB on-demand billing mode.
6. WHEN querying recent transactions, THE Repository SHALL return results in descending date order using the `TX#<YYYY-MM-DD>#<uuid>` sort key format.
7. WHEN querying monthly cash flow, THE Repository SHALL read pre-computed MonthlyAggregate items rather than aggregating raw transactions at read time.
8. THE System SHALL use the `@aws-sdk/lib-dynamodb` DynamoDBDocumentClient with `removeUndefinedValues: true`.

---

### Requirement 13: Authentication Security

**User Story:** As a security-conscious user, I want my session to be managed securely with proper token handling, so that my account cannot be accessed by unauthorised parties.

#### Acceptance Criteria

1. THE System SHALL store JWT tokens exclusively in httpOnly, Secure, SameSite=Strict cookies.
2. WHEN a request arrives at a protected route, THE Middleware SHALL verify the JWT signature using Cognito's JWKS endpoint via `aws-jwt-verify`. IF verification fails, THE Middleware SHALL redirect the request to `/login` with HTTP 302.
3. THE System SHALL protect all routes under `/dashboard`, `/transactions`, `/goals`, `/calculators`, `/investments`, `/fire`, and `/settings` from unauthenticated access by redirecting to `/login`.
4. THE System SHALL allow unauthenticated access to `/`, `/login`, `/signup`, and `/api/auth` routes.
5. THE Cognito_User_Pool SHALL enforce a minimum password length of 8 characters with at least one uppercase letter and one number.
6. THE Cognito_User_Pool SHALL use the Lite feature plan supporting up to 50,000 MAUs at no cost.
7. THE Cognito_User_Pool SHALL have deletion protection enabled.

---

### Requirement 14: Design System and UI

**User Story:** As a user, I want the application to have a consistent, professional visual design, so that it feels trustworthy and is easy to use.

#### Acceptance Criteria

1. THE System SHALL derive the entire colour palette from the money-green seed colour `#1aab62` using the defined token scale from `green-50` to `green-900`.
2. THE System SHALL define all colour tokens as CSS custom properties injected into `:root` by the RootLayout.
3. THE System SHALL consume design tokens via Tailwind CSS configuration.
4. THE System SHALL use `#ef4444` (red-500) for expense amounts and `#1aab62` (green-500) for income amounts throughout the UI.
5. THE System SHALL support light, dark, and system theme modes. WHEN a user changes their theme preference, THE System SHALL persist the selection to their user profile and apply it across all pages.
6. THE Sidebar SHALL be always visible on desktop viewports and collapsible on mobile viewports, providing navigation to Dashboard, Transactions, Goals, Calculators, Investments, FIRE, and Settings.
7. WHEN a server action completes successfully or fails, THE System SHALL display a toast notification for 5 seconds.

---

### Requirement 15: User Preferences

**User Story:** As a user, I want to configure my currency, locale, theme, and monthly budget, so that the application reflects my personal preferences.

#### Acceptance Criteria

1. WHEN a user updates their preferences, THE System SHALL persist the changes to the user profile item in DynamoDB. IF the persistence operation fails, THE System SHALL display an error toast and retain the previous preference values.
2. THE System SHALL support the following currency codes: USD, EUR, GBP, AUD, CAD, and NZD.
3. WHEN a user changes their currency or locale setting, THE System SHALL apply the selected currency and locale when formatting all monetary amounts across all pages.
4. WHEN a user changes their theme setting, THE System SHALL apply the selected theme (light, dark, or system) across all pages.
5. WHEN a user has set a monthly budget greater than zero, THE Spending_Summary_Widget SHALL display budget utilisation as `(totalExpenses / monthlyBudget) × 100` percent alongside actual spending.

---

### Requirement 16: Calculator Plan Persistence

**User Story:** As a user, I want to save and revisit my calculator scenarios, so that I can compare different plans over time without re-entering inputs.

#### Acceptance Criteria

1. WHEN a user saves a calculator plan with a name between 1 and 100 characters, THE System SHALL persist the plan name, type, serialised inputs, and serialised result snapshot to DynamoDB. IF the save operation fails, THE System SHALL display an error toast and retain the current calculator state.
2. THE System SHALL support plan types: HOME, FIRE, and INVESTMENT.
3. WHEN a user loads a saved plan, THE System SHALL deserialise the inputs and result and restore the calculator to that state. IF deserialisation fails, THE System SHALL display an error toast and leave the calculator in its current state.
4. THE System SHALL allow users to name their plans with between 1 and 100 characters.
5. WHEN a user deletes a saved plan, THE System SHALL remove it from DynamoDB.
6. THE System SHALL allow a maximum of 50 saved plans per plan type per user.

---

### Requirement 17: Error Handling

**User Story:** As a user, I want the application to handle errors gracefully and inform me clearly when something goes wrong, so that I can take corrective action without losing my work.

#### Acceptance Criteria

1. WHEN a ServerAction fails to persist data, THE System SHALL return `{ success: false, error: string }` and display a toast notification with the error message while keeping the form open with all previously entered values intact.
2. IF a dashboard widget fails to load, THEN THE System SHALL display a fallback card reading "Unable to load data" without affecting other widgets on the page.
3. WHEN the market data API is unavailable, THE System SHALL display "Quote unavailable — enter price manually" and render a manual price input field.
4. WHEN a take-home calculator tax year has no bracket data, THE System SHALL fall back to the most recent available year and display a banner indicating the approximation.
5. WHEN a benefit program has an `effectiveTo` date in the past, THE Benefit_Toggle_Panel SHALL render it with a greyed-out "Ended [date]" badge and prevent activation.
6. WHEN a user enters an RRSP room figure exceeding the CRA annual maximum for the selected tax year as defined in the TaxBracketRegistry, THE System SHALL display an inline warning without blocking the calculation.
7. WHEN a goal funding amount would cause `currentAmount` to exceed `targetAmount`, THE System SHALL display a confirmation dialog showing the overage amount (proposed amount minus remaining gap) before proceeding.

---

### Requirement 18: AWS Infrastructure

**User Story:** As a developer and operator, I want the application to run on a fully serverless, AWS-native infrastructure that scales to zero, so that hosting costs are negligible for personal use.

#### Acceptance Criteria

1. THE System SHALL be hosted on AWS Amplify with native Next.js SSR support and a global CDN.
2. THE System SHALL use Amazon Cognito Lite tier for authentication, supporting up to 50,000 MAUs at no cost.
3. THE System SHALL use Amazon DynamoDB on-demand billing with the always-free tier (25 GB storage, up to 200M requests/month).
4. THE System SHALL use Amazon S3 for CSV exports and profile avatars.
5. THE System SHALL use AWS Secrets Manager to store market API keys.
6. THE System SHALL use Amazon SES for transactional emails (verification and password reset).
7. THE System SHALL require no always-on infrastructure — all compute SHALL be serverless and scale to zero.

---

### Requirement 19: Transaction Aggregation

**User Story:** As a developer, I want transaction summaries to be computed correctly and efficiently, so that the dashboard loads quickly and financial figures are accurate.

#### Acceptance Criteria

1. WHEN aggregating transactions for a period, THE System SHALL compute `netCashFlow` as `totalIncome - totalExpenses`.
2. WHEN aggregating transactions for a period, THE System SHALL compute `savingsRate` as `((totalIncome - totalExpenses) / totalIncome) × 100` rounded to 2 decimal places when `totalIncome > 0`, and zero otherwise.
3. WHEN aggregating transactions for a period, THE System SHALL compute the category breakdown such that the sum of all category amounts equals `totalExpenses`.
4. WHEN aggregating transactions for a period, THE System SHALL compute the category breakdown percentages such that they sum to within ±1 percentage point of 100%.
5. WHEN a transaction is created, updated, or deleted, THE System SHALL update the corresponding MonthlyAggregate item atomically in the same DynamoDB TransactWrite operation. IF the atomic write fails, THE System SHALL return an error and make no partial changes.

---

### Requirement 20: Currency Formatting

**User Story:** As a user, I want all monetary amounts to be displayed in my preferred currency and locale format, so that figures are easy to read and culturally appropriate.

#### Acceptance Criteria

1. WHEN formatting a monetary amount, THE System SHALL divide the MinorUnits value by 100 before display.
2. WHEN formatting a monetary amount, THE System SHALL include the currency symbol appropriate for the user's locale.
3. WHEN formatting a negative monetary amount, THE System SHALL represent it with a leading minus sign or parentheses per the locale convention.
4. THE `formatCurrency` function SHALL accept a finite integer amount in MinorUnits, a valid ISO 4217 currency code from the supported set (USD, EUR, GBP, AUD, CAD, NZD), and a valid BCP 47 locale string, and SHALL return a non-empty string. IF any argument is invalid, THE function SHALL throw a descriptive error.
