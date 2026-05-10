# Implementation Plan: Moneymize

Moneymize is a full-stack Next.js personal finance dashboard. This plan converts the design into incremental coding tasks, building from infrastructure through data layer, feature modules, and integration.

## Tasks

- [ ] 1. Project scaffolding and AWS infrastructure setup
  - Initialise Next.js 14 App Router project with TypeScript, Tailwind CSS, ESLint, and Vitest
  - Configure 	ailwind.config.ts with the money-green design token scale (green-50 through green-900) and semantic tokens (income, expense, warning, info, surface tokens)
  - Create src/lib/design-tokens.ts exporting the full colorTokens map
  - Set up src/lib/db/dynamo-client.ts with DynamoDBDocumentClient (
emoveUndefinedValues: true)
  - Create .env.local.example documenting all required environment variables (DYNAMODB_TABLE_NAME, AWS_REGION, COGNITO_USER_POOL_ID, COGNITO_CLIENT_ID, MARKET_DATA_PROVIDER, NEXT_PUBLIC_*)
  - Configure Vitest with jsdom environment and @testing-library/jest-dom setup file
  - _Requirements: 12.8, 14.1, 14.2, 14.3, 18.1_

  - [ ] 1.1 Initialise Next.js project with TypeScript, Tailwind, and Vitest
    - Run create-next-app with App Router, TypeScript, Tailwind; install itest, @testing-library/react, @testing-library/jest-dom, ast-check, zustand, zod, 
echarts, 
eact-window, date-fns, uuid, lucide-react, @radix-ui/react-dialog, @radix-ui/react-dropdown-menu, @radix-ui/react-toggle, @aws-sdk/client-dynamodb, @aws-sdk/lib-dynamodb, ws-jwt-verify, mazon-cognito-identity-js, yahoo-finance2, decimal.js
    - _Requirements: 18.1_

  - [ ] 1.2 Configure design token system and Tailwind
    - Create src/lib/design-tokens.ts with full colorTokens map
    - Extend 	ailwind.config.ts to consume all tokens
    - _Requirements: 14.1, 14.2, 14.3_

  - [ ] 1.3 Set up DynamoDB client and environment config
    - Create src/lib/db/dynamo-client.ts (DynamoDBDocumentClient, TABLE_NAME export)
    - Create .env.local.example with all required variables
    - _Requirements: 12.8, 18.3_

- [ ] 2. Authentication — Cognito integration and auth pages
  - Implement src/lib/auth/session.ts with getSession() using ws-jwt-verify and CognitoJwtVerifier
  - Implement src/middleware.ts with JWT verification, public path allowlist, and silent token refresh logic
  - Create /api/auth/session route handler to set/clear httpOnly cookies
  - Build five auth pages: /login, /signup, /verify, /forgot, /reset using mazon-cognito-identity-js
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 1.8, 1.9, 1.10, 13.1, 13.2, 13.3, 13.4_

  - [ ] 2.1 Implement session management and JWT verification
    - Create src/lib/auth/session.ts with getSession(), Session interface
    - _Requirements: 1.3, 1.10, 13.1, 13.2_

  - [ ] 2.2 Implement Next.js middleware for route protection
    - Create src/middleware.ts with public path allowlist, JWT check, redirect to /login, and token refresh
    - _Requirements: 1.4, 1.5, 13.3, 13.4_

  - [ ]* 2.3 Write property test for middleware route protection
    - **Property 16: Middleware Route Protection**
    - **Validates: Requirements 1.4, 13.3, 13.4**

  - [ ] 2.4 Build auth pages (login, signup, verify, forgot, reset)
    - Implement all five auth pages with form validation, Cognito SDK calls, and error display
    - _Requirements: 1.1, 1.2, 1.6, 1.7, 1.8, 1.9_

  - [ ] 2.5 Implement /api/auth/session cookie route handler
    - Set/clear httpOnly, Secure, SameSite=Strict cookies on sign-in and sign-out
    - _Requirements: 1.3, 13.1_

- [ ] 3. Root layout, sidebar navigation, and Zustand store
  - Create src/app/layout.tsx (RootLayout) injecting CSS custom property tokens into :root, wrapping with ThemeProvider
  - Build Sidebar component with nav items for Dashboard, Transactions, Goals, Calculators, Investments, FIRE, Settings; collapsible on mobile
  - Implement src/store/finance-store.ts Zustand store with persist middleware
  - Add toast notification system (5-second auto-dismiss)
  - _Requirements: 14.2, 14.5, 14.6, 14.7_

  - [ ] 3.1 Create RootLayout with design token injection and ThemeProvider
    - Inject CSS custom properties into :root; wire ThemeProvider for light/dark/system
    - _Requirements: 14.2, 14.5_

  - [ ] 3.2 Build Sidebar navigation component
    - Responsive sidebar with all nav items; collapsible on mobile viewports
    - _Requirements: 14.6_

  - [ ] 3.3 Implement Zustand finance store
    - Create src/store/finance-store.ts with selectedPeriod, ctiveCurrency, sidebarCollapsed, pendingTransactions, and all actions
    - _Requirements: 14.5_

  - [ ] 3.4 Add toast notification system
    - Implement toast component with 5-second auto-dismiss; wire to server action success/failure
    - _Requirements: 14.7, 17.1_

- [ ] 4. DynamoDB single-table data layer and repository interfaces
  - Create src/lib/db/dynamo-types.ts with all DynamoDB item shape interfaces (UserProfileItem, TransactionItem, MonthlyAggregateItem, GoalItem, HoldingItem, CalculatorPlanItem)
  - Create src/lib/db/aggregates.ts with incrementMonthlyAggregate() using UpdateCommand with ADD expressions
  - Define typed repository interfaces: TransactionRepository, GoalRepository, HoldingRepository, CalculatorPlanRepository, UserProfileRepository
  - Implement DynamoTransactionRepository with TransactWrite for atomic transaction + aggregate writes
  - Implement DynamoGoalRepository, DynamoHoldingRepository, DynamoCalculatorPlanRepository, DynamoUserProfileRepository
  - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5, 12.6, 12.7, 12.8, 19.5_

  - [ ] 4.1 Define DynamoDB item types and repository interfaces
    - Create src/lib/db/dynamo-types.ts and all five repository interface files
    - _Requirements: 12.1, 12.3, 12.4_

  - [ ] 4.2 Implement aggregate maintenance helper
    - Create src/lib/db/aggregates.ts with incrementMonthlyAggregate()
    - _Requirements: 12.2, 19.5_

  - [ ] 4.3 Implement DynamoTransactionRepository
    - indById, indAll (with filters), getRecent, getSummary, getMonthlyCashFlow, create (TransactWrite), update, delete
    - _Requirements: 12.2, 12.5, 12.6, 12.7, 19.5_

  - [ ]* 4.4 Write integration tests for TransactionRepository
    - Test full CRUD lifecycle and atomic aggregate update using a local DynamoDB or mock
    - _Requirements: 12.2, 12.5, 12.6_

  - [ ] 4.5 Implement DynamoGoalRepository, DynamoHoldingRepository, DynamoCalculatorPlanRepository, DynamoUserProfileRepository
    - Full CRUD for each; plan repository enforces 50-plan-per-type limit
    - _Requirements: 12.1, 16.6_

  - [ ]* 4.6 Write integration tests for Goal, Holding, Plan, and UserProfile repositories
    - Test CRUD lifecycle and plan limit enforcement
    - _Requirements: 12.1, 16.6_

- [ ] 5. Checkpoint — Core infrastructure complete
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 6. Currency formatting and shared utilities
  - Implement src/lib/utils/format-currency.ts with ormatCurrency(amount, currency, locale) using Intl.NumberFormat
  - Implement src/lib/utils/calculate-savings-rate.ts with calculateSavingsRate(income, expenses)
  - Implement src/lib/utils/get-date-range.ts with getDateRange(period, customRange?)
  - Implement src/lib/utils/validate-transaction.ts with alidateTransaction(values) using Zod
  - _Requirements: 3.6, 3.7, 19.2, 20.1, 20.2, 20.3, 20.4_

  - [ ] 6.1 Implement formatCurrency utility
    - ormatCurrency(amount: number, currency: CurrencyCode, locale: string): string — divides minor units by 100, uses Intl.NumberFormat
    - _Requirements: 20.1, 20.2, 20.3, 20.4_

  - [ ]* 6.2 Write property test for formatCurrency
    - **Property 15: Currency Formatting Round-Trip Representation**
    - **Validates: Requirements 20.1, 20.2, 20.3, 20.4**

  - [ ] 6.3 Implement calculateSavingsRate, getDateRange, and validateTransaction utilities
    - calculateSavingsRate: returns [0, 100], zero when income is 0 or expenses exceed income
    - getDateRange: returns { start, end } for week/month/quarter/year/custom
    - alidateTransaction: Zod schema returning { valid, errors } keyed to field names
    - _Requirements: 3.6, 3.7, 19.2_

  - [ ]* 6.4 Write property test for calculateSavingsRate
    - **Property 5: Savings Rate Bounds**
    - **Validates: Requirements 19.2**

  - [ ]* 6.5 Write property test for validateTransaction
    - **Property 1: Transaction Validation Rejects Invalid Inputs**
    - **Validates: Requirements 3.6**

  - [ ]* 6.6 Write unit tests for getDateRange and formatCurrency edge cases
    - Test zero amounts, negative amounts, all supported currencies and locales
    - _Requirements: 20.1, 20.4_

- [ ] 7. Transaction management feature
  - Implement ggregateTransactions(transactions, period) pure function in src/lib/utils/aggregate-transactions.ts
  - Build TransactionForm component with all fields, Zod validation, and error display
  - Build TransactionList component with virtualised rendering (
eact-window) when >= 100 items
  - Implement server actions: createTransaction, updateTransaction, deleteTransaction (each validates JWT, calls repository, revalidates cache)
  - Wire filter bar with type, category, date range, search, tags, and amount range filters
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 19.1, 19.3, 19.4_

  - [ ] 7.1 Implement aggregateTransactions pure function
    - Computes 	otalIncome, 	otalExpenses, 
etCashFlow, savingsRate, and categoryBreakdown from a transaction array
    - _Requirements: 19.1, 19.2, 19.3, 19.4_

  - [ ]* 7.2 Write property test for aggregateTransactions net cash flow invariant
    - **Property 3: Monthly Aggregate Net Cash Flow Invariant**
    - **Validates: Requirements 3.1, 3.3, 3.4, 19.1**

  - [ ]* 7.3 Write property test for aggregateTransactions category breakdown invariant
    - **Property 4: Transaction Aggregation Category Breakdown Invariant**
    - **Validates: Requirements 19.3, 19.4**

  - [ ] 7.4 Build TransactionForm component
    - Controlled form with type, amount, category, description, date, tags, notes; inline validation errors
    - _Requirements: 3.6, 3.7_

  - [ ] 7.5 Build TransactionList component with virtualisation
    - Use 
eact-window FixedSizeList when transaction count >= 100; edit and delete callbacks
    - _Requirements: 3.5, 3.9_

  - [ ]* 7.6 Write property test for transaction filter consistency
    - **Property 2: Transaction Filter Consistency**
    - **Validates: Requirements 3.5**

  - [ ] 7.7 Implement createTransaction, updateTransaction, deleteTransaction server actions
    - Each extracts userId from JWT, validates with Zod, calls DynamoTransactionRepository, revalidates 	ransactions cache tag
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.8_

  - [ ]* 7.8 Write unit tests for transaction server actions
    - Test validation rejection, successful create/update/delete, and aggregate update side-effects
    - _Requirements: 3.1, 3.2, 3.3, 3.4_

  - [ ] 7.9 Build /transactions page wiring form, list, and filter bar together
    - Server component fetches initial data; client components handle interactions
    - _Requirements: 3.5_

- [ ] 8. Financial goals feature
  - Implement projectGoalCompletion(goal, monthlyContribution, annualReturn) pure function
  - Build GoalCard component with progress bar, projected completion date, on-track indicator, and shortfall display
  - Build GoalForm component for create/edit
  - Implement server actions: createGoal, updateGoal, ddFundsToGoal, deleteGoal
  - Wire /goals page
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8_

  - [ ] 8.1 Implement projectGoalCompletion pure function
    - Compound growth simulation; returns projectedCompletionDate, monthsRemaining, onTrack, shortfallMonths, milestones
    - _Requirements: 4.7, 4.8_

  - [ ]* 8.2 Write property test for goal projection on-track determination
    - **Property 6: Goal Projection On-Track Determination**
    - **Validates: Requirements 4.8**

  - [ ] 8.3 Build GoalCard and GoalForm components
    - GoalCard: progress bar, projected date, on-track/off-track indicator, shortfall in months
    - GoalForm: name, targetAmount, currentAmount, targetDate, category, colour picker
    - _Requirements: 4.6, 4.7, 4.8_

  - [ ] 8.4 Implement goal server actions (create, update, addFunds, delete)
    - ddFundsToGoal shows confirmation dialog when overfunding; caps at 	argetAmount on confirm
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 17.7_

  - [ ]* 8.5 Write unit tests for goal server actions and overfunding guard
    - Test overfunding confirmation flow, cap behaviour, and CRUD operations
    - _Requirements: 4.3_

  - [ ] 8.6 Build /goals page
    - Server component fetches goals; renders GoalCard list with add/edit/delete modals
    - _Requirements: 4.1, 4.6_

- [ ] 9. Dashboard overview page
  - Build all six dashboard widgets: NetWorthWidget, SpendingSummaryWidget, CashFlowChart, GoalsProgressWidget, RecentTransactionsWidget, FIREProgressWidget
  - Wrap each widget in an ErrorBoundary with "Unable to load data" fallback
  - Implement /dashboard server component fetching all data in parallel via repositories
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8, 15.5, 17.2_

  - [ ] 9.1 Build NetWorthWidget and SpendingSummaryWidget
    - NetWorthWidget: net worth + trend indicator vs previous month
    - SpendingSummaryWidget: doughnut chart by category; budget utilisation when monthlyBudget > 0
    - _Requirements: 2.3, 2.4, 15.5_

  - [ ] 9.2 Build CashFlowChart and RecentTransactionsWidget
    - CashFlowChart: bar chart of income vs expenses over trailing 12 months from AGG#MONTH# items
    - RecentTransactionsWidget: last 5 transactions with category, amount, date
    - _Requirements: 2.5, 2.7_

  - [ ] 9.3 Build GoalsProgressWidget and FIREProgressWidget
    - GoalsProgressWidget: up to 3 active goals with progress bars and target dates
    - FIREProgressWidget: FIRE number progress summary
    - _Requirements: 2.6_

  - [ ] 9.4 Build /dashboard server component with parallel data fetching and ErrorBoundary wrappers
    - Parallel Promise.all across all repository calls; each widget wrapped in <ErrorBoundary>
    - _Requirements: 2.1, 2.2, 2.8, 17.2_

  - [ ]* 9.5 Write component tests for dashboard widgets
    - Test each widget renders correctly with mock data and shows fallback on error
    - _Requirements: 2.3, 2.4, 2.5, 2.6, 2.7, 2.8_

- [ ] 10. Checkpoint — Data layer and core features complete
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 11. Buy vs Wait calculator
  - Implement calculateBuyVsWait(inputs) pure function in src/lib/calculators/buy-vs-wait/calculator.ts
  - Build BuyVsWaitCalculator client component with live-recompute on input change (debounced 300 ms), inline validation, and results panel
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7_

  - [ ] 11.1 Implement calculateBuyVsWait pure function
    - Monthly compound growth loop; returns easible, monthsToSave, djustedPriceAtPurchase, investmentGainIfWait, opportunityCostBuyNow, 
ecommendation
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ]* 11.2 Write property test for Buy vs Wait feasibility and recommendation
    - **Property 7: Buy vs Wait Feasibility and Recommendation**
    - **Validates: Requirements 5.1, 5.2, 5.3, 5.4, 5.5**

  - [ ] 11.3 Build BuyVsWaitCalculator client component
    - Input fields with inline validation; live recompute within 300 ms on any change; results panel with recommendation
    - _Requirements: 5.1, 5.6, 5.7_

  - [ ]* 11.4 Write unit tests for calculateBuyVsWait edge cases
    - Test zero monthly savings (infeasible), boundary at 600 months, zero inflation, zero return
    - _Requirements: 5.5_

- [ ] 12. Home Purchase calculator
  - Implement calculateHomePurchaseCosts(inputs, activeBenefits) pure function with CMHC rules, provincial LTT schedules, mortgage formula, and benefit offsets
  - Create src/lib/calculators/home-purchase/benefit-registry.ts with BENEFIT_REGISTRY array (FHSA, RRSP HBP, FTHBI, ON LTT rebate)
  - Build BenefitTogglePanel component driven entirely by registry data
  - Build HomePurchaseCostBreakdown component
  - Implement saveHomePurchasePlan server action
  - Wire /calculators/home-purchase page
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 6.11, 6.12, 17.5_

  - [ ] 12.1 Implement calculateHomePurchaseCosts pure function
    - CMHC minimum down payment rules, CMHC premium brackets, provincial LTT schedules for all 13 provinces, mortgage amortisation formula, benefit offset summation, months-to-save calculation
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 6.8, 6.9_

  - [ ]* 12.2 Write property test for CMHC premium bracket rules
    - **Property 8: Home Purchase CMHC Premium Bracket Rules**
    - **Validates: Requirements 6.2, 6.3**

  - [ ]* 12.3 Write property test for net cash required invariant
    - **Property 9: Home Purchase Net Cash Required Invariant**
    - **Validates: Requirements 6.8**

  - [ ] 12.4 Create BenefitRegistry with initial benefit entries
    - src/lib/calculators/home-purchase/benefit-registry.ts with FHSA, RRSP HBP, FTHBI (expired), ON LTT rebate
    - _Requirements: 6.6, 6.10, 6.11_

  - [ ] 12.5 Build BenefitTogglePanel and HomePurchaseCostBreakdown components
    - BenefitTogglePanel: renders from registry; expired benefits show greyed "Ended [date]" badge with disabled toggle
    - HomePurchaseCostBreakdown: itemised cost and offset summary
    - _Requirements: 6.6, 6.7, 6.10, 6.11, 17.5_

  - [ ] 12.6 Implement saveHomePurchasePlan server action and wire /calculators/home-purchase page
    - Server action persists plan to DynamoDB; page wires all components with live recompute
    - _Requirements: 6.12, 16.1, 16.2_

  - [ ]* 12.7 Write unit tests for calculateHomePurchaseCosts
    - Test all CMHC brackets, all provincial LTT schedules, benefit offset application, mortgage formula
    - _Requirements: 6.2, 6.3, 6.4, 6.5_

- [ ] 13. Market Data API layer
  - Define MarketDataProvider interface in src/lib/market/market-data-provider.ts
  - Implement YahooFinanceProvider using yahoo-finance2
  - Implement getMarketProvider() factory reading MARKET_DATA_PROVIDER env var
  - Create /api/market/quote route handler with unstable_cache (60-second TTL) and 502 error handling
  - Create /api/market/search and /api/market/history route handlers
  - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5, 11.6, 11.7_

  - [ ] 13.1 Define MarketDataProvider interface and YahooFinanceProvider implementation
    - getQuote, searchTickers, getHistoricalPrices methods; all prices in minor units
    - _Requirements: 11.1, 11.4, 11.5, 11.6_

  - [ ] 13.2 Implement provider factory and market API route handlers
    - getMarketProvider() factory; /api/market/quote with 60-second unstable_cache; 502 on provider error/timeout; /api/market/search; /api/market/history
    - _Requirements: 11.2, 11.3, 11.4, 11.7_

  - [ ]* 13.3 Write unit tests for market API route handler
    - Test cache hit, cache miss, provider error (502), missing ticker (400)
    - _Requirements: 11.2, 11.3_

- [ ] 14. Investment / Seed Money calculator
  - Implement calculateInvestmentGrowth(inputs) pure function supporting monthly/quarterly/annual compounding
  - Build TickerSearch component with debounced live quote fetch and error handling
  - Build InvestmentCalculator client component with projection chart (
echarts)
  - Implement saveInvestmentPlan server action
  - Wire /calculators/investment page
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7_

  - [ ] 14.1 Implement calculateInvestmentGrowth pure function
    - Supports monthly, quarterly, nnually compounding; returns inalValue, 	otalContributed, 	otalGrowth, growthPercentage, yearlyBreakdown
    - _Requirements: 7.1, 7.2, 7.6_

  - [ ]* 14.2 Write property test for investment growth non-negativity
    - **Property 10: Investment Growth Non-Negativity**
    - **Validates: Requirements 7.1, 7.2**

  - [ ] 14.3 Build TickerSearch component and InvestmentCalculator page
    - TickerSearch: debounced input, calls /api/market/quote, populates seed amount; shows "Quote unavailable" fallback
    - InvestmentCalculator: live recompute, projection chart, save plan button
    - _Requirements: 7.3, 7.4, 7.5, 7.7_

  - [ ] 14.4 Implement saveInvestmentPlan server action and wire /calculators/investment page
    - _Requirements: 7.7, 16.1, 16.2_

  - [ ]* 14.5 Write unit tests for calculateInvestmentGrowth
    - Test all three compounding frequencies, zero seed amount, zero contribution, boundary years
    - _Requirements: 7.1, 7.2, 7.6_

- [ ] 15. Take-Home Pay calculator and tax engine
  - Create src/lib/calculators/take-home/tax-bracket-registry.ts with TaxBracketSet[], CPPRates[], EIRates[] for 2020–2025 (all 13 provinces + federal)
  - Implement calculateTakeHome(inputs, registry, cppRates, eiRates) pure function with progressive brackets, CPP1+2, EI, BPA credits, ON/PEI surtax, per-paycheque breakdown
  - Implement optimizeRRSP(currentResult, availableRRSPRoom) pure function
  - Build TakeHomeCalculator client component with all input panels, tax breakdown summary, paycheque timeline chart, and RRSP optimizer panel
  - Wire /calculators/take-home page
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9, 8.10, 8.11, 8.12, 8.13, 8.14, 8.15, 8.16, 8.17, 8.18, 17.4, 17.6_

  - [ ] 15.1 Create TaxBracketRegistry with 2020–2025 data for all provinces
    - TaxBracketSet[] for federal + 13 provinces; CPPRates[] and EIRates[] per year; SurtaxThreshold[] for ON and PEI
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.17, 8.18_

  - [ ] 15.2 Implement calculateTakeHome pure function
    - Progressive federal and provincial brackets; CPP1+2; EI (QC rate); BPA credits; ON/PEI surtax; per-paycheque breakdown; tax year fallback with banner
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9, 8.10, 8.11, 17.4_

  - [ ]* 15.3 Write property test for take-home pay core invariants
    - **Property 11: Take-Home Pay Core Invariants**
    - **Validates: Requirements 8.8, 8.9, 8.10**

  - [ ]* 15.4 Write property test for per-paycheque consistency
    - **Property 12: Take-Home Per-Paycheque Consistency**
    - **Validates: Requirements 8.7**

  - [ ] 15.5 Implement optimizeRRSP pure function
    - Finds nearest lower bracket boundary; builds contribution scenarios (, , , , optimal, max room); validates optimalContribution <= availableRRSPRoom
    - _Requirements: 8.12, 8.13, 8.14, 8.15, 8.16_

  - [ ]* 15.6 Write property test for RRSP optimisation bounds
    - **Property 14: RRSP Optimisation Bounds**
    - **Validates: Requirements 8.13, 8.14**

  - [ ] 15.7 Build TakeHomeCalculator client component and wire /calculators/take-home page
    - Income panel, deductions panel, bonus/RSU panel, RRSP optimizer panel, tax breakdown summary, paycheque timeline chart; RRSP room warning
    - _Requirements: 8.6, 8.11, 8.12, 8.15, 8.16, 17.6_

  - [ ]* 15.8 Write unit tests for calculateTakeHome across provinces
    - Test ON surtax, PEI surtax, QC EI rate, CPP2 ceiling, BPA credit, tax year fallback
    - _Requirements: 8.2, 8.3, 8.4, 8.5, 8.11_

- [ ] 16. Checkpoint — Calculator modules complete
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 17. FIRE Dashboard
  - Implement calculateFIRE(inputs) pure function with FIRE number, years-to-FI simulation, Coast FIRE, 30-year withdrawal phase, and milestones
  - Build FIREDashboard client component with FIREScenarioSelector, projection timeline chart, Coast FIRE panel, withdrawal rate analysis
  - Implement saveFirePlan server action
  - Wire /fire page
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7, 9.8, 9.9, 9.10_

  - [ ] 17.1 Implement calculateFIRE pure function
    - FIRE number (nnualExpenses / withdrawalRate); compound growth accumulation loop; Coast FIRE number and age; 30-year withdrawal simulation; 25%/50%/75% milestones
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7_

  - [ ]* 17.2 Write property test for FIRE number and projection invariants
    - **Property 13: FIRE Number and Projection Invariants**
    - **Validates: Requirements 9.2, 9.4, 9.5**

  - [ ] 17.3 Build FIREDashboard client component and FIREScenarioSelector
    - Scenario selector (Lean/Regular/Fat/Coast/Barista); projection timeline chart; Coast FIRE panel; withdrawal phase table; recomputes on scenario change
    - _Requirements: 9.8, 9.9_

  - [ ] 17.4 Implement saveFirePlan server action and wire /fire page
    - _Requirements: 9.10, 16.1, 16.2_

  - [ ]* 17.5 Write unit tests for calculateFIRE
    - Test already-at-FIRE-number, zero contributions, Coast FIRE age >= 65, withdrawal phase depletion
    - _Requirements: 9.2, 9.3, 9.5, 9.6_

- [ ] 18. Investment Portfolio page
  - Build PortfolioHoldings table component with live price enrichment from MarketDataProvider
  - Build HoldingForm modal for add/edit
  - Implement server actions: createHolding, updateHolding, deleteHolding
  - Wire /investments page with live price enrichment on load
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5, 10.6_

  - [ ] 18.1 Build PortfolioHoldings table component
    - Displays ticker, name, shares, average cost basis, current price, current value, gain/loss, gain/loss %; sorted by current value descending; "Price unavailable" indicator on API failure
    - _Requirements: 10.2, 10.5, 10.6_

  - [ ] 18.2 Build HoldingForm modal and implement holding server actions
    - createHolding, updateHolding, deleteHolding server actions with JWT extraction and Zod validation
    - _Requirements: 10.1, 10.3, 10.4_

  - [ ] 18.3 Wire /investments page with live price enrichment
    - Server component fetches holdings from repository; enriches each with current price from MarketDataProvider; renders PortfolioHoldings
    - _Requirements: 10.2, 10.6_

  - [ ]* 18.4 Write component tests for PortfolioHoldings
    - Test rendering with enriched data, "Price unavailable" fallback, sort order
    - _Requirements: 10.5, 10.6_

- [ ] 19. User preferences and settings page
  - Build /settings page with currency, locale, theme, and monthly budget inputs
  - Implement updateUserPreferences server action persisting to UserProfileItem
  - Apply currency/locale formatting globally via Zustand store; apply theme via CSS class on <html>
  - _Requirements: 15.1, 15.2, 15.3, 15.4, 15.5_

  - [ ] 19.1 Build settings page and updateUserPreferences server action
    - Form with currency (USD/EUR/GBP/AUD/CAD/NZD), locale, theme (light/dark/system), monthly budget; server action persists to DynamoDB; error toast on failure
    - _Requirements: 15.1, 15.2, 15.3, 15.4_

  - [ ] 19.2 Wire currency/locale and theme preferences across the app
    - Load preferences in RootLayout; pass to Zustand store; apply theme class to <html>; ormatCurrency uses active currency/locale everywhere
    - _Requirements: 15.3, 15.4, 15.5_

  - [ ]* 19.3 Write unit tests for updateUserPreferences server action
    - Test persistence success, failure toast, and preference validation
    - _Requirements: 15.1_

- [ ] 20. Calculator plan persistence (load, list, delete)
  - Build plan list UI for Home Purchase, FIRE, and Investment calculators (load saved plans, delete plans)
  - Implement loadCalculatorPlan, deleteCalculatorPlan server actions with deserialisation error handling
  - Enforce 50-plan-per-type limit in DynamoCalculatorPlanRepository
  - _Requirements: 16.1, 16.2, 16.3, 16.4, 16.5, 16.6_

  - [ ] 20.1 Build saved plan list UI for all three calculator types
    - Dropdown or panel listing saved plans by name; load restores calculator state; delete removes from DynamoDB
    - _Requirements: 16.3, 16.4, 16.5_

  - [ ] 20.2 Implement loadCalculatorPlan and deleteCalculatorPlan server actions
    - loadCalculatorPlan: deserialises inputs and result; error toast on deserialisation failure
    - deleteCalculatorPlan: removes from DynamoDB
    - _Requirements: 16.3, 16.5, 17.1_

  - [ ]* 20.3 Write unit tests for plan persistence server actions
    - Test save success, save failure toast, load with valid/invalid JSON, delete, 50-plan limit enforcement
    - _Requirements: 16.1, 16.3, 16.6_

- [ ] 21. Calculators hub page and navigation wiring
  - Build /calculators hub page listing all four calculators with descriptions and links
  - Ensure all calculator routes (/calculators/buy-vs-wait, /calculators/home-purchase, /calculators/investment, /calculators/take-home) are protected by middleware
  - _Requirements: 13.3_

  - [ ] 21.1 Build /calculators hub page and verify all route protection
    - Hub page with cards for each calculator; verify middleware protects all calculator routes
    - _Requirements: 13.3_

- [ ] 22. Final checkpoint — Full integration and all tests passing
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with * are optional and can be skipped for faster MVP delivery
- All monetary amounts are stored and computed in MinorUnits (integer cents) throughout — never floating-point dollars
- The design uses TypeScript/Next.js; all code examples and implementations use TypeScript
- Property-based tests use ast-check; unit and component tests use itest + @testing-library/react
- Each task references specific requirements for traceability
- Checkpoints (tasks 5, 10, 16, 22) ensure incremental validation at natural boundaries
- The BenefitRegistry and TaxBracketRegistry are data-driven — adding new entries requires no algorithm changes
- All server actions extract userId exclusively from the verified Cognito JWT, never from client input
- DynamoDB access uses the IAM execution role on Amplify — no database credentials in environment variables

## Task Dependency Graph

```json
{
  "waves": [
    {
      "id": 0,
      "tasks": ["1.1", "1.2", "1.3"]
    },
    {
      "id": 1,
      "tasks": ["2.1", "2.2", "4.1"]
    },
    {
      "id": 2,
      "tasks": ["2.3", "2.4", "2.5", "3.1", "4.2", "6.1"]
    },
    {
      "id": 3,
      "tasks": ["3.2", "3.3", "3.4", "4.3", "6.2", "6.3"]
    },
    {
      "id": 4,
      "tasks": ["4.4", "4.5", "6.4", "6.5", "6.6"]
    },
    {
      "id": 5,
      "tasks": ["4.6", "7.1", "9.1"]
    },
    {
      "id": 6,
      "tasks": ["7.2", "7.3", "7.4", "7.5", "9.2", "9.3", "13.1"]
    },
    {
      "id": 7,
      "tasks": ["7.6", "7.7", "9.4", "13.2", "11.1"]
    },
    {
      "id": 8,
      "tasks": ["7.8", "7.9", "9.5", "13.3", "11.2", "11.3", "8.1", "12.1", "15.1"]
    },
    {
      "id": 9,
      "tasks": ["11.4", "8.2", "8.3", "12.2", "12.3", "12.4", "15.2", "14.1"]
    },
    {
      "id": 10,
      "tasks": ["8.4", "8.5", "12.5", "15.3", "15.4", "14.2", "14.3"]
    },
    {
      "id": 11,
      "tasks": ["8.6", "12.6", "15.5", "14.4", "17.1"]
    },
    {
      "id": 12,
      "tasks": ["12.7", "15.6", "14.5", "17.2", "17.3"]
    },
    {
      "id": 13,
      "tasks": ["15.7", "17.4", "17.5", "18.1"]
    },
    {
      "id": 14,
      "tasks": ["15.8", "18.2", "19.1"]
    },
    {
      "id": 15,
      "tasks": ["18.3", "19.2", "20.1"]
    },
    {
      "id": 16,
      "tasks": ["18.4", "19.3", "20.2"]
    },
    {
      "id": 17,
      "tasks": ["20.3", "21.1"]
    }
  ]
}
```
